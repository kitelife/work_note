第一章 类型
---------------

引用类型包括slice、map和channel。它们有复杂的内部结构，除了申请内存外，还需要初始化相关属性。

内置函数new计算类型大小，为其分配零值内存，返回指针。而make会被编译器翻译成具体的创建函数，由其分配内存和初始化成员结构，返回对象而非指针。

------

不支持隐式类型转换，即便是从窄向宽转换也不行。

不能将其他类型的值当作bool值使用。

------

字符串是不可变值类型，内部用指针指向UTF-8字节数组。

- 默认值是空字符串""。
- 用索引号访问某字节，如s[i]。
- 不能用序号获取字节元素指针。&s[i]非法。
- 不可变类型，无法修改字节数组。
- 字节数组尾部不包含NULL。

要修改字符串，可先将其转换成``[]rune``或``[]byte``，完成后再转换为``string``。无论哪种转换，都会重新分配内存，并复制字节数组。
::

  s := "abcd"
  bs := []byte(s)
  bs[1] = 'B'
  s2 := string(bs)

  u := "电脑"
  us := []rune(u)
  us[1] = '话'
  u2 := string(us)

用for循环遍历字符串时，也有byte和rune两种方式。
::

  func main() {
    s := "abc汉字"

    for i := 0; i < len(s); i++ { // byte
      fmt.Printf("%c,", s[i])
    }

    fmt.Println()

    for _, r := range s {     // rune
      fmt.Printf("%c,", r)
    }
  }

输出：
::

  a,b,c,æ,±,,å,­,,
  a,b,c,汉,字,

------

可以在unsafe.Pointer和任意类型指针间进行转换。
::

  func main() {
    x := 0x12345678

    p := unsafe.Pointer(&x)   // *int -> Pointer
    n := (*[4]byte)(p)    // Pointer -> *[4]byte

    for i := 0; i < len(n); i++ {
      fmt.Println("%X", n[i])
    }
  }


输出：
::

  78 56 34 12

------

可将类型分为命名和未命名两大类。命名类型包括bool、int、string等，而array、slice、map等和具体元素类型、长度等有关，属于未命名类型。

具有相同声明的未命名类型被视为同一类型。

- 具有相同基类型的指针。
- 具有相同元素类型和长度的array。
- 具有相同元素类型的slice。
- 具有相同键值类型的map。
- 具有相同元素类型和传送方向的channe。
- 具有相同字段序列（字段名、类型、标签、顺序）的匿名struct。
- 签名相同（参数和返回值，不包括参数名称）的function。
- 方法集相同（方法名、方法签名相同、和次序无关）的interface。

------

第二章 表达式
--------------------

switch分支表达式可以是任意类型，不限于常量。可省略break，默认自动终止。如需要继续下一分支，需使用fallthrough。
::

  x := []int{1, 2, 3}
  i := 2

  switch i {
    case x[1]:
      println("a")
      fallthrough
    case 1, 3:
      println("b")
    default:
      println("c")
  }


输出：
::

  a
  b

省略条件表达式，可当if...else if...else使用。
::

  switch {
    case x[1] > 0:
      println("a")
    case x[1] < 0:
      println("b")
    default:
      println("c")
  }

  switch i := x[2]; {  // 带初始化语句
    case i > 0:
      println("a")
    case i < 0:
      println("b")
    default:
      println("c")
  }

支持函数内goto跳转。表签名区分大小写，未使用的标签会引发错误。

break可用于for、switch、select，而continue仅能用于for循环。

第三章 函数
-----------------

函数定义不支持嵌套（nested）、重载（overload）和默认参数（default parameter）。

- 无需声明原型
- 支持不定长变参
- 支持多返回值
- 支持命名返回参数
- 支持匿名函数和闭包

函数是第一类对象，可作为参数传递。建议将复杂签名定义为函数类型，以便于阅读。
::

  func test(fn func() int) int {
    return fn()
  }

  type FormatFunc func(s string, x, y int) string     // 定义函数原型
  func format(fn FormatFunc, s string, x, y int) string {
    return fn(s, x, y)
  }

  func main() {
    s1 := test(func() int { return 100 })     // 直接将匿名函数当参数

    s2  := format(func(s string, x, y int) string {
      return fmt.Sprintf(s, x, y)
    }, " %d, %d", 10, 20)

    println(s1, s2)
  }

