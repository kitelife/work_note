图解Git
===========

`英文原文 <http://marklodato.github.io/visual-git-guide/index-en.html>`_

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

checkout命令用于从历史提交（或暂存区域）中拷贝文件到工作目录，也可用于切换分支。

当给定某个文件名时，git会从指定的提交中拷贝文件到暂存区域和工作目录。比如，git checkout HEAD~ foo.c会将提交节点HEAD~
（即当前提交节点的父节点）中的foo.c复制到工作目录并且加到暂存区域中（？）。（如果命令中没有指定提交节点，则会从暂存区域中拷贝内容。）

.. image:: /_static/pics/checkout-files.svg

当不指定文件名，而是给出一个（本地）分支时，那么HEAD标识会移动到那个分支（也就是说，我们“切换”到那个分支了），
然后暂存区域和工作目录中的内容会和HEAD对应的提交节点一致。新提交节点中的所有文件都会被复制（到暂存区域和工作目录中）；
只存在于老的提交节点（ed489）中的文件会被删除；不属于上述两者的文件会被忽略，不受影响。

如果既没有指定文件名，也没有指定分支名，而是一个标签、远程分支、SHA-1值或者是像master~3类似的东西，就得到一个匿名分支（被分离的HEAD标识），
称作detached HEAD。这样可以很方便地在历史版本之间互相切换。比如说你想要编译1.6.6.1版本的git，你可以运行git checkout v1.6.6.1（这是一个标签，而非分支名），编译，安装，然后切换回另一个分支，比如说git checkout master。


Reset
^^^^^^^

reset命令把当前分支指向另一个位置，并且有选择地变动工作目录和索引。也用来从历史仓库中复制文件到索引，而不动工作目录。

如果不给选项，那么当前分支指向到那个提交。如果用--hard选项，那么工作目录页更新，如果用--soft选项，那么都不变。

.. image:: /_static/pics/reset-commit.svg

If a commit is not given, it defaults to HEAD. In this case, the branch is not moved, but the stage (and optionally the working directory, if --hard is given) are reset to the contents of the last commit.

.. image:: /_static/pics/reset.svg

如果给了文件名(或者 -p选项), 那么工作效果和带文件名的checkout差不多，除了索引被更新。


Rebase
^^^^^^^^^

衍合是合并命令的另一种选择。合并把两个分支合并进行一次提交，提交历史并不是线性的。衍合是将当前分支的历史在另一个分支上重演，提交历史是线性的。

.. image:: /_static/pics/rebase.svg

The above command takes all the commits that exist in topic but not in master (namely 169a6 and 2c33a), replays them onto master, and then moves the branch head to the new tip.
Note that the old commits will be garbage collected if they are no longer referenced.

To limit how far back to go, use the --onto option. The following command replays onto master the most recent commits on the current branch since 169a6 (exclusive), namely 2c33a.

.. image:: /_static/pics/rebase-onto.svg