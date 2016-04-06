# 路由管理

根据Beego官方文档 http://beego.me/docs/mvc/controller/router.md ，注册路由的常用方法有：

- beego.Get
- beego.Post
- beego.Put
- beego.Head
- beego.Options
- beego.Delete
- beego.Any

- beego.Router

我们来看看这些方法的实现。

```go
// Get used to register router for Get method
// usage:
//    beego.Get("/", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Get(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Get(rootpath, f)
	return BeeApp
}

// Post used to register router for Post method
// usage:
//    beego.Post("/api", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Post(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Post(rootpath, f)
	return BeeApp
}

// Delete used to register router for Delete method
// usage:
//    beego.Delete("/api", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Delete(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Delete(rootpath, f)
	return BeeApp
}

// Put used to register router for Put method
// usage:
//    beego.Put("/api", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Put(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Put(rootpath, f)
	return BeeApp
}

// Head used to register router for Head method
// usage:
//    beego.Head("/api", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Head(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Head(rootpath, f)
	return BeeApp
}

// Options used to register router for Options method
// usage:
//    beego.Options("/api", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Options(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Options(rootpath, f)
	return BeeApp
}

// Patch used to register router for Patch method
// usage:
//    beego.Patch("/api", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Patch(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Patch(rootpath, f)
	return BeeApp
}

// Any used to register router for all methods
// usage:
//    beego.Any("/api", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func Any(rootpath string, f FilterFunc) *App {
	BeeApp.Handlers.Any(rootpath, f)
	return BeeApp
}
```

```go
// Router adds a patterned controller handler to BeeApp.
// it's an alias method of App.Router.
// usage:
//  simple router
//  beego.Router("/admin", &admin.UserController{})
//  beego.Router("/admin/index", &admin.ArticleController{})
//
//  regex router
//
//  beego.Router("/api/:id([0-9]+)", &controllers.RController{})
//
//  custom rules
//  beego.Router("/api/list",&RestController{},"*:ListFood")
//  beego.Router("/api/create",&RestController{},"post:CreateFood")
//  beego.Router("/api/update",&RestController{},"put:UpdateFood")
//  beego.Router("/api/delete",&RestController{},"delete:DeleteFood")
func Router(rootpath string, c ControllerInterface, mappingMethods ...string) *App {
	BeeApp.Handlers.Add(rootpath, c, mappingMethods...)
	return BeeApp
}
```

可以看到这些方法都是直接调用 `BeeApp.Handlers` 的相关方法来注册路由的。`BeeApp.Handlers` 类型为：

```go
// ControllerRegister containers registered router rules, controller handlers and filters.
type ControllerRegister struct {
	routers      map[string]*Tree
	enableFilter bool
	filters      map[int][]*FilterRouter
	pool         sync.Pool
}
```

`ControllerRegister` 的 `Get`、`Post`等方法都是调用其`AddMethod`方法来注册路由，如 `Get` 方法的实现：

```go
// Get add get method
// usage:
//    Get("/", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func (p *ControllerRegister) Get(pattern string, f FilterFunc) {
	p.AddMethod("get", pattern, f)
}
```

`AddMethod`的实现为：

```go
// AddMethod add http method router
// usage:
//    AddMethod("get","/api/:id", func(ctx *context.Context){
//          ctx.Output.Body("hello world")
//    })
func (p *ControllerRegister) AddMethod(method, pattern string, f FilterFunc) {
   method = strings.ToUpper(method)
	if _, ok := HTTPMETHOD[method]; method != "*" && !ok {
		panic("not support http method: " + method)
	}
	// f 最终会被封装到一个controllerInfo中
	route := &controllerInfo{}
	route.pattern = pattern
	// routerType 估计在路由查找时用得到
	route.routerType = routerTypeRESTFul
	route.runFunction = f
	methods := make(map[string]string)
	if method == "*" {
		for _, val := range HTTPMETHOD {
		   // 值val的含义是请求处理的方法名
			methods[val] = val
		}
	} else {
		methods[method] = method
	}
	// 当前这个路由支持的HTTP Method有哪些
	route.methods = methods
	for k := range methods {
		if k == "*" {
			for _, m := range HTTPMETHOD {
			    // 这才开始注册呢
				p.addToRouter(m, pattern, route)
			}
		} else {
			p.addToRouter(k, pattern, route)
		}
	}
}
```

