图解Git
===========

`中文翻译 <http://marklodato.github.io/visual-git-guide/index-zh-cn.html>`_

基本用法
-----------

.. image:: /_static/pics/basic-usage.svg

- git add files 把当前文件放入暂存区域。
- git commit 给暂存区域生成快照并提交。
- git reset -- files 用来撤销最后一次git add files，你也可以用git reset 撤销所有暂存区域文件。
- git checkout -- files 把文件从暂存区域复制到工作目录，用来丢弃本地修改。

约定
-------

后文中以下面的形式使用图片。

.. image:: /_static/pics/conventions.svg

绿色的5位字符表示提交的ID，分别指向父节点。分支用橘色显示，分别指向特定的提交。当前分支由附在其上的HEAD标识。
这张图片里显示最后5次提交，ed489是最新提交。 master分支指向此次提交，另一个maint分支指向祖父提交节点。


命令详解
-----------

Diff
^^^^^^^^

.. image:: /_static/pics/diff.svg

Commit
^^^^^^^^^

如果想更改一次提交，使用 git commit --amend。git会使用与当前提交相同的父节点进行一次新提交，旧的提交会被取消。

.. image:: /_static/pics/commit-amend.svg

Checkout
^^^^^^^^^^^^

