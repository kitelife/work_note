# 请求处理主流程

接着 *启动流程*，来看看请求处理流程。

`app.Handlers`的类型为：`*ControllerRegistor`，其`ServeHTTP`方法，也即请求处理的入口，实现如下：

```go
// Implement http.Handler interface.
func (p *ControllerRegister) ServeHTTP(rw http.ResponseWriter, r *http.Request) {
    // 请求处理开始时间
	startTime := time.Now()
	var (
		runRouter  reflect.Type
		findRouter bool
		runMethod  string
		routerInfo *controllerInfo
	)
	context := p.pool.Get().(*beecontext.Context)
	context.Reset(rw, r)
	defer p.pool.Put(context)
	// 处理出错时，交给recoverPanic来做错误处理，并响应
   // 还记得 **启动流程** 中的 registerDefaultErrorHandler() 吗？
	defer p.recoverPanic(context)

	context.Output.EnableGzip = BConfig.EnableGzip

	if BConfig.RunMode == DEV {
		context.Output.Header("Server", BConfig.ServerName)
	}

	var urlPath string
	if !BConfig.RouterCaseSensitive {
		urlPath = strings.ToLower(r.URL.Path)
	} else {
		urlPath = r.URL.Path
	}

	// filter wrong http method
	if _, ok := HTTPMETHOD[r.Method]; !ok {
		http.Error(rw, "Method Not Allowed", 405)
		goto Admin
	}

	// filter for static file
	if p.execFilter(context, BeforeStatic, urlPath) {
		goto Admin
	}

    // 尝试处理静态文件请求
	serverStaticRouter(context)
	// 当前请求的可能确实是个静态文件，所以在serverStaticRouter中已经做出了响应
	if context.ResponseWriter.Started {
	   // 那么以下也就不用再接着去查找非静态文件的路由了
		findRouter = true
		goto Admin
	}

	// session init
	if BConfig.WebConfig.Session.SessionOn {
		var err error
		context.Input.CruSession, err = GlobalSessions.SessionStart(rw, r)
		if err != nil {
			Error(err)
			exception("503", context)
			return
		}
		// 得记得释放呀
		defer func() {
			context.Input.CruSession.SessionRelease(rw)
		}()
	}

    // 解析请求体
	if r.Method != "GET" && r.Method != "HEAD" {
		if BConfig.CopyRequestBody && !context.Input.IsUpload() {
			context.Input.CopyBody(BConfig.MaxMemory)
		}
		context.Input.ParseFormOrMulitForm(BConfig.MaxMemory)
	}

	if p.execFilter(context, BeforeRouter, urlPath) {
		goto Admin
	}

	if !findRouter {
		httpMethod := r.Method
		if t, ok := p.routers[httpMethod]; ok {
		    // 如何匹配的？
			runObject := t.Match(urlPath, context)
			// 这里貌似很牛逼的样子
			if r, ok := runObject.(*controllerInfo); ok {
				routerInfo = r
				findRouter = true
				if splat := context.Input.Param(":splat"); splat != "" {
					for k, v := range strings.Split(splat, "/") {
						context.Input.SetParam(strconv.Itoa(k), v)
					}
				}
			}
		}

	}

	//if no matches to url, throw a not found exception
	if !findRouter {
		exception("404", context)
		goto Admin
	}

	if findRouter {
		//execute middleware filters
		if p.execFilter(context, BeforeExec, urlPath) {
			goto Admin
		}
		isRunnable := false
		if routerInfo != nil {
			if routerInfo.routerType == routerTypeRESTFul {
				if _, ok := routerInfo.methods[r.Method]; ok {
					isRunnable = true
					routerInfo.runFunction(context)
				} else {
					exception("405", context)
					goto Admin
				}
			} else if routerInfo.routerType == routerTypeHandler {
				isRunnable = true
				routerInfo.handler.ServeHTTP(rw, r)
			} else {
				runRouter = routerInfo.controllerType
				method := r.Method
				if r.Method == "POST" && context.Input.Query("_method") == "PUT" {
					method = "PUT"
				}
				if r.Method == "POST" && context.Input.Query("_method") == "DELETE" {
					method = "DELETE"
				}
				if m, ok := routerInfo.methods[method]; ok {
					runMethod = m
				} else if m, ok = routerInfo.methods["*"]; ok {
					runMethod = m
				} else {
					runMethod = method
				}
			}
		}

		// also defined runRouter & runMethod from filter
		if !isRunnable {
			//Invoke the request handler
			vc := reflect.New(runRouter)
			execController, ok := vc.Interface().(ControllerInterface)
			if !ok {
				panic("controller is not ControllerInterface")
			}

			//call the controller init function
			execController.Init(context, runRouter.Name(), runMethod, vc.Interface())

			//call prepare function
			execController.Prepare()

			//if XSRF is Enable then check cookie where there has any cookie in the  request's cookie _csrf
			if BConfig.WebConfig.EnableXSRF {
				execController.XSRFToken()
				if r.Method == "POST" || r.Method == "DELETE" || r.Method == "PUT" ||
					(r.Method == "POST" && (context.Input.Query("_method") == "DELETE" || context.Input.Query("_method") == "PUT")) {
					execController.CheckXSRFCookie()
				}
			}

			execController.URLMapping()

			if !context.ResponseWriter.Started {
				//exec main logic
				switch runMethod {
				case "GET":
					execController.Get()
				case "POST":
					execController.Post()
				case "DELETE":
					execController.Delete()
				case "PUT":
					execController.Put()
				case "HEAD":
					execController.Head()
				case "PATCH":
					execController.Patch()
				case "OPTIONS":
					execController.Options()
				default:
					if !execController.HandlerFunc(runMethod) {
						var in []reflect.Value
						method := vc.MethodByName(runMethod)
						method.Call(in)
					}
				}

				//render template
				if !context.ResponseWriter.Started && context.Output.Status == 0 {
					if BConfig.WebConfig.AutoRender {
						if err := execController.Render(); err != nil {
							panic(err)
						}
					}
				}
			}

			// finish all runRouter. release resource
			execController.Finish()
		}

		//execute middleware filters
		if p.execFilter(context, AfterExec, urlPath) {
			goto Admin
		}
	}

	p.execFilter(context, FinishRouter, urlPath)

Admin:
	timeDur := time.Since(startTime)
	//admin module record QPS
	if BConfig.Listen.EnableAdmin {
		if FilterMonitorFunc(r.Method, r.URL.Path, timeDur) {
			if runRouter != nil {
				go toolbox.StatisticsMap.AddStatistics(r.Method, r.URL.Path, runRouter.Name(), timeDur)
			} else {
				go toolbox.StatisticsMap.AddStatistics(r.Method, r.URL.Path, "", timeDur)
			}
		}
	}

	if BConfig.RunMode == DEV || BConfig.Log.AccessLogs {
		var devInfo string
		if findRouter {
			if routerInfo != nil {
				devInfo = fmt.Sprintf("| % -10s | % -40s | % -16s | % -10s | % -40s |", r.Method, r.URL.Path, timeDur.String(), "match", routerInfo.pattern)
			} else {
				devInfo = fmt.Sprintf("| % -10s | % -40s | % -16s | % -10s |", r.Method, r.URL.Path, timeDur.String(), "match")
			}
		} else {
			devInfo = fmt.Sprintf("| % -10s | % -40s | % -16s | % -10s |", r.Method, r.URL.Path, timeDur.String(), "notmatch")
		}
		if DefaultAccessLogFilter == nil || !DefaultAccessLogFilter.Filter(context) {
			Debug(devInfo)
		}
	}

	// Call WriteHeader if status code has been set changed
	if context.Output.Status != 0 {
		context.ResponseWriter.WriteHeader(context.Output.Status)
	}
}
```

