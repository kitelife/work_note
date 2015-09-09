## C学习笔记

字符常量默认是一个int整数，但编译器可以自行决定将其解释为char或int。

如：

``` c
char c = 'a';
printf("%c, size(char)=%d, size('a')=%d;\n", c, sizeof(c), sizeof('a'));
```

输出为：

``` c
a, size(char)=1, size('a')=4;
```

浮点数默认类型是double，可以添加后缀F来表示float，L表示long double

------

字面值（literal）是源代码中用来描述固定值的记号（token），可能是整数、浮点数、字符、字符串。

字符常量默认是int类型，除非用前置L表示 wchar_t 宽字符类型。

``` c
char c = 0x61;
char c2 = 'a';
char c3 = '\x61';

printf("%c, %c, %c\n", c, c2, c3);
```

输出为：

``` 
a, a, a
```

在Linux系统中，默认字符集是UTF-8，可以用 wctomb 等函数进行转换。

wchar_t 默认是4字节长度，足以容纳所有 UCS-4 Unicode 字符。

``` c
setlocale(LC_CTYPE, "en_US.UTF-8");

wchar_t wc = L'中';
// {} 表示什么意思？
char buf[100] = {};

int len = wctomb(buf, wc);
printf("%d\n", len);

for (int i = 0; i < len; i++)
{
    printf("0x%02X", (unsigned char)buf[i]);
}
```

输出为：

``` 
3
0xE4 0xB8 0xAD
```

C语言中的字符串是一个以NULL（也就是\0）结尾的char数组。

空字符串在内存中占用一个字节，包含一个NULL字符，也就是说要表示一个长度为1的字符串最少需要2个字节（strlen和sizeof表示的含义不同）。

------

当运算符的几个操作数类型不同时，就需要进行类型转换。通常编译器会做某些自动的隐式转换操作，在不丢失信息的前提下，将位宽“窄”的操作数转换成“宽”类型。

其他隐式转换还包括：

- 赋值和初始化时，右操作数总是被转换成左操作数类型
- 函数调用时，总是将实参转换为形参类型
- 将return表达式的结果转换为函数返回类型
- 任何类型0值和NULL指针都视为 false ，反之为 true

------

C99新增的内容，我们可以直接用以下语法声明一个结构或数组指针：

``` 
(类型名称){初始化列表}
```

演示：

``` c
int* i = &(int){ 123 };             // 整型变量，指针
int* x = (int[]){ 1, 2, 3, 4};      // 数组，指针
struct data_t* data = &(struct data_t){ .x = 123 };     // 结构，指针
func(123, &(struct data_t){ .x = 123 });                // 函数参数，结构指针参数
```

如果是静态或全局变量，那么初始化列表必须是编译期变量。

sizeof：返回操作数占用内存空间大小，单位字节（byte）。sizeof返回值是size_t类型，操作数可以是类型或变量。

逗号运算符是一个二元运算符，确保操作数从左到右被顺序处理，并返回右操作数的值和类型。

如：

``` c
int i = 1;
long long x = (i++, (long long)i);
printf("%lld\n", x);
```

输出为：

``` 
2
```

无条件跳转：break, continue, goto, return。

goto仅在函数内跳转，常用于跳出嵌套循环。如果在函数外跳转，可使用longjmp。

setjmp将当前位置的相关信息（堆栈帧、寄存器等）保存到jmp_buf结构中，并返回0。当后续代码执行longjmp跳转时，需要提高一个状态码。代码执行时将返回setjmp处，并返回longjmp所提供的状态码。

示例：

``` c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <setjmp.h>

void test(jmp_buf *env)
{
    printf("1....\n");
    longjmp(*env, 10);
}

int main(int argc, char* argv[])
{
    jmp_buf env;
    int ret = setjmp(env);      // 执行 longjmp 将返回该位置，ret 等于 longjmp 提供的状态码。

    if (ret == 0)
    {
        test(&env);
    }
    else
    {
        printf("2....(%d)\n", ret);
    }

    return EXIT_SUCCESS;
}
```

注意区分定义“函数类型”和“函数指针 类型”的区别。函数名是一个指向当前函数的指针。

如：

