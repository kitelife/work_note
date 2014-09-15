JavaScript基础
=================

21 JavaScript Tips And Tricks For JavaScript Developers
----------------------------------------------------------

1. Converting JavaScript Array to CSV
::

    var fruits = ['apple', 'peaches', 'oranges', 'mangoes'];
    var str = fruits.valueOf();

The ``valueOf()`` method will convert an array in javascript to a comma separated string.

Now what if you want to use pipe (|) as delimeter instead of comma. You can convert a js
array into a delimeted string using join() method::

    var fruits = ['apple', 'peaches', 'oranges', 'mangoes'];
    var str = fruits.join("|");

2. Convert CSV to Array in JavaScript
::

    var str = "apple,peaches,oranges,mangoes";
    var fruitsArray = str.split(",");

3. Remove Array element by Index
::

    function removeByIndex(arr, index) {
        arr.splice(index, 1);
    }

    test = new Array();
    test[0] = 'Apple';
    test[1] = 'Ball';
    test[2] = 'Cat';
    test[3] = 'Dog';

    alert("Array before removing elements: "+test);

    removeByIndex(test, 2);

    alert("Array after removing elements: "+test);

4. Remove Array element by Value
::

    function removeByValue(arr, val) {
        for(var i=0; i<arr.length; i++) {
            if(arr[i] == val) {
                arr.splice(i, 1);
                break;
            }
        }
    }

    var somearray = ["mon", "tue", "wed", "thur"]

    removeByValue(somearray, "tue");

5. Calling JavaScript function from String
::

    var strFun = "someFunction"; //Name of the function to be called
    var strParam = "this is the parameter"; //Parameters to be passed in function

    //Create the function
    var fn = window[strFun];

    //Call the function
    fn(strParam);

6. Generate Random number from 1 to N
::

    //random number from 1 to N
    var random = Math.floor(Math.random() * N + 1);

    //random number from 1 to 10
    var random = Math.floor(Math.random() * 10 + 1);

    //random number from 1 to 100
    var random = Math.floor(Math.random() * 100 + 1);

11. Deleting Multiple Values From Listbox in JavaScript
::

    function selectBoxRemove(sourceID) {
        //get the listbox object from id.
        var src = document.getElementById(sourceID);

        //iterate through each option of the listbox
        for(var count= src.options.length-1; count >= 0; count--) {

            //if the option is selected, delete the option
            if(src.options[count].selected == true) {

                try {
                    src.remove(count, null);

                } catch(error) {
                    src.remove(count);
                }
            }
        }
    }

12. Listbox Select All/Deselect All using JavaScript
::

    function listboxSelectDeselect(listID, isSelect) {
        var listbox = document.getElementById(listID);
        for(var count=0; count < listbox.options.length; count++) {
            listbox.options[count].selected = isSelect;
        }
    }

13. Listbox Move selected items Up / Down
::

    function listbox_move(listID, direction) {
        var listbox = document.getElementById(listID);
        var selIndex = listbox.selectedIndex;

        if(-1 == selIndex) {
            alert("Please select an option to move.");
            return;
        }

        var increment = -1;
        if(direction == 'up')
            increment = -1;
        else
            increment = 1;

        if((selIndex + increment) < 0 ||
            (selIndex + increment) > (listbox.options.length-1)) {
            return;
        }

        var selValue = listbox.options[selIndex].value;
        var selText = listbox.options[selIndex].text;
        listbox.options[selIndex].value = listbox.options[selIndex + increment].value
        listbox.options[selIndex].text = listbox.options[selIndex + increment].text

        listbox.options[selIndex + increment].value = selValue;
        listbox.options[selIndex + increment].text = selText;

        listbox.selectedIndex = selIndex + increment;
    }
    //..
    //..

    listbox_move('countryList', 'up'); //move up the selected option
    listbox_move('countryList', 'down'); //move down the selected option


14. Listbox Move Left/Right Options
::

    function listbox_moveacross(sourceID, destID) {
        var src = document.getElementById(sourceID);
        var dest = document.getElementById(destID);

        for(var count=0; count < src.options.length; count++) {
            if(src.options[count].selected == true) {
                var option = src.options[count];

                var newOption = document.createElement("option");
                newOption.value = option.value;
                newOption.text = option.text;
                newOption.selected = true;
                try {
                         dest.add(newOption, null); //Standard
                         src.remove(count, null);
                 }catch(error) {
                         dest.add(newOption); // IE only
                         src.remove(count);
                 }
                count--;
            }
        }
    }
    //..
    //..

    listbox_moveacross('countryList', 'selectedCountryList');

