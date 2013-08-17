Go语言
=========

语言基础
-------------

Go语言程序设计的一些规则
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Go语言之所以简洁，是因为它有一些默认的行为。

- 大写字母开头的变量是可导出的，即其他包可以读取，是公用变量；小写字母开头的不可导出，是私有变量。

- 大写字母开头的函数也是一样，相当于class中带public关键词的公有函数；小写字母开头就是有private关键词的私有函数。


Panic和Recover
^^^^^^^^^^^^^^^^^^^

Go语言没有像Java那样的异常机制，它不能抛出异常，而是使用了panic和recover机制。一定要记住，你应当把它作为最后的手段来使用，也就是说，你的代码中应当没有，或者很少有panic的东西。

**Panic**

panic是一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。当函数F调用panic，函数F的执行被中断，但是F中的延迟函数会正常执行，然后F返回到
调用它的地方。在调用的地方，F的行为就像调用了panic。这一过程继续向上，直到发生panic的goroutine中所有调用的函数返回，此时程序退出。恐慌可以直接调用
panic产生，也可以由运行时错误产生，例如访问越界的数组。

**Recover**

recover是一个内建函数，可以让进入令人恐慌的流程中的goroutine恢复过来。recover仅在延迟函数中有效。在正常的执行过程中，调用recover会返回nil，并且没有其他任何
效果。如果当前的goroutine陷入恐慌，调用recover可以捕获到panic的输入值，并且恢复正常的执行。


``main`` 函数和 ``init`` 函数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Go里面有两个保留的函数： ``init`` 函数（能够应用于所有的 ``package`` ）和 ``main`` 函数（只能应用于 ``package main`` ）。这两个函数在定义时不能有任何的参数和返回值。虽然一个 ``package`` 里面可以写任意多个 ``init`` 函数，但这无论是对于可读性还是以后的可维护性来说，我们都强烈建议用户在一个 ``package`` 中每个文件只写一个 ``init`` 函数。

Go程序会自动调用 ``init()`` 和 ``main()`` ，所以你不需要在任何地方调用这两个函数。每个 ``package`` 中的 ``init`` 函数都是可选的，但 ``package main`` 就必须包含一个 ``main`` 函数。

程序的初始化和执行都起始于 ``main`` 包。如果 ``main`` 包还导入了其它的包，那么就会在编译时将它们依次导入。有时一个包会被多个包同时导入，那么它只会被导入一次（例如很多包可能都会用到 ``fmt`` 包，但它只会被导入一次，因为没有必要导入多次）。当一个包被导入时，如果该包还导入了其它的包，那么会先将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行 ``init`` 函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开始对 ``main`` 包中的包级常量和变量进行初始化，然后执行`main`包中的 ``init`` 函数（如果存在的话），最后执行 ``main`` 函数。下图详细地解释了整个执行过程：

.. image:: https://raw.github.com/astaxie/build-web-application-with-golang/master/ebook/images/2.3.init.png

面向对象
^^^^^^^^^^^

method附属在一个给定的类型上，它的语法和函数的声明语法几乎一样，只是在func后面增加了一个receiver（也就是method所依从的主体）。

method的语法如下：
::

  func (r ReceiverType) funcName(parameters) (results)
  
::

	package main
	
	import (
		"fmt"
		"math"
  	)
  	
  	type Rectangle struct {
		width, height float64
  	}
  	
  	type Circle struct {
	  	radius float64
  	}
  	
  	func (r Rectangle) area() float64 {
	  	return r.width * r.height
  	}
  	
  	func (c Circle) area() float64 {
	  	return c.radius * c.radius * math.Pi
  	}
  	
  	func main() {
		r1 := Rectangle{12, 2}
	  	r2 := Rectangle{9, 4}
	  	c1 := Circle{10}
	  	c2 := Circle{25}
    
	  	fmt.Println("Area of r1 is: ", r1.area())
	  	fmt.Println("Area of r2 is: ", r2.area())
	  	fmt.Println("Area of c1 is: ", c1.area())
	  	fmt.Println("Area of c2 is: ", c2.area())
  	}
  
