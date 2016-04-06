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
// beego.Run("localhost")
// beego.Run(":8089")
// beego.Run("127.0.0.1:8089")
func Run(params ...string) {
	initBeforeHTTPRun()

	if len(params) > 0 && params[0] != "" {
		strs := strings.Split(params[0], ":")
		if len(strs) > 0 && strs[0] != "" {
			BConfig.Listen.HTTPAddr = strs[0]
		}
		if len(strs) > 1 && strs[1] != "" {
			BConfig.Listen.HTTPPort, _ = strconv.Atoi(strs[1])
		}
	}

	BeeApp.Run()
}
```

其中 `initBeforeHttpRun()` 实现如下：

```go
func initBeforeHTTPRun() {
	// if AppConfigPath is setted or conf/app.conf exist
	err := ParseConfig()
	if err != nil {
		panic(err)
	}
	//init log
	for adaptor, config := range BConfig.Log.Outputs {
		err = BeeLogger.SetLogger(adaptor, config)
		if err != nil {
			fmt.Printf("%s with the config `%s` got err:%s\n", adaptor, config, err)
		}
	}

	SetLogFuncCall(BConfig.Log.FileLineNum)

	//init hooks
	AddAPPStartHook(registerMime)
	AddAPPStartHook(registerDefaultErrorHandler)
	AddAPPStartHook(registerSession)
	AddAPPStartHook(registerDocs)
	AddAPPStartHook(registerTemplate)
	AddAPPStartHook(registerAdmin)

	for _, hk := range hooks {
		if err := hk(); err != nil {
			panic(err)
		}
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
	addr := BConfig.Listen.HTTPAddr

	if BConfig.Listen.HTTPPort != 0 {
		addr = fmt.Sprintf("%s:%d", BConfig.Listen.HTTPAddr, BConfig.Listen.HTTPPort)
	}

	var (
		err        error
		l          net.Listener
		endRunning = make(chan bool, 1)
	)

	// run cgi server
	if BConfig.Listen.EnableFcgi {
		if BConfig.Listen.EnableStdIo {
		    // app.Handlers
			if err = fcgi.Serve(nil, app.Handlers); err == nil { // standard I/O
				BeeLogger.Info("Use FCGI via standard I/O")
			} else {
				BeeLogger.Critical("Cannot use FCGI via standard I/O", err)
			}
			return
		}
		if BConfig.Listen.HTTPPort == 0 {
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
		if err = fcgi.Serve(l, app.Handlers); err != nil {
			BeeLogger.Critical("fcgi.Serve: ", err)
		}
		return
	}

	app.Server.Handler = app.Handlers
	app.Server.ReadTimeout = time.Duration(BConfig.Listen.ServerTimeOut) * time.Second
	app.Server.WriteTimeout = time.Duration(BConfig.Listen.ServerTimeOut) * time.Second

	// run graceful mode
	if BConfig.Listen.Graceful {
		httpsAddr := BConfig.Listen.HTTPSAddr
		app.Server.Addr = httpsAddr
		if BConfig.Listen.EnableHTTPS {
			go func() {
				time.Sleep(20 * time.Microsecond)
				if BConfig.Listen.HTTPSPort != 0 {
					httpsAddr = fmt.Sprintf("%s:%d", BConfig.Listen.HTTPSAddr, BConfig.Listen.HTTPSPort)
					app.Server.Addr = httpsAddr
				}
				server := grace.NewServer(httpsAddr, app.Handlers)
				server.Server.ReadTimeout = app.Server.ReadTimeout
				server.Server.WriteTimeout = app.Server.WriteTimeout
				if err := server.ListenAndServeTLS(BConfig.Listen.HTTPSCertFile, BConfig.Listen.HTTPSKeyFile); err != nil {
					BeeLogger.Critical("ListenAndServeTLS: ", err, fmt.Sprintf("%d", os.Getpid()))
					time.Sleep(100 * time.Microsecond)
					endRunning <- true
				}
			}()
		}
		if BConfig.Listen.EnableHTTP {
			go func() {
				server := grace.NewServer(addr, app.Handlers)
				server.Server.ReadTimeout = app.Server.ReadTimeout
				server.Server.WriteTimeout = app.Server.WriteTimeout
				if BConfig.Listen.ListenTCP4 {
					server.Network = "tcp4"
				}
				if err := server.ListenAndServe(); err != nil {
					BeeLogger.Critical("ListenAndServe: ", err, fmt.Sprintf("%d", os.Getpid()))
					time.Sleep(100 * time.Microsecond)
					endRunning <- true
				}
			}()
		}
		<-endRunning
		return
	}

	// run normal mode
	app.Server.Addr = addr
	if BConfig.Listen.EnableHTTPS {
		go func() {
			time.Sleep(20 * time.Microsecond)
			if BConfig.Listen.HTTPSPort != 0 {
				app.Server.Addr = fmt.Sprintf("%s:%d", BConfig.Listen.HTTPSAddr, BConfig.Listen.HTTPSPort)
			}
			BeeLogger.Info("https server Running on %s", app.Server.Addr)
			if err := app.Server.ListenAndServeTLS(BConfig.Listen.HTTPSCertFile, BConfig.Listen.HTTPSKeyFile); err != nil {
				BeeLogger.Critical("ListenAndServeTLS: ", err)
				time.Sleep(100 * time.Microsecond)
				endRunning <- true
			}
		}()
	}
	if BConfig.Listen.EnableHTTP {
		go func() {
			app.Server.Addr = addr
			BeeLogger.Info("http server Running on %s", app.Server.Addr)
			if BConfig.Listen.ListenTCP4 {
				ln, err := net.Listen("tcp4", app.Server.Addr)
				if err != nil {
					BeeLogger.Critical("ListenAndServe: ", err)
					time.Sleep(100 * time.Microsecond)
					endRunning <- true
					return
				}
				if err = app.Server.Serve(ln); err != nil {
					BeeLogger.Critical("ListenAndServe: ", err)
					time.Sleep(100 * time.Microsecond)
					endRunning <- true
					return
				}
			} else {
				if err := app.Server.ListenAndServe(); err != nil {
					BeeLogger.Critical("ListenAndServe: ", err)
					time.Sleep(100 * time.Microsecond)
					endRunning <- true
				}
			}
		}()
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



