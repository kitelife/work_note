jQuery
============

源码阅读(v1.6.2)
------------

class2type：::
    class2type = {};
    
    jQuery.each("Boolean Number String Function Array Date RegExp Object".split(" "), function(i, name){
        class2type[ "[object " + name + "]" ] = name.toLowerCase();
    });

    // determine the internal JavaScript [[Class]] of an object
    type: function(obj) {
        return obj == null ?
            String(obj) :
            class2type[ toString.call(obj) ] || "object";
    }


isFunction：::
    isFunction: function (obj) {
        return jQuery.type(obj) === "function";
    }

isArray：::
    // 尽可能使用原生的方法
    isArray: Array.isArray || function (obj) {
        return jQuery.type(obj) === "array";
    }

isPlainObject：::
    // 检查一个对象是否是一个普通的对象（以"{}"或"new Object"创建的对象）
    isPlainObject: function (obj) {
        // Must be an Object
        // Because of IE, we also have to check the presence of the constructor property.
        // Make sure that DOM nodes and window objects don't pass through, as well
        if (!obj || jQuery.type(obj) !== "object" || obj.nodeType || jQuery.isWindow(obj)) {
            return false;
        }

        // Not own constructor property must be Object
        if (obj.constructor && !hasOwn.call(obj, "constructor") && !hasOwn.call(obj.constructor.prototype, "isPrototypeOf")) {
            return false;
        }

        // Own properties are enumerated firstly, so to speed up,
        // if last one is own, then all properties are own.
        
        var key;
        for (key in obj) {
        }

        return key === undefined || hasOwn.call(obj, key);
    }


