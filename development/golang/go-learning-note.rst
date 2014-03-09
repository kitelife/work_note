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
  
------
