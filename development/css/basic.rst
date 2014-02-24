基础知识
============

- 设置表格列的最大宽度：::

    max-width:  max-width: 300px;

- 表格单元格的内容长度超过单元格的宽度时，如何让其自动换行：::

    word-wrap: break-word;

- `text-aligin: center;` 和 `margin: 0 auto;` 的异同：
  
text-align:center设置文本或img标签等一些内联对象（或与之类似的元素）的居中。margin:0 auto是设置块元素（或与之类似的元素）的居中。但这两个属性IE与FF的理解也有所不同。

.. seealso:: `正确的使用margin:0 auto与body{text-align:center;}实现元素居中 <http://www.52css.com/article.asp?id=505>`_

------

20 Very Useful CSS Stylesheet Tips & Tricks
-----------------------------------------------

1. Round Corners without images

Here is a simple CSS technique of rounding off the corners of the DIV using some css attributes. This technique will work in Firefox, Safari, Chrome and any other CSS3-compatible browser. This technique will not work in Internet Explorer.::

    div {
        -moz-border-radius: 10px;
        -webkit-border-radius: 10px;
        border-radius: 10px;
    }

To round a specific corner (top-left or bottom-right) use below stylesheet.::

    div {
        -moz-border-radius-topleft: 10px;
        -webkit-border-top-left-radius: 10px;
    }

2. Create an IE Specific Stylesheet

Create a separate stylesheet altogether and include it in the webpage whenever the client is using Internet Explorer.

**IE Only** ::

    <!--[if IE]>
        <link rel="stylesheet" type="text/css" href="ie-only.css" />
    <![endif]-->

**IE 7 Only** ::

    <!--[if IE 7]>
        <link href="IE-7-SPECIFIC.css" rel="stylesheet" type="text/css">
    <![endif]-->

.. seealso:: `How To: Create An IE Specific Stylesheet <http://viralpatel.net/blogs/how-to-create-ie-specific-css-stylesheet/>`_

3. Background Image of Textbox::

    input#sometextbox {
        background-image:url('back-image.gif');
        background-repeat:no-repeat;
        padding-left:20px;
    }

7. Rotate Text using CSS

The rotation property of Internet Explorer’s BasicImage filter can accept one of four values: 0, 1, 2, or 3 which will rotate the element 0, 90, 180 or 270 degress respectively.::

    .rotate-style {
        /* Safari */
        -webkit-transform: rotate(-90deg);

        /* Firefox */
        -moz-transform: rotate(-90deg);

        /* Internet Explorer */
        filter: progid:DXImageTransform.Microsoft.BasicImage(rotation=3);
    }

9. Change Text Selection Color

By default, browsers uses blue color as the text selection. You can change this color to match your website theme.::

    /* Mozilla based browsers */
    ::-moz-selection {
       background-color: #FFA;
       color: #000;
    }

    /* Works in Safari */
    ::selection {
       background-color: #FFA;
       color: #000;
    }

11. Centering a Website

Most of website uses this technique to center the content.::

    <body>
        <div id="page-wrap">
            <!-- all websites HTML here -->
        </div>
    </body>


    #page-wrap {
        width: 800px;
        margin: 0 auto;
    }

12. CSS Drop Caps

If your browser supports the pseudo-class “first-letter”, the first letter will be a drop-cap.::

    p:first-letter {
       font-size : 300%;
       font-weight : bold;
       float : left;
       width : 1em;
    }

13. Attribute-Specific Icons

CSS Attribute selectors are very powerful giving you many options to control styles of different elements e.g. you can add an icon based on the href attribute of the a tag to let the user know whether link points to an image, pdf, doc file etc.::

    a[href$='.doc'] {
        padding:0 20px 0 0;
        background:transparent url(/graphics/icons/doc.gif) no-repeat center rightright;
    }

14. Capitalize Text

This trick is especially useful for displaying title of an article on a web page with all its words starting with capital letter.::

    text-transform: capitalize;

    text-transform: lowercase

    text-transform: uppercase

- none: No capitalization. The text renders as it is. This is default
- capitalize: Transforms the first character of each word to uppercase
- uppercase: Transforms all characters to uppercase
- lowercase: Transforms all characters to lowercase
- inherit: Specifies that the value of the text-transform property should be inherited from the parent element

15. CSS Text Shadow

Regular text shadow:::

    p { text-shadow: 1px 1px 1px #000; }

Multiple shadows:::

    p { text-shadow: 1px 1px 1px #000, 3px 3px 5px blue; }

The first two values specify the length of the shadow offset. The first value specifies the horizontal distance and the second specifies the vertical distance of the shadow. The third value specifies the blur radius and the last value describes the color of the shadow.

18. CSS Pointer Cursors
::

    input[type=submit],label,select,.pointer { cursor:pointer; }


.. seealso:: `20 Very Useful CSS Stylesheet Tips & Tricks <http://viralpatel.net/blogs/20-very-useful-css-stylesheet-tips-tricks/>`_

------

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

资源
-------

1. `Alice---写样式的更好方式 <http://aliceui.org/>`_
2. `20 Very Useful CSS Stylesheet Tips & Tricks <http://viralpatel.net/blogs/20-very-useful-css-stylesheet-tips-tricks/>`_
3. `CSS Architecture <http://engineering.appfolio.com/2012/11/16/css-architecture/>`_
4. `50 Useful CSS Snippets Every Designer Should Have <http://www.hongkiat.com/blog/css-snippets-for-designers/>`_
5. `Pure-A set of small, responsive CSS modules that you can use in every web project <http://purecss.io/>`_ (看起来挺不错，有时间看看)
6. `Github CSS StyleGuide <https://github.com/styleguide/css>`_
7. `学习CSS布局 <http://zh.learnlayout.com/>`_
8. 颜色选择器： `https://kuler.adobe.com/create/color-wheel/ <https://kuler.adobe.com/create/color-wheel/>`_ , `http://colorschemedesigner.com/ <http://colorschemedesigner.com/>`_
9. `CSS动画简介 <http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html>`_