代码片段中 `controllerInfo` 的定义为：

```go
type controllerInfo struct {
	pattern        string
	controllerType reflect.Type
	methods        map[string]string
	handler        http.Handler
	runFunction    FilterFunc
	routerType     int
}
```

------

`Add` 与 `Get`、`Post`等方法的实现不同：

```go
// Add controller handler and pattern rules to ControllerRegister.
// usage:
//	default methods is the same name as method
//	Add("/user",&UserController{})
//	Add("/api/list",&RestController{},"*:ListFood")
//	Add("/api/create",&RestController{},"post:CreateFood")
//	Add("/api/update",&RestController{},"put:UpdateFood")
//	Add("/api/delete",&RestController{},"delete:DeleteFood")
//	Add("/api",&RestController{},"get,post:ApiFunc"
//	Add("/simple",&SimpleController{},"get:GetFunc;post:PostFunc")
func (p *ControllerRegister) Add(pattern string, c ControllerInterface, mappingMethods ...string) {
    
    // ValueOf returns a new Value initialized to the concrete value
    // stored in the interface i.  ValueOf(nil) returns the zero Value.
	reflectVal := reflect.ValueOf(c)
	
	// Indirect returns the value that v points to.
   // If v is a nil pointer, Indirect returns a zero Value.
   // If v is not a pointer, Indirect returns v.
   // Type returns v's type.
	t := reflect.Indirect(reflectVal).Type()
	methods := make(map[string]string)
	if len(mappingMethods) > 0 {
	   // semi 是一组路由配置
		semi := strings.Split(mappingMethods[0], ";")
		for _, v := range semi {
		  // colon 是一个 verb => method 映射
			colon := strings.Split(v, ":")
			if len(colon) != 2 {
				panic("method mapping format is invalid")
			}
			// comma 是 HTTP Verb 组
			comma := strings.Split(colon[0], ",")
			for _, m := range comma {
			     // 先检测 HTTP Verb 是否有效
				if _, ok := HTTPMETHOD[strings.ToUpper(m)]; m == "*" || ok {
				    // 通过反射以字符串获取对应的方法，看看是否为有效的目标方法
					if val := reflectVal.MethodByName(colon[1]); val.IsValid() {
					   // 为毛不直接存储 val，而是存 colon[1]
						methods[strings.ToUpper(m)] = colon[1]
					} else {
						panic("'" + colon[1] + "' method doesn't exist in the controller " + t.Name())
					}
				} else {
					panic(v + " is an invalid method mapping. Method doesn't exist " + m)
				}
			}
		}
	}

	route := &controllerInfo{}
	route.pattern = pattern
	//
	route.methods = methods
	// 这个地方与 AddMethod 中的不同
	route.routerType = routerTypeBeego
	// 还多了这一行
	// 存了当前路由到底是哪个controller类的实例
	route.controllerType = t
	
	// 如果并未提供mappingMethods，则使用默认的 GET -> Get，POST -> Post, ...
	if len(methods) == 0 {
		for _, m := range HTTPMETHOD {
			p.addToRouter(m, pattern, route)
		}
	} else {
		for k := range methods {
			if k == "*" {
				for _, m := range HTTPMETHOD {
					p.addToRouter(m, pattern, route)
				}
			} else {
				p.addToRouter(k, pattern, route)
			}
		}
	}
}
```

代码片段中 `ControllerInterface` 定义为：

```go
// ControllerInterface is an interface to uniform all controller handler.
type ControllerInterface interface {
	Init(ct *context.Context, controllerName, actionName string, app interface{})
	Prepare()
	Get()
	Post()
	Delete()
	Put()
	Head()
	Patch()
	Options()
	Finish()
	Render() error
	XSRFToken() string
	CheckXSRFCookie() bool
	HandlerFunc(fn string) bool
	URLMapping()
}
```

------

