Git-简易指南
=================

`中文翻译 <http://rogerdudler.github.io/git-guide/index.zh.html>`_

检出仓库
-----------

执行如下命令可以创建一个本地仓库的克隆版本
::

    git clone /path/to/repository

如果是远端服务器上的仓库，你的命令会是这个样子的
::

    git clone username@host:/path/to/repository


工作流
---------

你的本地仓库由git维护的三棵“树”组成。第一个是你的 **工作目录** ，它持有实际文件；第二个是 **缓存区（index）** ，
临时保存你的改动；最后是 **HEAD** ，指向你最近一次提交后的结果（？或者git pull之后的结果？）

.. image:: /_static/pics/trees.png


分支
--------

分支是用来将特性开发绝缘开来的。在你创建仓库的时候，master是“默认”的。在其他分支上进行开发，完成后再将它们合并到主分支上。

创建一个叫做“feature_x”的分支，并切换过去
::

    git checkout -b feature_x

切换回主分支
::

    git checkout master

删除分支
::

    git branch -d feature_x

除非你将分支推送到远端仓库，不然该分支就是 *不为他人所见* 的
::

    git push origin <branch>


更新与合并
-------------

要更新你的本地仓库至最新改动，执行
::

    git pull

以在你的工作目录中获取（fetch）并合并（merge）远端的改动。

要合并其他分支到你的当前分支（例如master），执行
::

    git merge <branch>

两种情况下，git都会尝试去自动合并改动。不幸的是，自动合并并非每次都能成功，并可能导致 *冲突（conflicts）* 。
这时候就需要你修改这些文件来人肉合并这些 *冲突（conflicts）* 了。改完之后，你需要执行如下命令以将它们标记为合并成功
::

    git add <filename>

在合并之前，也可以使用如下命令查看
::

    git diff <source_branch> <target_branch>


标签
-------

在软件发布时创建标签，是被推荐的。这是个旧有概念，在SVN中也有。可以执行如下命令来创建一个叫做1.0.0的标签
::

    git tag 1.0.0 1b2e1d63ff

1b2e1d63ff是你想要标记的提交ID的前10位字符。使用如下命令获取提交ID
::

    git log

你也可以用该提交ID的少一些的前几位，只要它是唯一的。


替换本地改动
---------------

假如你做错了事，你可以使用如下命令替换掉本地改动
::

    git checkout -- <filename>

此命令会使用Index中的最新内容替换掉你的工作目录中的文件。已添加到缓冲区的改动，以及新文件，都不受影响。

假如你想要丢弃你所有的本地改动与提交，可以到服务器上获取最新的版本并将你本地主分支指向到它
::

    git fetch origin
    git reset --hard origin/master


有用的贴士
-------------

内建的图形化git
::

    gitk

彩色的git输出
::

    git config color.ui true    # 针对单个代码库
    git config --global color.ui true   # 全局有效

显示历史记录时，只显示一行注释信息
::

    git config format.pretty oneline    # 如上，若想全局有效，则加--global选项
