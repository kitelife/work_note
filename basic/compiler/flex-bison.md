# 读书笔记：flex与bison

## Flex和Bison简介

词法分析：lexical analysis，或称scanning

语法分析：syntax analysis，或称parsing

------

词法分析把输入分割成一个个有意义的词块，称为记号（token）；语法分析则确定这些记号是如何彼此关联的。

flex程序主要由一系列带有指令的正则表达式组成，这些指令确定了正则表达式匹配后相应的动作（action）。

```flex
/* 正如Unix的wc程序 */
/* 文件名为fb1-1.l */
%{
int chars = 0;
int words = 0;
int lines = 0;
%}
%%
[a-zA-Z]+   { words++; chars += strlen(yytext); }
\n          { chars++; lines++; }
.           { chars++; }
%%

void main(int argc, char **argv)
{
    yylex();
    printf("%8d%8d%8d\n", lines, words, chars);
}
int yywrap()
{
    return 1;
}
```

flex程序包含三个部分，各部分之间通过仅有%%的行来分割。第一个部分包含声明和选项设置，第二个部分是一系列的模式和动作，第三部分则是会被拷贝到生成的词法分析器里面的C代码，它们通常是一些与动作代码相关的例程。

在声明部分，`%{` 和 `%}` 之间的代码会被原样照抄到生成的C文件的开头部分。

在第二部分，每个模式处在一行的开头处，接着是模式匹配时所需要执行的C代码。这儿的C代码是用`{}`括住的一行或者多行语句。

在任意一个flex的动作中，变量yytext总是被设为指向本次匹配的输入文本。

末尾的C代码是主程序，负责调用flex提供的词法分析例程yylex()，并输出结果。

运行：

```
flex fb1-1.l
cc lex.yy.c
./a.out
```

------

### 让Flex和Bison协同工作

```flex
/* 一个简单的flex词法分析器fb1-3.l */
/* 识别出用于计算器的记号并把它们输出 */

%%
"+"     { printf("PLUS\n"); }
"-"     { printf("MINUS\n"); }
"*"     { printf("TIMES\n"); }
"/"     { printf("DIVIDE\n"); }
"|"     { printf("ABS\n"); }
[0-9]+  { printf("NUMBER %s\n", yytext); }
\n      { printf("NEWLINE\n"); }
[ \t]   { }
.       { printf("Mystery character %s\n", yytext); }
%%

void main(int argc, char **argv)
{
    yylex();
}
int yywrap()
{
    return 1;
}
```