可以看到 `Add` 和 `AddMethod` 方法最终都是调用 `addToRouter` 来注册路由：

```go
func (p *ControllerRegister) addToRouter(method, pattern string, r *controllerInfo) {
    // 如果配置路由大小写不敏感，则统一转成小写
	if !BConfig.RouterCaseSensitive {
		pattern = strings.ToLower(pattern)
	}
	// 从这里可以看出，路由是分层次管理的，先按method，再看pattern
	if t, ok := p.routers[method]; ok {
		t.AddRouter(pattern, r)
	} else {
		t := NewTree()
		t.AddRouter(pattern, r)
		p.routers[method] = t
	}
}
```

代码中 `NewTree()` 返回的数据类型为：

```go
// Tree has three elements: FixRouter/wildcard/leaves
// fixRouter sotres Fixed Router
// wildcard stores params
// leaves store the endpoint information
type Tree struct {
	//prefix set for static router
	prefix string
	//search fix route first
	fixrouters []*Tree
	//if set, failure to match fixrouters search then search wildcard
	wildcard *Tree
	//if set, failure to match wildcard search
	leaves []*leafInfo
}

// NewTree return a new Tree
func NewTree() *Tree {
	return &Tree{}
}
```

`Tree` 的 `AddRouter` 方法实现如下：

```go
// AddRouter call addseg function
func (t *Tree) AddRouter(pattern string, runObject interface{}) {
	t.addseg(splitPath(pattern), runObject, nil, "")
}

// "/"
// "admin" ->
func (t *Tree) addseg(segments []string, route interface{}, wildcards []string, reg string) {
	if len(segments) == 0 {
	   // 叶子节点
		if reg != "" {
			t.leaves = append(t.leaves, &leafInfo{runObject: route, wildcards: wildcards, regexps: regexp.MustCompile("^" + reg + "$")})
		} else {
			t.leaves = append(t.leaves, &leafInfo{runObject: route, wildcards: wildcards})
		}
	} else {
		seg := segments[0]
		// 看看 pattern 的当前部分是否包含正则或特殊字符
		iswild, params, regexpStr := splitSegment(seg)
		// if it's ? meaning can igone this, so add one more rule for it
		// 如果第一个param的第一个元素是 : ，则表示当前部分允许不匹配
		if len(params) > 0 && params[0] == ":" {
			t.addseg(segments[1:], route, wildcards, reg)
			params = params[1:]
		}
		//Rule: /login/*/access match /login/2009/11/access
		//if already has *, and when loop the access, should as a regexpStr
		if !iswild && utils.InSlice(":splat", wildcards) {
			iswild = true
			// 
			regexpStr = seg
		}
		//Rule: /user/:id/*
		if seg == "*" && len(wildcards) > 0 && reg == "" {
			regexpStr = "(.+)"
		}
		if iswild {
			if t.wildcard == nil {
			   // 子树
				t.wildcard = NewTree()
			}
			if regexpStr != "" {
				if reg == "" {
					rr := ""
					for _, w := range wildcards {
						if w == ":splat" {
							rr = rr + "(.+)/"
						} else {
							rr = rr + "([^/]+)/"
						}
					}
					regexpStr = rr + regexpStr
				} else {
					regexpStr = "/" + regexpStr
				}
			} else if reg != "" {
				if seg == "*.*" {
					regexpStr = "/([^.]+).(.+)"
					params = params[1:]
				} else {
					for range params {
						regexpStr = "/([^/]+)" + regexpStr
					}
				}
			} else {
				if seg == "*.*" {
					params = params[1:]
				}
			}
			//
			t.wildcard.addseg(segments[1:], route, append(wildcards, params...), reg+regexpStr)
		} else {
			var ok bool
			var subTree *Tree
			for _, subTree = range t.fixrouters {
				if t.prefix == seg {
					ok = true
					break
				}
			}
			if !ok {
				subTree = NewTree()
				subTree.prefix = seg
				t.fixrouters = append(t.fixrouters, subTree)
			}
			//
			subTree.addseg(segments[1:], route, wildcards, reg)
		}
	}
}
```


