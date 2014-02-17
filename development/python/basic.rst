基础知识
===============

对于元组、列表、字符串等类型的变量，在其上面加上星号（*）然后传递，其效果是将该变量数据进行unpack（解包）后在传递。

.. seealso:: `知乎：元组的reference前加个星号是什么意思？ <http://www.zhihu.com/question/20801578>`_

------

根据字符串获取某对象以该字符串命名的属性/方法:

::

  def getattr(object, name, default=None): # known special case of getattr
    """
    getattr(object, name[, default]) -> value
    
    Get a named attribute from an object; getattr(x, 'y') is equivalent to x.y.
    When a default argument is given, it is returned when the attribute doesn't
    exist; without it, an exception is raised in that case.
    """
    

根据字符串查询某对象是否有以该字符串命名的属性/方法：

::
  
  def hasattr(p_object, name): # real signature unknown; restored from __doc__
    """
    hasattr(object, name) -> bool
    
    Return whether the object has an attribute with the given name.
    (This is done by calling getattr(object, name) and catching exceptions.)
    """
  

反射
-----------

.. seealso:: `Python自省（反射）指南 <http://www.cnblogs.com/huxi/archive/2011/01/02/1924317.html>`_
.. seealso:: `stackoverflow - Python: Once and for all. What does the Star operator mean in Python? <http://stackoverflow.com/questions/2921847/python-once-and-for-all-what-does-the-star-operator-mean-in-python>`_


魔术方法
-----------

.. seealso:: 1. `A Guide to Python's Magic Methods <http://www.rafekettler.com/magicmethods.html>`_ , 2. `Python魔术方法指南 <http://pycoders-weekly-chinese.readthedocs.org/en/latest/issue6/a-guide-to-pythons-magic-methods.html>`_


装饰器
----------

.. seealso:: 1. `Python装饰器入门 <http://youngsterxyf.github.io/2012/07/30/a-primer-on-python-decorators/>`_ , 2. `装饰器与函数式Python <http://youngsterxyf.github.io/2013/01/04/Decorators-and-Functional-Python/>`_


参考资料
------------

- `Python入门指南(2.7) <http://www.pythontab.com/html/pythonshouce27/index.html>`_ ， `英文版 <http://docs.python.org/2/tutorial/>`_
- `PythonTab <http://www.pythontab.com/>`_
- `Python's Hardest Problem <http://www.jeffknupp.com/blog/2012/03/31/pythons-hardest-problem/>`_
- `The Hitchhiker’s Guide to Packaging <http://guide.python-distribute.org/index.html>`_
- `Unofficial Windows Binaries for Python Extension Packages <http://www.lfd.uci.edu/~gohlke/pythonlibs/>`_
- `Python modules you should know <https://devcharm.com/pages/11-python-modules-you-should-know>`_
- `Python Central <http://www.pythoncentral.io/>`_
- `蟒周刊 <http://weekly.pychina.org/>`_
- `Using Pandas and XlsxWriter to create Excel charts <http://pandas-xlsxwriter-charts.readthedocs.org/>`_
- `Parallelism in one line - A Better Model for Day to Day Threading Tasks <https://medium.com/building-things-on-the-internet/40e9b2b36148>`_ , `译文 <http://blog.jobbole.com/58700/>`_