------

上述代码中使用了 `responseWriter`，其定义如下：

```go
//responseWriter is a wrapper for the http.ResponseWriter
//started set to true if response was written to then don't execute other handler
type responseWriter struct {
	writer  http.ResponseWriter
	started bool
	status  int
}
```

------

静态文件请求处理函数 `serverStaticRouter` 定义在文件 `staticfile.go`中，实现如下所示：

```go
func serverStaticRouter(ctx *context.Context) {
    // 非GET、HEAD请求不认为是静态文件请求
	if ctx.Input.Method() != "GET" && ctx.Input.Method() != "HEAD" {
		return
	}

	forbidden, filePath, fileInfo, err := lookupFile(ctx)
	if err == errNotStaticRequest {
		return
	}

	if forbidden {
		exception("403", ctx)
		return
	}

	if filePath == "" || fileInfo == nil {
		if BConfig.RunMode == DEV {
			Warn("Can't find/open the file:", filePath, err)
		}
		http.NotFound(ctx.ResponseWriter, ctx.Request)
		return
	}
	if fileInfo.IsDir() {
		//serveFile will list dir
		http.ServeFile(ctx.ResponseWriter, ctx.Request, filePath)
		return
	}

	var enableCompress = BConfig.EnableGzip && isStaticCompress(filePath)
	var acceptEncoding string
	if enableCompress {
		acceptEncoding = context.ParseEncoding(ctx.Request)
	}
	b, n, sch, err := openFile(filePath, fileInfo, acceptEncoding)
	if err != nil {
		if BConfig.RunMode == DEV {
			Warn("Can't compress the file:", filePath, err)
		}
		http.NotFound(ctx.ResponseWriter, ctx.Request)
		return
	}

	if b {
		ctx.Output.Header("Content-Encoding", n)
	} else {
		ctx.Output.Header("Content-Length", strconv.FormatInt(sch.size, 10))
	}

	http.ServeContent(ctx.ResponseWriter, ctx.Request, filePath, sch.modTime, sch)
	return
}
```
------