16. Rounding Numbers to ‘N’ Decimals
::

    var num = 2.443242342;
    alert(num.toFixed(2)); // 2.44

Note that we use toFixed() method here. toFixed(n) provides n length after the decimal point; whereas toPrecision(x) provides x total length::

    num = 500.2349;
    result = num.toPrecision(4); // result will equal 500.2

18. Remove Duplicates from JavaScript Array
::

    function removeDuplicates(arr) {
        var temp = {};
        for (var i = 0; i < arr.length; i++)
            temp[arr[i]] = true;

        var r = [];
        for (var k in temp)
            r.push(k);
        return r;
    }

    //Usage
    var fruits = ['apple', 'orange', 'peach', 'apple', 'strawberry', 'orange'];
    var uniquefruits = removeDuplicates(fruits);
    //print uniquefruits ['apple', 'orange', 'peach', 'strawberry'];

19. Trim a String in JavaScript
::

    if (!String.prototype.trim) {
        String.prototype.trim=function() {
            return this.replace(/^\s+|\s+$/g, '');
        };
    }

    //usage
    var str = "  some string    ";
    str.trim();
    //print str = "some string"


20. Redirect webpage in JavaScript

This javascript code should perform http redirect on a given URL::

    window.location.href = "http://viralpatel.net";

21. Encode a URL in JavaScript
::

    var myOtherUrl = "http://example.com/index.html?url=" + encodeURIComponent(myUrl);

.. seealso:: `21 JavaScript Tips And Tricks For JavaScript Developers <http://viralpatel.net/blogs/javascript-tips-tricks/>`_

Style Guide
--------------

**Equality**

Strict equality checks (===) should be used in favor of ==. The only exception
is when checking for undefined and null by way of null.::

    // Check for both undefined and null values, for some important reason.
    undefOrNull == null;

**Type Checks**

* String: typeof object === "string"
* Number: typeof object === "number"
* Boolean: typeof object === "boolean"
* Object: typeof object === "object"
* Plain Object: jQuery.isPlainObject( object )
* Function: jQuery.isFunction( object )
* Array: jQuery.isArray( object )
* Element: object.nodeType
* null: object === null
* null or undefined: object == null
* undefined:
 * Global Variables: typeof variable === "undefined"
 * Local Variables: variable === undefined
 * Properties: object.prop === undefined

http://contribute.jquery.org/style-guide/js/

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

- `JSHint <http://www.jshint.com/>`_
- `How to Use Closure Linter <https://developers.google.com/closure/utilities/docs/linter_howto?hl=zh-CN&csw=1>`_
- `Closure Linter Support in PHP_CodeSniffer <http://www.squizlabs.com/view?a=6065>`_
- `JavaScript 标准教程 <http://javascript.ruanyifeng.com/>`_ （阮一峰）
- Learning from jQuery
- JavaScript DOM编程艺术
- High Performance JavaScript
- `Learning JavaScript Design Patterns <http://addyosmani.com/resources/essentialjsdesignpatterns/book/>`_
- JavaScript Patterns
- 高性能网站建设指南
- `21 JavaScript Tips And Tricks For JavaScript Developers <http://viralpatel.net/blogs/javascript-tips-tricks/>`_
- `JavaScript: The Important Parts <http://benlakey.com/2013/05/26/javascript-the-important-parts/>`_
- `The race for speed part 1: The JavaScript engine family tree <http://creativejs.com/2013/06/the-race-for-speed-part-1-the-javascript-engine-family-tree/>`_ , `The race for speed part 2: How JavaScript compilers work <http://creativejs.com/2013/06/the-race-for-speed-part-2-how-javascript-compilers-work/>`_ , `The race for speed part 3: JavaScript compiler strategies <http://creativejs.com/2013/06/the-race-for-speed-part-3-javascript-compiler-strategies/>`_ , `The race for speed part 4: The future for JavaScript <http://creativejs.com/2013/06/the-race-for-speed-part-4-the-future-for-javascript/>`_
- `15+ Memorable jQuery Timeline Plugins <http://www.tripwiremagazine.com/2013/06/jquery-timeline-plugins.html>`_
- `Frameworkless JavaScript <https://moot.it/blog/technology/frameworkless-javascript.html>`_
- `JavaScript - The Right Way <http://jstherightway.org/>`_
- `Superhero.js <http://superherojs.com/>`_ 赞！
- `angularjs-styleguide <https://github.com/johnpapa/angularjs-styleguide>`_
