# 启动流程

先来看看Go的import逻辑，因为Beego（或者其他Go语言项目）的启动流程都依赖于import的逻辑。

![init](media/init.png)

------

在Bee工具生成的Beego项目目录中，`main.go`文件内容如下所示：

```go
package main

import (
	_ "hello/routers"
	"github.com/astaxie/beego"
)

func main() {
	beego.Run()
}
```

`routers/router.go`内容：

```go
package routers

import (
	"hello/controllers"
	"github.com/astaxie/beego"
)

func init() {
    beego.Router("/", &controllers.MainController{})
}
```

从代码直观来看，一个Beego应用的启动流程如下：

1. 引入beego包
2. 注册路由
3. 执行beego包的Run方法

------

从Beego本身的工程目录可以看到，属于包`beego`的文件有很多，其中Run方法在文件`beego.go`中，方法定义如下：

```go
// Run beego application.
// beego.Run() default run on HttpPort
// beego.Run(":8089")
// beego.Run("127.0.0.1:8089")
func Run(params ...string) {
   // 这个方法调用时可以传入地址端口参数，但这里的实现比较丑
	if len(params) > 0 && params[0] != "" {
		strs := strings.Split(params[0], ":")
		if len(strs) > 0 && strs[0] != "" {
			HttpAddr = strs[0]
		}
		if len(strs) > 1 && strs[1] != "" {
			HttpPort, _ = strconv.Atoi(strs[1])
		}
	}
	initBeforeHttpRun()

   // 如果配置启用管理页面（其实是管理服务后台）
	if EnableAdmin {
		go beeAdminApp.Run()
	}

	BeeApp.Run()
}
```

其中 `initBeforeHttpRun()` 实现如下：

```go
func initBeforeHttpRun() {
   /* 从这里看出，默认的配置有两处：
   - filepath.Join(AppPath, "conf", "app.conf")
   - filepath.Join(workPath, "conf", "app.conf")
   */
	// if AppConfigPath not In the conf/app.conf reParse config
	if AppConfigPath != filepath.Join(AppPath, "conf", "app.conf") {
		err := ParseConfig()
		if err != nil && AppConfigPath != filepath.Join(workPath, "conf", "app.conf") {
			// configuration is critical to app, panic here if parse failed
			panic(err)
		}
	}

	//init mime
	// AddAPPStartHook函数可用于在应用启动之前执行一些hook
	// 最后一个hook是initMime
	AddAPPStartHook(initMime)

	// do hooks function
	// 逐个执行hook
	for _, hk := range hooks {
		err := hk()
		if err != nil {
			panic(err)
		}
	}

   // 如果配置启用session
	if SessionOn {
		var err error
		sessionConfig := AppConfig.String("sessionConfig")
		if sessionConfig == "" {
			sessionConfig = `{"cookieName":"` + SessionName + `",` +
				`"gclifetime":` + strconv.FormatInt(SessionGCMaxLifetime, 10) + `,` +
				`"providerConfig":"` + filepath.ToSlash(SessionSavePath) + `",` +
				`"secure":` + strconv.FormatBool(EnableHttpTLS) + `,` +
				`"enableSetCookie":` + strconv.FormatBool(SessionAutoSetCookie) + `,` +
				`"domain":"` + SessionDomain + `",` +
				`"cookieLifeTime":` + strconv.Itoa(SessionCookieLifeTime) + `}`
		}
		GlobalSessions, err = session.NewManager(SessionProvider,
			sessionConfig)
		if err != nil {
			panic(err)
		}
		go GlobalSessions.GC()
	}

   // 加载所有的模板文件
	// ViewsPath默认为views子目录
	err := BuildTemplate(ViewsPath)
	if err != nil {
		if RunMode == "dev" {
			Warn(err)
		}
	}

   // 注册默认的错误处理方法
	registerDefaultErrorHandler()

	if EnableDocs {
		Get("/docs", serverDocs)
		Get("/docs/*", serverDocs)
	}
}
```

`BeeApp.Run` 一句中的BeeApp来自 `config.go` 文件的 `init` 方法，调用 `NewApp()` 来初始化：

```go
// NewApp returns a new beego application.
func NewApp() *App {
   // 这玩意比较关键，所有的路由全在里面了
	cr := NewControllerRegister()
	app := &App{Handlers: cr, Server: &http.Server{}}
	return app
}
```

`BeeApp.Run` 实现如下所示：

