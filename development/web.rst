Web开发问题记录
==================

IE 8没有内置好用的前端调试工具，可以通过在待调试的页面中引入Firebug Lite来调试。::

    <script type="text/javascript" src="https://getfirebug.com/firebug-lite.js"></script>

http://getfirebug.com/firebuglite

浏览器兼容问题
----------------

1.
对于页面隐藏元素，应该使用::

    <input type="hidden" name="..." value="..." />

来实现，这种方式所有浏览器应该都是兼容的。

对于非隐藏输入域的隐藏元素，IE并不支持。

------

2.
IE 8以及更早的版本不支持JavaScript字符串的trim()方法。可如下解决::

    var browser = navigator.appName;
    if(browser === 'Microsoft Internet Explorer'){
        String.prototype.trim = function() {
            return this.replace(/(^\s*)|(\s*$)/g, "");
        }
    }

另外，还有ltrim()和rtrim()方法。


3.
IE 8以及更早也不支持JavaScript数组的forEach方法，有两种解决方案：1）以for循环
替代；2）为Array.prototype添加forEach方法。::
    
    if ( !Array.prototype.forEach ) {
        Array.prototype.forEach = function(fn, scope) {
            for(var i = 0, len = this.length; i < len; ++i) {
                fn.call(scope, this[i], i, this);
            }
        }
    }

https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Array/forEach

4.
对于JavaScript数组的map方法，IE 8也不支持，需自己实现。::

    if (!Array.prototype.map) {
        Array.prototype.map = function(callback, thisArg) {
            var T, A, k;
        
            if (this == null) {
                throw new TypeError(" this is null or not defined");
            } 
            // 1. 将O赋值为调用map方法的数组.
            var O = Object(this);
            // 2.将len赋值为数组O的长度.
            var len = O.length >>> 0;
            // 4.如果callback不是函数,则抛出TypeError异常.
            if ({}.toString.call(callback) != "[object Function]") {
                throw new TypeError(callback + " is not a function");
            }
            // 5. 如果参数thisArg有值,则将T赋值为thisArg;否则T为undefined.
            if (thisArg) {
                T = thisArg;
            }
            // 6. 创建新数组A,长度为原数组O长度len
            A = new Array(len);
            // 7. 将k赋值为0
            k = 0;
            // 8. 当 k < len 时,执行循环.
            while(k < len) {
                var kValue, mappedValue;
                //遍历O,k为原数组索引
                if (k in O) { 
                    //kValue为索引k对应的值.
                    kValue = O[ k ]; 
                    // 执行callback,this指向T,参数有三个.分别是kValue:值,k:索引,O:原数组.
                    mappedValue = callback.call(T, kValue, k, O); 
                    // 返回值添加到新书组A中.
                    A[ k ] = mappedValue;
                }
                // k自增1
                k++;
            }
            // 9. 返回新数组A
            return A;
        };      
    }

https://developer.mozilla.org/zh-CN/docs/JavaScript/Reference/Global_Objects/Array/map

5.
IE 8(在iframe中无法正常使用json)以及更早版本对于JSON没有原生支持，可使用Douglas Crockford写的json2.js，但要
考虑如何根据条件加载该文件。若仅需要解析JSON字符串返回JavaScript对象，也可以使用
jQuery的jQuery.parseJSON方法，但jQuery没有stringify方法。


最佳实践
------------

1.
外部CSS文件在<head>中引入，外部JS文件在<body>的最后位置引入。