有返回值的函数，必须有明确的终⽌语句，否则会引发编译错误。

------

变参本质上就是slice。只能有一个，且必须是最后一个。

------

不能用容器对象接收多返回值。只能用多个变量，或“_”忽略。

多返回值可直接作为其他函数的调用实参。

命名返回参数允许defer延迟调用通过闭包读取和修改。
::

  func add(x, y int) (z int) {
    defer func() {
      z += 100
    }()

    z = x + y
    return
  }

  func main() {
    println(add(1, 2))    // 输出：103
  }

显式return返回前，会先修改命名返回参数。
::

  func add(x, y int) (z int) {
    defer func() {
      println(z)      // 输出：203
    }()

    z = x + y
    return z + 200        // 执行顺序：(z = z + 200) -> (call defer) -> (ret)
  }

  func main() {
    println(add(1, 2))      // 输出：203
  }

------

匿名函数可赋值给变量，作为结构字段，或者在channel里传送。
::

  // function variable

  fn := func() { println("Hello World!") }
  fn()

  // function collection
  fns := [](func(x int) int){
    func(x int) int { return x + 1 },
    func(x int) int { return x + 2 },
  }

  println(fns[0](100))

  // function as field
  d := struct {
    fn func() string
  }{
    fn: func() string { return "Hello, World!" },
  }

  println(d.fn())

  // channel of function
  fc := make(chan func() string, 2)
  fc <- func() string { return "Hello, World!" }
  println((<-fc)())

------

闭包复制的是原对象指针，这就很容易解释延迟引用现象。
::

  func test() func() {
    x := 100
    fmt.Printf("x (%p) = %d\n", &x, x)

    return func() {
      fmt.Printf("x (%p) = %d\n", &x, x)
    }
  }

  func main() {
    f := test()
    f()
  }

输出：
::

  x (0x2101ef018) = 100
  x (0x2101ef018) = 100

------

关键字defer用于注册延迟调用。这些调用直到ret前才被执行，通常用于释放资源或错误处理。

多个defer注册，按FILO次序执行。哪怕函数或某个延迟调用发生错误，这些调用依旧会被执行。

如果需要保护代码片段，可将代码块重构成匿名函数，如此可确保后续代码被执行。
::

  func test(x, y int) {
    var z int

    func() {
      defer func() {
        if recover() != nil { z = 0 }
      }()

      z = x / y
      return
    }()

    println("x / y =", z)
  }

------


第四章 数据
----------------

Array
^^^^^^^^

和以往认知的数组有很大不同。

- 数组是值类型，赋值和传参会复制整个数组，而不是指针。
- 数组长度必须是常量，且是类型的组成部分。[2]int和[3]int是不同类型
- 支持“==”、"!="操作符，因为内存总是被初始化过的。
- 指针数组[n]\*T，数组指针\*[n]T。

值拷贝行为会造成性能问题，通常会建议使用slice，或者数组指针。

内置函数len和cap都返回数组长度（元素数量）。

Slice
^^^^^^^^^

slice并不是数组或者数组指针。它通过内部指针和相关属性引用数组片段，以实现变长方案。
::

  struct Slice
  {               // must not move anything
    byte* array;  // actual data
    uintgo  len;  // number of elements
    uintgo  cap;  // allocated number of elements
  }

- 引用类型。但自身是结构体，值拷贝传递。
- 属性len标识可用元素数量，读写操作不能超过该限制。
- 属性cap标识最大扩张容量，不能超过数组限制。
- 如果slice == nil，那么len、cap结果都等于0。

大批量添加数据时，应该避免使用append，因为频繁创建slice对象会影响性能。**可一次性分配len足够大的slice，直接用索引号进行操作。**
还有，**及时释放不再使用的slice对象**，避免持有过期数组，造成GC无法回收。

Map
^^^^^^

map是引用类型，哈希表。键必须是支持相等运算符(==、!=)类型，比如number、string、pointer、array、struct，以及对应的interface。
值可以是任意类型，没有限制。


Struct
^^^^^^^^^^^

值类型，赋值和传参会复制全部内容，可用“_”定义补位字段(?)，支持指向自身类型的指针成员。

支持“==”、“!=”相等操作符，可用作map键类型。

