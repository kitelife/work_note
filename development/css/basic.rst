基础知识
============

**层叠顺序**

样式表属性的声明可能会出现在多个样式表中，也可能在同一个样式表中出现多次。这意味着应用规则的顺序极为重要。优先级从低到高层叠顺序如下所示：

#) 浏览器声明
#) 用户普通声明
#) 作者普通声明
#) 作者重要声明
#) 用户重要声明


**CSS框模型**

.. image:: /_static/pics/boxdim.png


**定位**

*相对*

相对定位：先按照普通方式定位，然后根据所需偏移量进行移动，如图：

.. image:: /_static/pics/relative_position.png

*浮动*

浮动框会移动到行的左边或右边。有趣的特征在于，其他框会浮动在它的周围。下面这段HTML代码：

::

    <p>
        <img style="float:right" src="images/image.gif" width="100" height="100">
        Lorem ipsum dolor sit amet, consectetuer...
    </p>

显示效果如下：

.. image:: /_static/pics/relative_result.png

*绝对定位和固定定位*

这种布局是准确定义的，与普通流无关。元素不参与普通流。尺寸是想对于容器而言的。在固定定位中，容器就是可视区域。

.. image:: /_static/pics/absolute_position.png

请注意，即使在文档滚动时，固定框也不会移动。


**分层展示**

这是由z-index CSS属性指定的。它代表了框的第三个维度，也就是沿“z轴”方向的位置。

这些框分散到多个堆栈（称为堆栈上下文）中。在每一个堆栈中，会首先绘制后面的元素，然后在顶部绘制前面的元素，以便更靠近用户。如果出现重叠，新绘制的元素就会覆盖之前的元素。

堆栈是按照z-index属性进行排序的，示例如下：

::

    <style type="text/css">
      div {
        position: absolute;
        left: 2in;
        top: 2in;
      }
    </style>

    <p>
        <div
            style="z-index: 3;background-color:red; width: 1in; height: 1in; ">
        </div>
        <div style="z-index: 1;background-color:green;width: 2in; height: 2in;">
        </div>
    </p>

结果如下：

.. image:: /_static/pics/static_position.png

虽然红色div在标记中的位置比绿色div靠前（按理应该在常规流程中优先绘制），但是z-index属性的优先级更高，因此它移动到了根框所保持的堆栈中更靠前的位置。