使用method的时候重要注意以下几点：

- 虽然method的名字一模一样，但是如果接收者不一样，那么method就不一样。
- method里面可以访问接收者的字段
- 调用method，就像struct里面访问字段一样。

.. seealso:: method的Receiver是以值传递还是以指针（引用）传递，两者的差别在于，指针作为Receiver会对实例对象的内容发生操作，而普通类型作为Receiver仅仅是以副本作为操作对象，并不对原实例对象发生操作。

method可以定义在任何你自定义的类型、内置类型、struct等各种类型上面。

**method继承**

如果匿名字段实现了一个method，那么包含这个匿名字段的struct也能调用该method。
::

	package main

	import "fmt"
	
	type Human struct {
		name  string
		age   int
		phone string
	}
	
	type Student struct {
		Human  // 匿名字段
		school string
	}
	
	type Employee struct {
		Human   // 匿名字段
		company string
	}
	
	// 在Human上面定义了一个method
	func (h *Human) SayHi() {
		fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
	}
	
	func main() {
		mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
		sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}
		
		mark.SayHi()
		sam.SayHi()
	}

**method重写**

interface
^^^^^^^^^^^^

简单地说，interface是一组method的组合，我们通过interface来定义对象的一组行为。

interface类型定义了一组方法，如果某个对象实现了某个接口的所有方法，则此对象就实现了此接口。

空interface（interface{}）不包含任何的method，正因为如此，所有的类型都实现了空interface。空interface对于描述起不到任何的作用（因为它不包含任何的method），但是空interface在我们需要存储任意类型的数值时相当有用，因为它可以存储任意类型的数值，有点类似于C语言的 ``void *`` 类型。

一个函数把interface{}作为参数，那么它可以接受任意类型的值作为参数，如果一个函数返回interface{}，就可以返回任意类型的值。非常有用！

**嵌入interface**

如果一个interface1作为interface2的一个嵌入字段，那么interface2隐式地包含了interface1里面的method。

**反射**

Go语言实现了反射，所谓反射就是动态运行时的状态。我们一般用到的包是reflect包。

`The Laws of Reflection <http://golang.org/doc/articles/laws_of_reflection.html>`_

并发
^^^^^^^

**goroutine**

goroutine是Go语言并行设计的核心。goroutine说到底就是线程，但是它比线程更小，十几个goroutine可能体现在底层就是五六个线程，Go语言内部帮你实现了这些goroutine之间的内存共享。

**channels**

goroutine运行在相同的地址空间，因此访问共享内存必须做好同步。goroutine之间如何进行数据的通信？Go语言提供了一个很好的通信机制channel。channel可以与Unix shell中的双向管道做类比，通过它发送或接收值。这些值只能是特定的类型：channel类型。定义一个channel时，也需要定义发送到channel的值的类型。注意，必须使用make创建channel。
::

    ci := make(chan int)
    cs := make(chan string)
    cf := make(chan interface{})

channel通过操作符<-来接收和发送数据。
::

    ch <- v    // 发送v到channel ch。
    v := <-ch    // 从ch中接收数据，并赋值给v

标准库
---------


第三方库
-----------


Web框架
------------

- `Revel <http://robfig.github.io/revel/>`_
- `beego <http://beego.me/>`_

推荐阅读
-----------

- `Golang标准库 <https://github.com/polaris1119/The-Golang-Standard-Library-by-Example>`_
- `Go步行者 <http://gowalker.org/>`_
- `GoDoc <http://godoc.org/>`_
- `一步一步学习Revel Web开源框架 <http://www.cnblogs.com/ztiandan/archive/2013/01/17/2864498.html>`_
- `build-web-application-with-golang <https://github.com/astaxie/build-web-application-with-golang>`_
- `gopkg <https://github.com/astaxie/gopkg>`_
- `Network programming with Go <http://jan.newmarch.name/go/>`_
- `A Introduction to Programming in Go <http://www.golang-book.com/>`_
