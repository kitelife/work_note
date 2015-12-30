# 基础知识

对于元组、列表、字符串等类型的变量，在其上面加上星号（*）然后传递，其效果是将该变量数据进行unpack（解包）后在传递。

**扩展阅读**：[知乎：元组的reference前加个星号是什么意思?](http://www.zhihu.com/question/20801578)

根据字符串获取某对象以该字符串命名的属性/方法:

``` python
def getattr(object, name, default=None): # known special case of getattr
    """
    getattr(object, name[, default]) -> value

    Get a named attribute from an object; getattr(x, 'y') is equivalent to x.y.
    When a default argument is given, it is returned when the attribute doesn't
    exist; without it, an exception is raised in that case.
    """
```

根据字符串查询某对象是否有以该字符串命名的属性/方法：

``` python
def hasattr(p_object, name): # real signature unknown; restored from __doc__
    """
    hasattr(object, name) -> bool

    Return whether the object has an attribute with the given name.
    (This is done by calling getattr(object, name) and catching exceptions.)
    """
```

## 反射

[Python自省（反射）指南](http://www.cnblogs.com/huxi/archive/2011/01/02/1924317.html)

[stackoverflow - Python: Once and for all. What does the Star operator mean in Python?](http://stackoverflow.com/questions/2921847/python-once-and-for-all-what-does-the-star-operator-mean-in-python)

## 魔术方法

[A Guide to Python's Magic Methods](http://www.rafekettler.com/magicmethods.html)

[Python魔术方法指南](http://pycoders-weekly-chinese.readthedocs.org/en/latest/issue6/a-guide-to-pythons-magic-methods.html)

## 装饰器

[Python装饰器入门](http://youngsterxyf.github.io/2012/07/30/a-primer-on-python-decorators/)

[装饰器与函数式Python](http://youngsterxyf.github.io/2013/01/04/Decorators-and-Functional-Python/)

## 值得关注

- [pulsar](https://github.com/quantmind/pulsar)

> Event driven framework for Python

## 推荐阅读

- [The Hitchhiker’s Guide to Packaging](http://guide.python-distribute.org/index.html)
- [Unofficial Windows Binaries for Python Extension Packages](http://www.lfd.uci.edu/~gohlke/pythonlibs/)
- [蟒周刊](http://weekly.pychina.org/)