```go
// Run beego application.
func (app *App) Run() {
	addr := HttpAddr

	if HttpPort != 0 {
		addr = fmt.Sprintf("%s:%d", HttpAddr, HttpPort)
	}

	var (
		err error
		l   net.Listener
	)
	endRunning := make(chan bool, 1)

   // 使用FastCGI来提供服务
	if UseFcgi {
		if UseStdIo {
			err = fcgi.Serve(nil, app.Handlers) // standard I/O
			if err == nil {
				BeeLogger.Info("Use FCGI via standard I/O")
			} else {
				BeeLogger.Info("Cannot use FCGI via standard I/O", err)
			}
		} else {
			if HttpPort == 0 {
				// remove the Socket file before start
				if utils.FileExists(addr) {
					os.Remove(addr)
				}
				l, err = net.Listen("unix", addr)
			} else {
				l, err = net.Listen("tcp", addr)
			}
			if err != nil {
				BeeLogger.Critical("Listen: ", err)
			}
			err = fcgi.Serve(l, app.Handlers)
		}
	} else {
	    // 优雅地启动关闭服务
		if Graceful {
			app.Server.Addr = addr
			app.Server.Handler = app.Handlers
			app.Server.ReadTimeout = time.Duration(HttpServerTimeOut) * time.Second
			app.Server.WriteTimeout = time.Duration(HttpServerTimeOut) * time.Second
			if EnableHttpTLS {
				go func() {
					time.Sleep(20 * time.Microsecond)
					if HttpsPort != 0 {
						addr = fmt.Sprintf("%s:%d", HttpAddr, HttpsPort)
						app.Server.Addr = addr
					}
					// 看到app.Handlers没？
					server := grace.NewServer(addr, app.Handlers)
					server.Server = app.Server
					err := server.ListenAndServeTLS(HttpCertFile, HttpKeyFile)
					if err != nil {
						BeeLogger.Critical("ListenAndServeTLS: ", err, fmt.Sprintf("%d", os.Getpid()))
						time.Sleep(100 * time.Microsecond)
						endRunning <- true
					}
				}()
			}
			if EnableHttpListen {
				go func() {
				    // 看到没？
					server := grace.NewServer(addr, app.Handlers)
					server.Server = app.Server
					if ListenTCP4 && HttpAddr == "" {
						server.Network = "tcp4"
					}
					err := server.ListenAndServe()
					if err != nil {
						BeeLogger.Critical("ListenAndServe: ", err, fmt.Sprintf("%d", os.Getpid()))
						time.Sleep(100 * time.Microsecond)
						endRunning <- true
					}
				}()
			}
		} else {
			app.Server.Addr = addr
			//
			app.Server.Handler = app.Handlers
			app.Server.ReadTimeout = time.Duration(HttpServerTimeOut) * time.Second
			app.Server.WriteTimeout = time.Duration(HttpServerTimeOut) * time.Second

			if EnableHttpTLS {
				go func() {
					time.Sleep(20 * time.Microsecond)
					if HttpsPort != 0 {
						app.Server.Addr = fmt.Sprintf("%s:%d", HttpAddr, HttpsPort)
					}
					BeeLogger.Info("https server Running on %s", app.Server.Addr)
					err := app.Server.ListenAndServeTLS(HttpCertFile, HttpKeyFile)
					if err != nil {
						BeeLogger.Critical("ListenAndServeTLS: ", err)
						time.Sleep(100 * time.Microsecond)
						endRunning <- true
					}
				}()
			}

			if EnableHttpListen {
				go func() {
					app.Server.Addr = addr
					BeeLogger.Info("http server Running on %s", app.Server.Addr)
					if ListenTCP4 && HttpAddr == "" {
						ln, err := net.Listen("tcp4", app.Server.Addr)
						if err != nil {
							BeeLogger.Critical("ListenAndServe: ", err)
							time.Sleep(100 * time.Microsecond)
							endRunning <- true
							return
						}
						err = app.Server.Serve(ln)
						if err != nil {
							BeeLogger.Critical("ListenAndServe: ", err)
							time.Sleep(100 * time.Microsecond)
							endRunning <- true
							return
						}
					} else {
						err := app.Server.ListenAndServe()
						if err != nil {
							BeeLogger.Critical("ListenAndServe: ", err)
							time.Sleep(100 * time.Microsecond)
							endRunning <- true
						}
					}
				}()
			}
		}

	}
	<-endRunning
}
```

对于三种服务启动方式，这三种方式都要求`app.Handlers`实现了`http.Handler`接口：

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```



