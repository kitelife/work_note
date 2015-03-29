《C学习笔记》读书笔记
=========================

字符常量默认是一个int整数，但编译器可以自行决定将其解释为char或int。

如：

::

    char c = 'a';
    printf("%c, size(char)=%d, size('a')=%d;\n", c, sizeof(c), sizeof('a'));

输出为：

::

    a, size(char)=1, size('a')=4;

------

浮点数默认类型是double，可以添加后缀F来表示float，L表示long double

------

字面值（literal）是源代码中用来描述固定值的记号（token），可能是整数、浮点数、字符、字符串。

字符常量默认是int类型，除非用前置L表示 wchar_t 宽字符类型。

::

    char c = 0x61;
    char c2 = 'a';
    char c3 = '\x61';

    printf("%c, %c, %c\n", c, c2, c3);

输出为：

::

    a, a, a

在Linux系统中，默认字符集是UTF-8，可以用 wctomb 等函数进行转换。

wchar_t 默认是4字节长度，足以容纳所有 UCS-4 Unicode 字符。

::

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

输出为：

::

    3
    0xE4 0xB8 0xAD

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

::

    (类型名称){初始化列表}

演示：

::

    int* i = &(int){ 123 };             // 整型变量，指针
    int* x = (int[]){ 1, 2, 3, 4};      // 数组，指针
    struct data_t* data = &(struct data_t){ .x = 123 };     // 结构，指针
    func(123, &(struct data_t){ .x = 123 });                // 函数参数，结构指针参数

如果是静态或全局变量，那么初始化列表必须是编译期变量。


sizeof：返回操作数占用内存空间大小，单位字节（byte）。sizeof返回值是size_t类型，操作数可以是类型或变量。


逗号运算符是一个二元运算符，确保操作数从左到右被顺序处理，并返回右操作数的值和类型。

如：

::

    int i = 1;
    long long x = (i++, (long long)i);
    printf("%lld\n", x);

输出为：

::

    2


无条件跳转：break, continue, goto, return。

goto仅在函数内跳转，常用于跳出嵌套循环。如果在函数外跳转，可使用longjmp。

setjmp将当前位置的相关信息（堆栈帧、寄存器等）保存到jmp_buf结构中，并返回0。当后续代码执行longjmp跳转时，需要提高一个状态码。代码执行时将返回setjmp处，并返回longjmp所提供的状态码。

示例：

::

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


注意区分定义“函数类型”和“函数指针 类型”的区别。函数名是一个指向当前函数的指针。

如：

::

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

输出为：

::

    test
    test
    test


C语言中所有对象，包括指针本身都是“复制传值”传递，我们可以通过传递“指针的指针”来实现传出参数。

示例：

::

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


C99修饰符：

- extern：默认修饰符，用于函数表示“具有外部链接的标识符”，这类函数可用于任何程序文件。用于变量声明表示该变量在其他单元中定义。
- static：使用该修饰符的函数仅在其所在编译单元（源码文件）中可用。还可以表示函数内的静态变量。
- inline：修饰符inline建议编译器将函数代码内联到调用处，但编译器可自主决定是否完成。通常包含循环或递归函数不能被定义为inline函数。

------

当数组作为函数参数时，总是被隐式转换为指向数组第一元素的指针，也就是说我们再也无法用sizeof获得数组的实际长度了。

示例：

::

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

输出为：

::

    12
    4
    4


C99支持长度可变数组作为函数参数。

------

