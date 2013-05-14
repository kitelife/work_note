Web开发问题记录
==================

IE 8没有内置好用的前端调试工具，可以通过在待调试的页面中引入firebug来调试。


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
替代；2）为Array.prototype添加forEach方法。

对于JavaScript数组的map方法，IE 8也不支持，需自己实现。