开启session的 `context.Input.CruSession, err = GlobalSessions.SessionStart(w, r)` 一行中的 `GlobalSessions` 想必还记得吧？在**启动流程**中有 `GlobalSessions, err = session.NewManager(SessionProvider, sessionConfig)` 一行。 `GlobalSessions.SessionStart` 方法实现如下所示：

```go
// Start session. generate or read the session id from http request.
// if session id exists, return SessionStore with this id.
func (manager *Manager) SessionStart(w http.ResponseWriter, r *http.Request) (session SessionStore, err error) {
   // 看看请求头中是否有session id的cookie
	cookie, errs := r.Cookie(manager.config.CookieName)
	if errs != nil || cookie.Value == "" {
	   // 没有则新生成一个
		sid, errs := manager.sessionId(r)
		if errs != nil {
			return nil, errs
		}
		session, err = manager.provider.SessionRead(sid)
		cookie = &http.Cookie{
			Name:     manager.config.CookieName,
			Value:    url.QueryEscape(sid),
			Path:     "/",
			HttpOnly: true,
			Secure:   manager.isSecure(r),
			Domain:   manager.config.Domain,
		}
		if manager.config.CookieLifeTime > 0 {
			cookie.MaxAge = manager.config.CookieLifeTime
			cookie.Expires = time.Now().Add(time.Duration(manager.config.CookieLifeTime) * time.Second)
		}
		if manager.config.EnableSetCookie {
			http.SetCookie(w, cookie)
		}
		r.AddCookie(cookie)
	} else {
	   // 有，则取出目标session id，接着读取session数据
		sid, errs := url.QueryUnescape(cookie.Value)
		if errs != nil {
			return nil, errs
		}
		// 但可能目标session已经过期，不存在了，则也自动生成一个新的
		if manager.provider.SessionExist(sid) {
			session, err = manager.provider.SessionRead(sid)
		} else {
		   // 这代码拷贝得让我有些不爽呀
			sid, err = manager.sessionId(r)
			if err != nil {
				return nil, err
			}
			session, err = manager.provider.SessionRead(sid)
			cookie = &http.Cookie{
				Name:     manager.config.CookieName,
				Value:    url.QueryEscape(sid),
				Path:     "/",
				HttpOnly: true,
				Secure:   manager.isSecure(r),
				Domain:   manager.config.Domain,
			}
			if manager.config.CookieLifeTime > 0 {
				cookie.MaxAge = manager.config.CookieLifeTime
				cookie.Expires = time.Now().Add(time.Duration(manager.config.CookieLifeTime) * time.Second)
			}
			// 这是啥意思？
			if manager.config.EnableSetCookie {
				http.SetCookie(w, cookie)
			}
			r.AddCookie(cookie)
		}
	}
	return
}
```

