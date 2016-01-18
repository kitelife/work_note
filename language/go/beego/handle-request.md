# 请求处理

接着 *启动流程*，来看看请求处理流程。

`app.Handlers`的类型为：`*ControllerRegistor`，其`ServeHTTP`方法，也即请求处理的入口，实现如下：

```go
// Implement http.Handler interface.
func (p *ControllerRegistor) ServeHTTP(rw http.ResponseWriter, r *http.Request) {
   // 请求处理开始时间
	starttime := time.Now()
	var runrouter reflect.Type
	var findrouter bool
	var runMethod string
	var routerInfo *controllerInfo

   // 将http.ResponseWriter封装成自己定义的responseWriter，这样就可以干很多事情啦！
	w := &responseWriter{writer: rw}

   // 在dev模式下，会在响应头中加Server字段
	if RunMode == "dev" {
		w.Header().Set("Server", BeegoServerName)
	}

   // 请求处理上下文
	// init context
	context := &beecontext.Context{
		ResponseWriter: w,
		Request:        r,
		Input:          beecontext.NewInput(r),
		Output:         beecontext.NewOutput(),
	}
	context.Output.Context = context
	context.Output.EnableGzip = EnableGzip

   // 处理出错时，交给recoverPanic来做错误处理，并响应
   // 还记得 **启动流程** 中的 registerDefaultErrorHandler() 吗？
	defer p.recoverPanic(context)

	var urlPath string
	// 如果配置请求路径大小写不敏感，则统一转成小写
	if !RouterCaseSensitive {
		urlPath = strings.ToLower(r.URL.Path)
	} else {
		urlPath = r.URL.Path
	}
	// defined filter function
	// 用于执行不同阶段的过滤器
	// pos 参数就是指“阶段”
	do_filter := func(pos int) (started bool) {
		if p.enableFilter {
		    // 去除该阶段的过滤器列表
			if l, ok := p.filters[pos]; ok {
				for _, filterR := range l {
					if filterR.returnOnOutput && w.started {
						return true
					}
					// 可以看出过滤器需要实现ValidRouter
					// 验证URL路径并返回其中的URL参数
					if ok, params := filterR.ValidRouter(urlPath); ok {
						for k, v := range params {
							if context.Input.Params == nil {
								context.Input.Params = make(map[string]string)
							}
							context.Input.Params[k] = v
						}
						// filterFunc
						// 这个方法该怎么实现呢？
						filterR.filterFunc(context)
					}
					// 如果过滤器直接就处理了请求并作出响应
					if filterR.returnOnOutput && w.started {
						return true
					}
				}
			}
		}
		return false
	}

	// filter wrong httpmethod
	// 当前HTTP请求方法是否支持？
	if _, ok := HTTPMETHOD[r.Method]; !ok {
		http.Error(w, "Method Not Allowed", 405)
		// 去Admin记录一下当前请求的信息
		goto Admin
	}

	// filter for static file
	// 在处理静态文件请求之前执行该阶段的过滤器
	if do_filter(BeforeStatic) {
		goto Admin
	}

   // 尝试处理静态文件请求
	serverStaticRouter(context)
	// 当前请求的可能确实是个静态文件，所以在serverStaticRouter中已经做出了响应
	if w.started {
	    // 那么以下也就不用再接着去查找非静态文件的路由了
		findrouter = true
		goto Admin
	}

	// session init
	// 开启session
	if SessionOn {
		var err error
		context.Input.CruSession, err = GlobalSessions.SessionStart(w, r)
		if err != nil {
			Error(err)
			exception("503", context)
			return
		}
		// 得记得释放呀
		defer func() {
			context.Input.CruSession.SessionRelease(w)
		}()
	}

    // 解析请求体
	if r.Method != "GET" && r.Method != "HEAD" {
		if CopyRequestBody && !context.Input.IsUpload() {
			context.Input.CopyBody()
		}
		context.Input.ParseFormOrMulitForm(MaxMemory)
	}
    // 执行路由查找前的过滤器
	if do_filter(BeforeRouter) {
		goto Admin
	}

   // 接下来要查找匹配路由了
   // 这里什么时候条件会为真呢？
	if context.Input.RunController != nil && context.Input.RunMethod != "" {
		findrouter = true
		runMethod = context.Input.RunMethod
		runrouter = context.Input.RunController
	}

	if !findrouter {
		http_method := r.Method

       // PUT、DELETE 兼容方案
		if http_method == "POST" && context.Input.Query("_method") == "PUT" {
			http_method = "PUT"
		}

		if http_method == "POST" && context.Input.Query("_method") == "DELETE" {
			http_method = "DELETE"
		}

       //
		if t, ok := p.routers[http_method]; ok {
		   // 如何匹配的？
			runObject, p := t.Match(urlPath)
			// 这里貌似很牛逼的样子
			if r, ok := runObject.(*controllerInfo); ok {
				routerInfo = r
				findrouter = true
				if splat, ok := p[":splat"]; ok {
					splatlist := strings.Split(splat, "/")
					for k, v := range splatlist {
						p[strconv.Itoa(k)] = v
					}
				}
				if p != nil {
					context.Input.Params = p
				}
			}
		}

	}

	//if no matches to url, throw a not found exception
	if !findrouter {
		exception("404", context)
		goto Admin
	}

	if findrouter {
		//execute middleware filters
		if do_filter(BeforeExec) {
			goto Admin
		}
		isRunable := false
		if routerInfo != nil {
		    // RESTful风格的路由
			if routerInfo.routerType == routerTypeRESTFul {
				if _, ok := routerInfo.methods[r.Method]; ok {
					isRunable = true
					routerInfo.runfunction(context)
				} else {
					exception("405", context)
					goto Admin
				}
			} else if routerInfo.routerType == routerTypeHandler {
			    // 这是啥风格？
				isRunable = true
				routerInfo.handler.ServeHTTP(rw, r)
			} else {
				runrouter = routerInfo.controllerType
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

		// also defined runrouter & runMethod from filter
		if !isRunable {
			//Invoke the request handler
			vc := reflect.New(runrouter)
			execController, ok := vc.Interface().(ControllerInterface)
			if !ok {
				panic("controller is not ControllerInterface")
			}

			//call the controller init function
			execController.Init(context, runrouter.Name(), runMethod, vc.Interface())

			//call prepare function
			execController.Prepare()

			//if XSRF is Enable then check cookie where there has any cookie in the  request's cookie _csrf
			if EnableXSRF {
				execController.XsrfToken()
				if r.Method == "POST" || r.Method == "DELETE" || r.Method == "PUT" ||
					(r.Method == "POST" && (context.Input.Query("_method") == "DELETE" || context.Input.Query("_method") == "PUT")) {
					execController.CheckXsrfCookie()
				}
			}

			execController.URLMapping()

			if !w.started {
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
						in := make([]reflect.Value, 0)
						method := vc.MethodByName(runMethod)
						method.Call(in)
					}
				}

				//render template
				if !w.started && context.Output.Status == 0 {
				    // AutoRender 默认为 true
					if AutoRender {
						if err := execController.Render(); err != nil {
							panic(err)
						}
					}
				}
			}

			// finish all runrouter. release resource
			execController.Finish()
		}

		//execute middleware filters
		if do_filter(AfterExec) {
			goto Admin
		}
	}

	do_filter(FinishRouter)

Admin:
	timeend := time.Since(starttime)
	//admin module record QPS
	if EnableAdmin {
		if FilterMonitorFunc(r.Method, r.URL.Path, timeend) {
			if runrouter != nil {
				go toolbox.StatisticsMap.AddStatistics(r.Method, r.URL.Path, runrouter.Name(), timeend)
			} else {
				go toolbox.StatisticsMap.AddStatistics(r.Method, r.URL.Path, "", timeend)
			}
		}
	}

	if RunMode == "dev" || AccessLogs {
		var devinfo string
		if findrouter {
			if routerInfo != nil {
				devinfo = fmt.Sprintf("| % -10s | % -40s | % -16s | % -10s | % -40s |", r.Method, r.URL.Path, timeend.String(), "match", routerInfo.pattern)
			} else {
				devinfo = fmt.Sprintf("| % -10s | % -40s | % -16s | % -10s |", r.Method, r.URL.Path, timeend.String(), "match")
			}
		} else {
			devinfo = fmt.Sprintf("| % -10s | % -40s | % -16s | % -10s |", r.Method, r.URL.Path, timeend.String(), "notmatch")
		}
		if DefaultLogFilter == nil || !DefaultLogFilter.Filter(context) {
			Debug(devinfo)
		}
	}

	// Call WriteHeader if status code has been set changed
	if context.Output.Status != 0 {
		w.writer.WriteHeader(context.Output.Status)
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
   // 非GET/HEAD方法的请求不认为是静态文件请求
	if ctx.Input.Method() != "GET" && ctx.Input.Method() != "HEAD" {
		return
	}
	// 为毛不直接使用 ctx.Request.URL.Path?
	requestPath := path.Clean(ctx.Input.Request.URL.Path)
	i := 0
	// 遍历配置的静态文件目录，prefix表示URL前缀，即在Beego中可以配置多个 prefix => staticDir 对
	for prefix, staticDir := range StaticDir {
		if len(prefix) == 0 {
			continue
		}
		// 这两个文件比较特殊，所以特殊处理
		if requestPath == "/favicon.ico" || requestPath == "/robots.txt" {
			file := path.Join(staticDir, requestPath)
			if utils.FileExists(file) {
			    // 
				http.ServeFile(ctx.ResponseWriter, ctx.Request, file)
				return
			} else {
				i++
				if i == len(StaticDir) {
					http.NotFound(ctx.ResponseWriter, ctx.Request)
					return
				} else {
					continue
				}
			}
		}
		if strings.HasPrefix(requestPath, prefix) {
		    // 仅仅是路径字符串前缀还是不够的!
			if len(requestPath) > len(prefix) && requestPath[len(prefix)] != '/' {
				continue
			}
			// 拼接出目标静态文件路径
			file := path.Join(staticDir, requestPath[len(prefix):])
			finfo, err := os.Stat(file)
			if err != nil {
				if RunMode == "dev" {
					Warn("Can't find the file:", file, err)
				}
				http.NotFound(ctx.ResponseWriter, ctx.Request)
				return
			}
			//if the request is dir and DirectoryIndex is false then
			// 如果是目录的话,看看是否允许罗列出目录的内容
			if finfo.IsDir() {
				if !DirectoryIndex {
					exception("403", ctx)
					return
				} else if ctx.Input.Request.URL.Path[len(ctx.Input.Request.URL.Path)-1] != '/' {
					http.Redirect(ctx.ResponseWriter, ctx.Request, ctx.Input.Request.URL.Path+"/", 302)
					return
				}
			} else if strings.HasSuffix(requestPath, "/index.html") {
				file := path.Join(staticDir, requestPath)
				if utils.FileExists(file) {
					http.ServeFile(ctx.ResponseWriter, ctx.Request, file)
					return
				}
			}

			//This block obtained from (https://github.com/smithfox/beego) - it should probably get merged into astaxie/beego after a pull request
			// 检测当前静态文件类型是否需要压缩后传输
			// 不过我不太喜欢遍历的方式,我更喜欢map
			isStaticFileToCompress := false
			if StaticExtensionsToGzip != nil && len(StaticExtensionsToGzip) > 0 {
				for _, statExtension := range StaticExtensionsToGzip {
					if strings.HasSuffix(strings.ToLower(file), strings.ToLower(statExtension)) {
						isStaticFileToCompress = true
						break
					}
				}
			}

			if isStaticFileToCompress {
				var contentEncoding string
				if EnableGzip {
				   // 根据请求头Accept-Encoding来判断压缩方式
					contentEncoding = getAcceptEncodingZip(ctx.Request)
				}
             // 如果能流式地响应就更棒了！
				memzipfile, err := openMemZipFile(file, contentEncoding)
				if err != nil {
					return
				}

				if contentEncoding == "gzip" {
					ctx.Output.Header("Content-Encoding", "gzip")
				} else if contentEncoding == "deflate" {
					ctx.Output.Header("Content-Encoding", "deflate")
				} else {
					ctx.Output.Header("Content-Length", strconv.FormatInt(finfo.Size(), 10))
				}

             // http.ServeContent
				http.ServeContent(ctx.ResponseWriter, ctx.Request, file, finfo.ModTime(), memzipfile)

			} else {
				http.ServeFile(ctx.ResponseWriter, ctx.Request, file)
			}
			return
		}
	}
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