``` c
#include <stdio.h>
#include <stdlib.h>

typedef void(func_t)();          // 函数类型
typedef void(*func_ptr_t)();     // 函数指针类型

void test()
{
    printf("%s\n", __func__);
}

int main(int argc, char* argv[])
{
    func_t* func = test;         // 声明一个指针
    func_ptr_t func2 = test;     // 已经是指针类型
    void (*func3)();             // 声明⼀一个包含函数原型的函数指针变量
    func3 = test;
    func();
    func2();
    func3();
    return EXIT_SUCCESS;
}
```

输出为：

``` 
test
test
test
```

C语言中所有对象，包括指针本身都是“复制传值”传递，我们可以通过传递“指针的指针”来实现传出参数。

示例：

``` c
#include <stdio.h>
#include <stdlib.h>

void test(int** x)
{
    int* p = malloc(sizeof(int));
    *p = 123;
    *x = p;
}

int main(int argc, char* argv[])
{
    int* p;
    test(&p);
    printf("%d\n", *p);
    free(p);

    return EXIT_SUCCESS;
}
```

C99修饰符：

- extern：默认修饰符，用于函数表示“具有外部链接的标识符”，这类函数可用于任何程序文件。用于变量声明表示该变量在其他单元中定义。
- static：使用该修饰符的函数仅在其所在编译单元（源码文件）中可用。还可以表示函数内的静态变量。
- inline：修饰符inline建议编译器将函数代码内联到调用处，但编译器可自主决定是否完成。通常包含循环或递归函数不能被定义为inline函数。

------

当数组作为函数参数时，总是被隐式转换为指向数组第一元素的指针，也就是说我们再也无法用sizeof获得数组的实际长度了。

示例：

``` c
#include <stdio.h>
#include <stdlib.h>

void test(int x[])
{
    printf("%d\n", sizeof(x));
}

void test2(int* x)
{
    printf("%d\n", sizeof(x));
}

int main(int argc, char* argv[])
{
    int x[] = { 1, 2, 3 };
    printf("%d\n", sizeof(x));

    test(x);
    test2(x);

    return EXIT_SUCCESS;
}
```

输出为：

``` 
12
4
4
```

C99支持长度可变数组作为函数参数。

------

void指针：``void*`` 又被称为 万能指针 ，可以代表任何对象的地址，但没有该对象的类型。也就是说必须转型后才能进行对象操作。 ``void*`` 指针可以与其他任何类型指针进行隐式转换。

示例：

``` c
#include <stdio.h>
#include <stdlib.h>

void test(void* p, size_t len)
{
    unsigned char* cp = p;

    for(int i = 0; i < len; i++)
    {
        printf("%02x ", *(cp + i));
    }

    printf("\n");
}

int main(int argc, char* argv[])
{
    int x = 0x00112233;
    // sizeof的单位为字节，正好可以存储2个十六进制数
    test(&x, sizeof(x));

    return EXIT_SUCCESS;
}
```

输出为：

``` 
33 22 11 00
```

可以用初始化器初始化指针：

- 空指针常量NULL
- 相同类型的指针，或者指向限定符较少的相同类型指针
- void指针

指针运算：

- 对指针进行相等或不等运算来判断是否指向同一对象
- 对指针进行加法运算获取数组第n个元素指针
- 对指针进行减法运算，以获取指针所在元素的数组索引序号
- 对指针进行大小比较运算，相当于判断数组索引序号大小
- 我们可以直接用 ``&x[i]`` 获取指定序号元素的指针

限定符 const 可以声明“类型为指针的常量”和“指向常量的指针”。

示例：

``` c
int x[] = {1, 2, 3};

// 指针常量：指针本身为常量，不可修改，但可修改目标对象
int* const p1 = x;
*(p1 + 1) = 22;
printf("%d\n", x[1]);

// 常量指针：目标对象为常量，不可修改，但可修改指针
int const *p2 = x;
p2++;
printf("%d\n", *p2);
```

区别在于const是修饰 ``p`` 还是 ``*p`` 。

------

结构类型无法把自己作为成员类型，但可以包含“指向自己类型”的指针成员：

``` c
struct list_node
{
    struct list_node* prev;
    struct list_node* next;
    void* value;
};
```

定义不完整结构类型，只能使用小标签：

``` c
typedef struct node_t
{
    struct node_t* prev;
    struct node_t* next;
    void* value;
} list_node;
```

