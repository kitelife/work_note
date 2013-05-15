JavaScript基础
=================

Style Guide
--------------

**Equality**

Strict equality checks (===) should be used in favor of ==. The only exception
is when checking for undefined and null by way of null.::

    // Check for both undefined and null values, for some important reason.
    undefOrNull == null;

**Type Checks**

- String: typeof object === "string"
- Number: typeof object === "number"
- Boolean: typeof object === "boolean"
- Object: typeof object === "object"
- Plain Object: jQuery.isPlainObject( object )
- Function: jQuery.isFunction( object )
- Array: jQuery.isArray( object )
- Element: object.nodeType
- null: object === null
- null or undefined: object == null
- undefined:
-- Global Variables: typeof variable === "undefined"
-- Local Variables: variable === undefined
-- Properties: object.prop === undefined

------

**内部函数中的this**

::

    var name = "clever coder";
    var person = {
        name: "foocoder",
        hello: function (sth) {
            var sayhello = function (sth) {
                console.log(this.name + " says " + sth);
            };
            sayhello(sth);
        }
    }
    person.hello("hello world");    // clever coder says hello world

JavaScript的内部函数中，this没有按预想的绑定到外层函数对象上，而是绑定到了全局对象。
这里普遍被认为是JavaScript语言的设计错误，因为没有人想让内部函数中的this指向全局函数。
一般的处理方式是将this作为变量保存下来，一般约定为that或者self::

    var name = "clever coder";
    var person = {
        name: "foocoder",
        hello: function (sth) {
            var that = this;
            var sayhello = function (sth) {
                console.log(that.name + " says " + sth);
            };
            sayhello(sth);
        }
    }
    person.hello("hello world");    // foocoder says hello world


**名称解析顺序**

JavaScript中的所有作用域，包括全局作用域，都有一个特别的名称 ``this`` 指向当前对象。

函数作用域内也有默认的变量 ``arguments`` ，其中包含了传递到函数中的参数。

比如，当访问函数内的 ``foo`` 变量时，JavaScript会按照下面顺序查找：

1. 当前作用域内是否有 ``var foo`` 的定义。

2. 函数形式参数是否有使用 ``foo`` 名称的。

3. 函数自身是否叫做 ``foo`` 。

4. 回溯到上一级作用域，然后从 #1 重新开始。


材料
---------

- Learning from jQuery
- JavaScript DOM编程艺术
- High Performance JavaScript
- `Learning JavaScript Design Patterns <http://addyosmani.com/resources/essentialjsdesignpatterns/book/>`_
- JavaScript Patterns
- 高性能网站建设指南