可定义字段标签，用反射读取。标签是类型的组成部分。
::

  var u1 struct { name string "username" }
  var u2 struct { name string }

  u2 = u1   // Error: cannot use u1 (type struct { name string "username" }) as
            //        type struct { name string } in assignment


**匿名字段**

匿名字段不过是一种语法糖，从根本上说，就是一个与成员类型同名（不包含包名）的字段。被匿名嵌入的可以是任何类型，当然也包括指针。
::

  type User struct {
    name string
  }

  type Manager struct {
    User
    title string
  }

  m := Manager{
    User: User{"Tom"},    // 匿名字段的显式字段名，和类型名相同。
    title: "Administrator",
  }

可以像普通字段那样访问匿名字段成员，编译器从外向内逐级查找所有层次的匿名字段，直到发现目标或出错。
::

  type Resource struct {
    id int
  }

  type User struct {
    Resource
    name string
  }

  type Manager struct {
    User
    title string
  }

  var m Manager
  m.id = 1
  m.name = "Jack"
  m.title = "Administrator"

外层同名字段会遮蔽嵌入字段成员，相同层次的同名字段也会让编译器无所适从。解决⽅法是使用显式字段名。


第五章 方法
-----------------

可以像字段成员那样访问匿名字段方法，编译器负责查找。
::

  type User struct {
    id int
    name string
  }

  type Manager struct {
    User
  }

  func (self *User) ToString() string {   // receiver = &(Manager.User)
    return fmt.Sprintf("User: %p, %v", self, self)
  }

  func main() {
    m := Manager{User{1, "Tom"}}

    fmt.Printf("Manager: %p\n", &m)

    fmt.Println(m.ToString())
  }

输出：
::

  Manager: 0x2102281b0
  User: 0x2102281b0, &{1 Tom}


通过匿名字段，可获得和继承类似的复用能力。依据编译器查找次序，只需在外层定义同名方法，就可以实现"override"。
::

  type User struct {
    id int
    name string
  }

  type Manager struct {
    User
    title string
  }

  func (self *User) ToString() string {
    return fmt.Sprintf("User: %p, %v", self, self)
  }

  func (self *Manager) ToString() string {
    return fmt.Sprintf("Manager: %p, %v", self, self)
  }

  func main() {
    m := Manager{User{1, "Tom"}, "Administrator"}

    fmt.Println(m.ToString())
    fmt.Println(m.User.ToString())
  }

输出：
::

  Manager: 0x2102271b0, &{{1 Tom} Administrator}
  User: 0x2102271b0, &{1 Tom}


方法集
^^^^^^^^

每个类型都有与之关联的方法集，这会影响到接口实现规则。

- 类型T方法集包含全部receiver T方法
- 类型\*T方法集包含全部receiverT + \*T方法。
- 如类型S包含匿名字段T，则S方法集包含T方法。
- 如类型S包含匿名方法\*T，则S方法集包含T + \*T的方法。
- 不管嵌入T或\*T，\*S方法集总是包含T + \*T方法。


第六章 接口
------------------

接口是一个或多个方法签名的集合，任何类型的方法集中只要拥有与之对应的全部方法，就表示它“实现”了该接口，无需在该类型上显式添加接口声明。

- 接口命名习惯以er结尾，结构体。
- 接口只有方法签名，没有实现。
- 接口没有数据字段。
- 可在接口中嵌入其他接口。
- 类型可实现多个接口。

空接口interface{}没有任何方法签名，也就意味着任何类型都实现了空接口。其作用类似面向对象语言中的根对象object。

匿名接口可用作变量类型，或结构成员。
::

  type Tester struct {
    s interface {
      String() string
    }
  }

  type User struct {
    id int
    name string
  }

  func (self *User) String() string {
    return fmt.Sprintf("user %d, %s", self.id, self.name)
  }

  func main() {
    t := Tester{&User{1, "Tom"}}
    fmt.Println(t.s.String())
  }

输出：
::

  user 1, Tom


某些时候，让函数直接“实现”接口能省不少事。
::

  type Tester interface {
    Do()
  }

  type FuncDo func()
  func (self FunDo) Do() { self() }

  func main() {
    var t Tester = FuncDo(func() { println("Hello, World!") })
    t.Do()
  }


第七章 并发
--------------

