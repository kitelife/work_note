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

```flex
/* fb1-4.l */
/* 识别出用于计算器的记号并把它们输出 */
/* 注意yylex()函数的使用 */

%{
    enum yytokentype {
        NUMBER = 258,
        ADD = 259,
        SUB = 260,
        MUL = 261,
        DIV = 262,
        ABS = 263,
        EOL = 264
    };
    int yylval;
%}

%%
"+"     { return ADD; }
"-"     { return SUB; }
"*"     { return MUL; }
"/"     { return DIV; }
"|"     { return ABS; }
[0-9]+  { yylval = atoi(yytext); return NUMBER; }
\n      { return EOL; }
[ \t]   { /* 忽略空白字符 */ }
.       { printf("Mystery character %c\n", *yytext); }
%%
void main(int argc, char **argv)
{
    int tok;
    
    while(tok = yylex()) {
        printf("%d", tok);
        if(tok == NUMBER) printf(" = %d\n", yylval);
        else printf("\n");
    }
}
int yywrap()
{
    return 1;
}
```

### 文法与语法分析

每一个bison语法分析器在分析其输入时都会构造一棵语法分析树。在有些应用里，它把整棵树作为一个数据结构创建在内存中以便于后续使用。在其他应用里，语法分析树只是隐式地包含在语法分析器进行的一系列操作中。

#### BNF文法

为了编写一个语法分析器，需要一定的方法来描述语法分析器所使用的把一系列记号转化为语法分析树的规则。在计算机分析程序里最常用的语言就是上下文无关文法（Context-Free Grammar, CFG）。书写上下文无关文法的标准格式就是Backus-Naur范式（BackusNaur Form，BNF）。

BNF示例：

```
<exp> ::= <factor>
    | <exp> + <factor>
<factor> ::= NUMBER
    | <factor> * NUMBER
```

每一行就是一条规则，用来说明如何创建语法分析树的分支。

有效的BNF总是带有递归性的，规则会直接或者间接地指向自身。

#### Bison的规则描述语言

bison的规则基本上就是BNF，但是做了一点点简化以易于输入。

```flex
/* fb1-5.l*/
%{
#include "fb1-5.tab.h"
%}

%%
"+"     { return ADD; }
"-"     { return SUB; }
"*"     { return MUL; }
"/"     { return DIV; }
"|"     { return ABS; }
[0-9]+  { yylval = atoi(yytext); return NUMBER; }
\n      { return EOL; }
[ \t]   { /* 忽略空白字符 */ }
.       { printf("Mystery character %c\n", *yytext); }
%%
int yywrap()
{
    return 1;
}
```

```yacc
/* 简单的计算器 fb1-5.y */
%{
#include <stdio.h>

int yylex (void);
void yyerror (char const *);
%}

/* declare tokens */
%token NUMBER
%token ADD SUB MUL DIV ABS
%token EOL

%%

calclist:
    | calclist exp EOL { printf("= %d\n", $2); }
    ;

exp: factor { $$ = $1; }
    | exp ADD factor { $$ = $1 + $3; }
    | exp SUB factor { $$ = $1 - $3; }
    ;

factor: term { $$ = $1; }
    | factor MUL term { $$ = $1 * $3; }
    | factor DIV term { $$ = $1 / $3; }
    ;

term: NUMBER { $$ = $1; }
    | ABS term { $$ = $2 >= 0? $2 : -$2; }
    ;

%%
void main(int argc, char **argv)
{
    yyparse();
}
void yyerror (char const *s)
{
    fprintf (stderr, "%s\n", s);
}
```

#### 联合编译Flex和Bison程序

```makefile
fb1-5: fb1-5.l fb1-5.y
	bison -d fb1-5.y
	flex fb1-5.l
	cc -o $@ fb1-5.tab.c lex.yy.c
```