小标签可以和typedef定义的类型名相同：

``` c
typedef struct node_t
{
    struct node_t* prev;
    struct node_t* next;
    void* value;
} node_t;
```

在结构体内部使用匿名结构体成员，也是一种很常见的做法：

``` c
#include <stdio.h>
#include <stdlib.h>

typedef struct
{
    struct
    {
        int length;
        char chars[100];
    } s;

    int x;
} data_t;

int main(int argc, char* argv[])
{
    data_t d = { .s.length = 100, .s.chars = "abcd", .x = 1234 };
    printf("%d\n%s\n%d\n", d.s.length, d.s.chars, d.x);

    return EXIT_SUCCESS;
}
```

利用 stddef.h 中的 offsetof 宏可以获取结构成员的偏移量。



“不定长结构”就是在结构体尾部声明一个未指定长度的数组。用sizeof运算符时，该数组未计入结果。

示例：

``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct string
{
    int length;
    char chars[];
} string;

int main(int argc, char* argv[])
{
    int len = sizeof(string) + 10;  // 计算存储一个 10 字节长度的字符串（包括\0）所需的长度
    char buf[len];      // 从栈上分配所需的内存空间

    string* s = (string*)buf;   // 转换成 struct string 指针
    s->length = 9;
    strcpy(s->chars, "123456789");

    printf("%d\n%s\n", s->length, s->chars);

    return EXIT_SUCCESS;
}
```

对这类结构体进行拷贝的时候，尾部结构成员不会被复制，而且不能直接对弹性结构成员进行初始化。

示例：

``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct string
{
    int length;
    char chars[];
} string;

int main(int argc, char* argv[])
{
    int len = sizeof(string) + 10;
    char buf[len];

    string *s = (string*)buf;
    s->length = 9;
    strcpy(s->chars, "123456789");

    string s2 = *s;         // 复制 struct string s
    printf("%d\n%s\n", s2.length, s2.chars);        // s2.length正常，s2.chars 就悲剧了

    return EXIT_SUCCESS;
}
```

------

联合和结构的区别在于：联合每次只能存储一个成员，联合的长度由最宽成员类型决定。

------

位字段：可以把结构或联合的多个成员“压缩存储”在一个字段中，以节约内存。

``` c
struct
{
    unsigned int year: 22;
    unsigned int month: 4;
    unsigned int day: 5;
} d = {2010, 4, 30};

printf("size: %d\n", sizeof(d));
printf("year = %u, month = %u, day = %u\n", d.year, d.month, d.day);
```

输出为：

``` 
size: 4
year = 2010, month = 4, day = 30
```

用来做标志位也挺好，比用位移运算符更直观，更节省内存。

``` c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

int main(int argc, char* argv[])
{
    struct
    {
        bool a: 1;
        bool b: 1;
        bool c: 1;
    } flags = { .b = true };

    printf("%s\n", flags.b ? "b.T" : "b.F");
    printf("%s\n", flags.c ? "c.T" : "c.F");

    return EXIT_SUCCESS;
}
```

不能对位字段成员使用 offsetof。

------

具有静态生存周期的对象，会被初始化为默认值0（指针为NULL）。

------

预处理指令以 ``#`` 开始（其前面可以有space或tab），通常独立一行，但可以用“ ``\`` ”换行。

编译器会展开替换掉宏。如：

``` c
#define SIZE 10

int main(int argc, char* argv[])
{
    int x[SIZE] = {};
    return EXIT_SUCCESS;
}
```

展开：

``` c
int main(int argc, char* argv[])
{
    int x[10] = {};
    return 0;
}
```

利用宏可以定义伪函数，通常用 ``({...})`` 来组织多行语句，最后一个表达式作为返回值（无return，且有个“;”结束）。如：

``` c
#define test(x, y) ({   \
    int _z = x + y; \
    _z; })

int main(int argc, char* argv[])
{
    printf("%d\n", test(1, 2));
    return EXIT_SUCCESS;
}
```

展开：

``` c
int main(int argc, char* argv[])
{
    printf("%d\n", ({ int _z = 1 + 2; _z; }));
    return 0;
}
```

可以使用“ ``#if ... #elif ... #else ... #endif`` ”、 ``#define`` 、 ``#undef`` 进行条件编译。