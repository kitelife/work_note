Pro Git
===========

`英文版 <http://git-scm.com/book>`_ ， `中文版 <http://git-scm.com/book/zh>`_

1. 起步
--------

分布式版本控制系统的客户端并不只是提取最新版本的文件快照，而是把代码仓库完整地镜像下来。
这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。
因此每一次提取操作，实际上都是一次对代码仓库的完整备份。

Git基础
^^^^^^^^^^

**直接记录快照，而非差异比较**

Git和其他版本控制系统的主要差别在于，Git只关心文件数据的整体是否发生变化，而大多数其他系统
则只关心文件内容的具体差异。这类系统（CVS，Subversion，Perforce，Bazaar等）每次记录有哪些文件
作个更新，以及都更新了哪些行的什么内容。如下图所示：

.. image:: /_static/pics/save_diff.png

Git并不保存这些前后变化的差异数据。实际上，Git更像是把变化的文件作快照后，记录在一个微型的文件系统中。
每次提交更新时，它会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这一次快照的索引。
为提高性能，若文件没有变化，Git不会再次保存，而只对上次保存的快照作一链接。Git的工作方式如下图所示：

.. image:: /_static/pics/git_snapshot_way.png

**近乎所有操作都是本地执行**

**时刻保持数据完整性**

在保存到Git之前，所有数据都要进行内容的校验和（checksum）计算，并将结果作为数据的唯一标识和索引。换句话说，
不可能在你修改了文件或目录之后，Git一无所知。所以如果文件在传输时变得不完整，或者磁盘损坏导致文件数据缺失，
Git都能立即察觉。

Git使用SHA-1算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个SHA-1哈希值，作为指纹字符串。该字串
由40个十六进制字符组成。Git的工作完全依赖于这类指纹字符串，所以你会经常看到这样的哈希值。实际上，所有保存在
Git数据库中的东西都是用此哈希值来作索引的，而不是靠文件名。

**多数操作仅添加数据**

**文件的三种状态**

对于任何一个文件，在 Git 内都只有三种状态：已提交（committed），已修改（modified）和已暂存（staged）。
已提交表示该文件已经被安全地保存在本地数据库中了；已修改表示修改了某个文件，但还没有提交保存；
已暂存表示把已修改的文件放在下次提交时要保存的清单中。

每个项目都有一个 Git 目录（译注：如果 git clone 出来的话，就是其中 .git 的目录；如果 git clone --bare 的话，新建的目录本身就是 Git 目录。），它是 Git 用来保存元数据和对象数据库的地方。该目录非常重要，**每次克隆镜像仓库的时候，实际拷贝的就是这个目录里面的数据**。

从项目中取出某个版本的所有文件和目录，用以开始后续工作的叫做工作目录。这些文件实际上都是从 Git 目录中的压缩对象数据库中提取出来的，接下来就可以在工作目录中对这些文件进行编辑。

所谓的暂存区域只不过是个简单的文件，一般都放在 Git 目录中。有时候人们会把这个文件叫做索引文件，不过标准说法还是叫暂存区域。

安装 Git
^^^^^^^^^^

**从源代码安装**

Git 的工作需要调用 curl，zlib，openssl，expat，libiconv 等库的代码，所以需要先安装这些依赖工具。

初次运行Git前的配置
^^^^^^^^^^^^^^^^^^^^

一般在新的系统上，我们都需要先配置下自己的 Git 工作环境。配置工作只需一次，以后升级时还会沿用现在的配置。当然，如果需要，你随时可以用相同的命令修改已有的配置。

Git 提供了一个叫做 git config 的工具（译注：实际是 ``git-config`` 命令，只不过可以通过 git 加一个名字来呼叫此命令。），专门用来配置或读取相应的工作环境变量。而正是由这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

- ``/etc/gitconfig`` 文件：系统中对所有用户都普遍适用的配置。若使用 ``git config`` 时用 ``--system`` 选项，读写的就是这个文件。
- ``~/.gitconfig`` 文件：用户目录下的配置文件只适用于该用户。若使用 ``git config`` 时用 ``--global`` 选项，读写的就是这个文件。
- 当前项目的 git 目录中的配置文件（也就是工作目录中的 ``.git/config`` 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 ``.git/config`` 里的配置会覆盖 ``/etc/gitconfig`` 中的同名变量。

在 Windows 系统上，Git 会找寻用户主目录下的 .gitconfig 文件。主目录即 $HOME 变量指定的目录，一般都是 C:\Documents and Settings\$USER。此外，Git 还会尝试找寻 /etc/gitconfig 文件，只不过看当初 Git 装在什么目录，就以此作为根目录来定位。

**用户信息**

第一个要配置的是你个人的用户名称和电子邮件地址。这两条配置很重要，每次 Git 提交时都会引用这两条信息，说明是谁提交了更新，所以会随更新内容一起被永久纳入历史记录：

::

    $ git config --global user.name "John Doe"
    $ git config --global user.email johndoe@example.com

如果用了 ``--global`` 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。如果要在某个特定的项目中使用其他名字或者电邮，只要去掉 ``--global`` 选项重新配置即可，新的设定保存在当前项目的 ``.git/config`` 文件里。

**文本编辑器**

Git 需要你输入一些额外消息的时候，会自动调用一个外部文本编辑器给你用。默认会使用操作系统指定的默认编辑器，一般可能会是 Vi 或者 Vim。如果你有其他偏好，比如 Emacs 的话，可以重新设置：

::

    $ git config --global core.editor emacs

**差异分析工具**

在解决合并冲突时使用哪种差异分析工具。比如要改用 vimdiff 的话：

::

    $ git config --global merge.tool vimdiff

Git 可以理解 kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和 opendiff 等合并工具的输出信息。当然，你也可以指定使用自己开发的工具。

**查看配置信息**

要检查已有的配置信息，可以使用 ``git config --list`` 命令。

有时候会看到重复的变量名，那就说明它们来自不同的配置文件（比如 /etc/gitconfig 和 ~/.gitconfig），不过最终 Git 实际采用的是最后一个。

也可以直接查阅某个环境变量的设定，只要把特定的名字跟在后面即可，像这样：

::

    $ git config user.name
    Scott Chacon

2. Git基础
--------------

**忽略某些文件**

文件 ``.gitignore`` 的格式规范如下：

- 所有空行或者以注释符号 ``#`` 开头的行都会被Git忽略。
- 可以使用标准的glob模式匹配。
- 匹配模式最后跟反斜杠（ ``/`` ）说明要忽略的是目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加惊叹号（ ``!`` ）取反。

所谓的glob模式是指shell所使用的简化了的正则表达式。星号（ ``*``
）匹配零个或多个任意字符； ``[abc]``
匹配任何一个列在方括号中的字符（这个例子要么匹配一个a，要么匹配一个b，要么匹配一个c）；
问号（ ``?``
）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配。

``.gitignore`` 示例：

::

    # 此为注释 – 将被 Git 忽略
    # 忽略所有 .a 结尾的文件
    *.a
    # 但 lib.a 除外
    !lib.a
    # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
    /TODO
    # 忽略 build/ 目录下的所有文件
    build/
    # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
    doc/*.txt

**查看已暂存和未暂存的更新**

要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 ``git diff`` 。

此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没暂存
起来的变化内容。


若要查看已经暂存起来的文件和上次提交时的快照之间的差异，可以用 ``git diff --cached``
命令。

**跳过使用暂存区域**

尽管使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。Git提供了
一个跳过使用暂存区域的方式，只要在提交的时候，给 ``git commit`` 加上 ``-a``
选项，Git就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 ``git add``
步骤。

**移除文件**

如果我们想把文件从Git仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。
换句话说，仅是从跟踪清单中删除。比如一些大型日志文件或者一堆 ``.a`` 编译文件，不小心
纳入仓库后，要移除跟踪但不删除文件，以便稍后在 ``.gitignore`` 文件中补上，
用 ``--cached`` 选项即可：

::

    $ git rm --cached readme.txt

后面可以列出文件或者目录的名字，也可以使用glob模式，如：

::

    $ git rm log/\*.log

撤销操作
^^^^^^^^^^^

**修改最后一次提交**

有时候我们提交完了才发现漏掉了几个文件没有加，或者提交信息写错了。想要撤销刚才的提交操作，
可以使用 ``--amend`` 选项重新提交：

::

    $ git commit --amend

此命令将使用当前的暂存区域快照提交。如果刚才提交完没有作任何改动，直接运行此命令
的话，相当于有机会重新编辑提交说明，但将要提交的文件快照和之前的一样。

启动文本编辑器后，会看到上次提交时的说明，编辑它确认没问题后保存退出，就会使用新的
提交说明覆盖刚才失误的提交。

如果刚才提交时忘了暂存某些修改，可以先补上暂存操作，然后再运行 ``--amend``
提交：

::

    $ git commit -m 'initial commit'
    $ git add forgotten_file
    $ git commit --amend

上面的三条命令最终只是产生一个提交，第二个提交命令修正了第一个的提交内容。

**取消已经暂存的文件**

::

    $ git reset HEAD <file>

**取消对文件的修改**

::

    $ git checkout -- <file>

记住，任何已经提交到Git的都可以被恢复。即便在已经删除的分支中的提交，
或者用 ``--amend`` 重新改写的提交，都可以被恢复（关于数据恢复的内容见第九章）。
所以，你可能失去的数据，仅限于没有提交过的，对Git来说它们就像从未存在过一样。

远程仓库的使用
^^^^^^^^^^^^^^^^^^

**查看当前的远程库**

要查看当前配置有哪些远程仓库，可以用 ``git remote``
命令，它会列出每个远程库的简短名字。在克隆完某个项目后，至少可以看到一个名为origin的远程库，
Git默认使用这个名字来标识你所克隆的原始仓库。

也可以加上 ``-v`` 选项，显示对应的克隆地址：

::

    $ git remote -v
    origin  git://github.com/schacon/ticgit.git

**添加远程仓库**

要添加一个新的远程仓库，可以指定一个简单的名字，以便将来引用，运行 ``git remote
add [shortname] [url]`` 。

**查看远程仓库信息**

我们可以使用命令 ``git remote show <remote-name>``
来查看某个远程仓库的详细信息。

**远程仓库的删除和重命名**

在新版Git中可以用 ``git remote rename`` 命令修改某个远程仓库在本地的简称，
比如想把 ``pb`` 改成 ``paul`` ，可以这么运行：

::

    $ git remote rename pb paul
    $ git remote
    origin
    paul


碰到远端仓库服务器迁移，或者原来的克隆镜像不再使用，又或者某个参与者不再贡献代码，
那么需要移除对应的远端仓库，可以运行 ``git remote rm`` 命令：

::

    $ git remote rm paul
    $ git remote
    origin

打标签
^^^^^^^

**列出已有的标签**

直接运行 ``git tag`` 即可。

显示的标签按字母顺序排列，所以标签的先后并不表示重要程度的轻重。

**新建标签**

Git使用的标签有两种类型：轻量级的（lightweight）和含附注的（annotated）。轻量级标签就像是个不会变化的分支，实际上它就是个
指向特定提交对象的引用。而含附注标签，实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件
地址和日期，以及标签说明，标签本身也允许使用GNU Privary Guard（GPG）来签署或炎症。一般我们建议使用含附注型的标签，以便
保留相关信息。如果只是临时性加注标签，或者不需要旁注额外信息，用轻量级标签页没问题。

**含附注的标签**

创建一个含附注类型的标签非常简单，用 ``-a`` （取 ``annotated`` 的首字母）指定标签名字即可：

::

    $ git tag -a v1.4 -m "my version 1.4"
    $ git tag
    v0.1
    v1.3
    v1.4

而 ``-m`` 选项则指定了对应的标签说明，Git会将此说明一同保存在标签对象中。如果没有给出该选项，Git会启动文本编辑软件供你
输入标签说明。

可以使用 ``git show`` 命令查看相应的标签的版本信息，并连同显示打标签时的提交对象。

**签署标签**

如果你有自己的私钥，还可以用GPG来签署标签，只需要把之前的 ``-a`` 改为 ``-s`` （取 ``signed`` 的首字母）即可。

**轻量级标签**

轻量级标签实际上就是一个保存着对应提交对象的校验和信息的文件。要创建这样的标签， ``-a`` ， ``-s`` 或 ``-m`` 选项都不用，
直接给出标签名字即可：

::

    $ git tag v1.4-lw
    $ git tag
    v0.1
    v1.3
    v1.4
    v1.4-lw
    v1.5

**验证标签**

可以使用 ``git tag -v [tag-name]`` （取 ``verify`` 的首字母）的方式验证已经签署的标签。此命令会调用GPG来验证签名，所以你
需要有签署者的公钥，存放在keyring中，才能验证。

**后期加注标签**

可以在后期对早先的某次提交加注标签。只要在打标签的时候跟上对应提交对象的校验和（或前几位字符）即可：

::

    $ git tag -a v1.2 9fceb02

**分享标签**

默认情况下， ``git push`` 并不会把标签传送到远端服务器上，只有通过显式命令才能分享标签到远端仓库。其命令格式如同推送分支，
运行 ``git push origin [tagname]`` 即可。

如果要一次推送所有本地新增的标签上去，可以使用 ``--tags`` 选项。

现在，其他人克隆共享仓库或拉取数据同步后，也会看到这些标签。

技巧和窍门
^^^^^^^^^^^^^

**自动补全**

如果你用的是Bash shell，可以试试看Git提供的自动补全脚本。下载Git源码，进入 ``contrib/completion`` 目录，会看到一个
``git-completion.bash`` 文件。将此文件复制为你自己的用户主目录中的 ``.git-completion.bash`` 文件，并把下面一行内容添加到
你的 ``.bashrc`` 文件中：

::

    source ~/.git-completion.bash

如果在Windows上安装了msysGit，默认使用的Git Bash就已经配好了这个自动补全脚本，可以直接使用。

**Git命令别名**

Git并不会推断你输入的几个字符将会是哪条命令。如果想偷懒，少输入几个命令的字符，可以使用 ``git config`` 为命令设置别名。
例如：

::

    $ git config --global alias.co checkout
    $ git config --global alias.br branch
    $ git config --global alias.ci commit
    $ git config --global alias.st status

现在，如果要输入 ``git commit`` 只需键入 ``git ci`` 即可。

使用这种技术还可以创造出新的命令，比方说取消暂存文件时的输入比较繁琐，可以自己设置一下：

::

    $ git config --global alias.unstage 'reset HEAD --'     # 这里的reset HEAD -- 为什么不是reset HEAD？

这样一来，下面的两条命令完全等同：

::

    $ git unstage fileA
    $ git reset HEAD fileA

显然，使用别名的方式看起来更清楚。另外，我们还经常设置 ``last`` 命令：

::

    $ git config --global alias.last 'log -1 HEAD'

然后要看最后一次的提交信息，就变得简单多了：

::

    $ git last

可以看出，实际上Git只是简丹地在命令中替换了你设置的别名。不过有时候我们希望运行某个外部命令，而非Git的子命令，这个好办，
只需要在命令前加上 ``!`` 就行。如果你自己写了些处理Git仓库信息的脚本的话，就可以使用这种技术包装起来，例如设置 ``git visual``
启动 ``gitk`` 。

::

    $ git config --global alias.visual '!gitk'

3. Git分支
------------

和许多其他版本控制系统不同，Git鼓励在工作流程中频繁使用分支与合并，哪怕一天之内进行许多次都没有关系。

何谓分支
^^^^^^^^^^^

在Git中提交时，会保存一个提交（commit）对象，该对象包含一个指向暂存内容快照的指针，包含本次提交的作者等相关附属信息，包含
零个或多个指向该提交对象的父对象指针：首次提交时没有直接祖先的，普通提交有一个祖先，由两个或多个分支合并产生的提交则有多
个祖先。

暂存操作会对每一个要暂存的文件（确实发生变更或新增文件）计算校验和，然后把当前版本的文件快照保存到Git仓库中（Git使用blob
类型的对象存储这些快照），并将校验和加入暂存区域：

::

    $ git add README test.rb LICENCE
    $ git commit -m 'initial commit of my project'

当使用 ``git commit`` 新建一个提交对象前，Git会先计算每一个子目录的校验和，然后在Git仓库中将这些目录保存为树（tree）对象。
之后Git创建的提交对象，除了包含相关提交信息以外，还包含着指向这个树对象的指针，如此它就可以在将来需要的时候，重现此次快照
的内容了。

现在，Git仓库中有五个对象：三个表示文件快照内容的blob对象；一个记录着目录树内容及其中各个文件对应blob对象索引的tree对象；
以及一个包含指向tree对象（根目录）的索引和其他提交信息元数据的commit对象。

.. image:: /_static/pics/commit-ds.png

作些修改后再次提交，那么这次的提交对象会包含一个指向上次提交对象的指针。两次提交后，
仓库历史会变成如下图所示的样子：

.. image:: /_static/pics/commit-links.png

Git 中的分支，其实本质上仅仅是个指向 commit 对象的可变指针。Git 会使用 master
作为分支的默认名字。在若干次提交后，你其实已经有了一个指向最后一次提交对象的
master 分支，它在每次提交的时候都会自动向前移动。

.. image:: /_static/pics/master-pointer.png

那么，Git 又是如何创建一个新的分支的呢？答案很简单，创建一个新的分支指针。
比如新建一个 testing 分支，可以使用 ``git branch`` 命令：

::

    $ git branch testing

这会在当前commit对象上新建一个分支指针。

.. image:: /_static/pics/branch-master.png

那么，Git是如何知道你当前在哪个分支上工作的呢？其实答案很简单，它保存着一个名为
HEAD的特别指针。请注意它和你熟知的许多其他版本控制系统（比如Subversion）里的HEAD
概念大不相同。在Git中，它是一个指向你正在工作中的本地分支的指针（将HEAD想象为当前
分支的别名）。运行 ``git branch`` 命令仅仅是建立了一个新的分支，但不会自动切换到
这个分支中去。

.. image:: /_static/pics/HEAD-point-current.png

要切换到其他分支，可以执行 ``git checkout`` 命令。如切换到新建的testing分支：

::

    $ git checkout testing

这样HEAD就指向了testing分支。

.. image:: /_static/pics/HEAD-point-testing.png

由于Git中的分支实际上仅是一个包含所指对象校验和（40个字符长度SHA-1字串）的文件，
所以创建和销毁一个分支就变得非常廉价。说白了，新建一个分支就是向一个文件写入41个
字节（外加一个换行符）。

因为每次提交时都记录了祖先信息（即 ``parent``
对象），将来要合并分支时，寻找恰当的合并基础（即共同祖先）的工作其实已经自然而然
地摆在那里了，所以实现起来非常容易。Git鼓励开发者频繁使用分支，正是因为有着这些
特性作保障。

分支的新建与合并
^^^^^^^^^^^^^^^^^^

**分支的新建与切换**

要新建并切换到该分支，运行 ``git checkout`` 并加上 ``-b`` 参数：

::

    $ git checkout -b iss53
    Switched to a new branch "iss53"

这相当于执行下面这两条命令：

::

    $ git branch iss53
    $ git checkout iss53

如果当你在分支iss53中工作时，因为突发原因需要fix其他bug，唯一需要的仅仅是先切换回
master分支。
    
在切换分支之前，留心你的暂存区或者工作目录里，那些还没有提交的修改，它会和你即将
检出的分支产生冲突从而阻止Git为你切换分支。切换分支的时候最好保持一个清洁的工作
区域。目前已经提交了所有的修改，所以接下来可以正常转换到 master分支：

::

    $ git checkout master
    Switched to branch "master"

此时工作目录中的内容和你在解决问题 #53 之前一模一样。有牢记：Git会把工作目录的
内容恢复为检出某分支时它所指向的那个提交对象的快照。它会自动添加、删除和修改文件
以确保目录的内容和你当时提交时完全一样。

接下来创建一个紧急修补分支 ``hotfix`` 来开展fix工作，直到搞定：

::

    $ git checkout -b 'hotfix'
    Switched to a new branch "hotfix"
    $ vim index.html
    $ git commit -a -m 'fixed the broken email address'
    [hotfix]: created 3a0874c: "fixed the broken email address"
    1 files changed, 0 insertions(+), 1 deletions(-)

.. image:: /_static/pics/hotfix-branch.png

有必要作些测试，确保修补是成功的，然后回到 ``master`` 分支并把它合并进来，然后
发布到生产服务器。用 ``git merge`` 命令来进行合并：

::

    $ git checkout master
    $ git merge hotfix
    Updating f42c576..3a0874c
    Fast forward
     README |    1 -
     1 files changed, 0 insertions(+), 1 deletions(-)

请注意，合并时出现了“Fast forward”的提示。由于当前 master 分支所在的提交对象是
要并入的 hotfix 分支的直接上游，Git 只需把 master 分支指针直接右移。换句话说，
如果顺着一个分支走下去可以到达另一个分支的话，那么 Git 在合并两者时，只会简单地
把指针右移，因为这种单线的历史分支不存在任何需要解决的分歧，所以这种合并过程可以
称为快进（Fast forward）。

.. image:: /_static/pics/master-hotfix-same.png

在那个超级重要的修补发布以后，你想要回到被打扰之前的工作。由于当前hotfix分支和
master都指向相同的提交对象，所以hotfix已经完成了历史使命，可以删掉了。
使用 ``git branch`` 的 ``-d`` 选项执行删除操作：

::

    $ git branch -d hotfix
    Deleted branch hotfix (3a0874c)

现在回到之前未完成的 #53 问题修复分支上继续工作。

不用担心之前hotfix分支的修改内容尚未包含到iss53中来。如果确实需要纳入此次修补，
可以用 ``git merge master`` 把master分支合并到iss53；或者等iss53完成之后，
再将iss53分支中的更新并入master。

**分支的合并**

在问题#53相关的工作完成之后，可以合并回master分支。实际操作同前面合并hotfix分支
差不多，只需回到master分支，运行 ``git merge`` 命令指定要合并进来的分支：

::

    $ git checkout master
    $ git merge iss53
    Merge made by recursive.
     README |    1 +
     1 files changed, 1 insertions(+), 0 deletions(-)

但是这次合并操作的底层实现，并不同于之前hotfix的并入方式。因为这次你的开发历史
是从更早的地方开始分叉的。由于当前master分支所指向的提交对象（C4）并不是iss53
分支的直接祖先，Git不得不进行一些额外处理。就此例而言，Git会用两个分支的末端（C4
和C5）以及它们的共同祖先（C2）进行一次简单的三方合并计算。下图用红框标出了Git
用于合并的三个提交对象：

.. image:: /_static/pics/merge-three-part.png

这次，Git没有简单地把分支指针右移，而是对三方合并后的结果重新做一个新的快照，
并自动创建一个指向它的提交对象（C6）（见图 3-17）。这个提交对象比较特殊，它有
两个祖先（C4 和 C5）。

.. image:: /_static/pics/after-merge-three-part.png

**遇到冲突时的分支合并**

如果在不同的分支中都修改了同一个文件的同一部分，Git就无法干净地把两者合到一起
（译注：逻辑上说，这种问题只能由人来裁决）。如果你在解决问题 ``#53`` 的过程中
修改了 ``hotfix`` 中修改的部分，将得到类似下面的结果：

::

    $ git merge iss53
    Auto-merging index.html
    CONFLICT (content): Merge conflict in index.html
    Automatic merge failed; fix conflicts and then commit the result.

Git作了合并，但没有提交，它会停下来等你解决冲突。要看看哪些文件在合并时发生冲突，
可以用 ``git status`` 查阅。

在手动编辑解决了所有文件里的所有冲突后，运行 ``git add`` 将把它们标记为已解决
状态（译注：实际上就是来一次快照保存到暂存区域。）。因为一旦暂存，就表示冲突
已经解决。

分支的管理
^^^^^^^^^^^

- ``git branch`` ：列出当前所有分支的清单。分支名之前带 ``*``
  表示该分支为当前所在的分支。
- ``git branch -v`` ：查看各个分支最后一个提交对象的信息。
- ``git branch --merged``
  ：查看哪些分支已经被并入当前分支（也就是说哪些分支是当前分支的直接上游）。
- ``git branch --no-merged`` ：查看尚未合并到当前分支的分支。

对于尚未合并的分支，简单地使用 ``git branch -d`` 删除该分支会提示错误，因为那样
会丢失数据。不过，如果你确实想要删除该分支上的改动，可以用大写的删除选项 ``-D`` 
强制执行。

利用分支进行开发的工作流
^^^^^^^^^^^^^^^^^^^^^^^^^^^

远程分支
^^^^^^^^^^^

远程分支（remote branch）是对远程仓库中的分支的索引。它们是一些无法移动的本地
分支；只有在Git进行网络交互时才会更新。远程分支就像是书签，提醒着你上次连接
远程仓库时上面各分支的位置。

我们用 ``(远程仓库名)/(分支名)`` 这样的形式表示远程分支。比如我们想看看上次同
``origin`` 仓库通讯时 ``master`` 分支的样子，就应该查看 ``origin/master`` 分支。
如果你和同伴一起修复某个问题，但他们先推送了一个 ``iss53`` 分支到远程仓库，
虽然你可能也有一个本地的 ``iss53`` 分支，但指向服务器上最新更新的却应该是
``origin/iss53`` 分支。

举例说明：假设你们团队有个地址为 ``git.ourcompany.com`` 的Git服务器。如果你从
这里克隆，Git会自动为你将此远程仓库命名为 ``origin`` ，并下载其中所有的数据，
建立一个指向它的 ``master`` 分支的指针，在本地命名为 ``origin/master`` ，但你无法
在本地更改其数据。接着，Git建立一个属于你自己的本地 ``master`` 分支，始于
``origin`` 上 ``master`` 分支相同的位置，你可以就此开始工作：

.. image:: /_static/pics/clone-details.png

如果你在本地 ``master`` 分支做了些改动，与此同时，其他人向 ``git.ourcompany.com`` 
推送了他们的更新，那么服务器上的 ``master`` 分支就会向前推进，而于此同时，你在
本地的提交历史正朝向不同方向发展。不过只要你不和服务器通讯，你的 ``origin/master`` 
指针仍然保持原位不会移动：

.. image:: /_static/pics/different-move.png

可以运行 ``git fetch origin`` 来同步远程服务器上的数据到本地。该命令首先找到
``origin`` 是哪个服务器（本例为 ``git.ourcompany.com`` ），从上面获取你尚未拥有
的数据，更新你本地的数据库，然后把 ``origin/master`` 的指针移到它最新的位置上：

.. image:: /_static/pics/git-fetch.png

**推送本地分支**

要想和其他人分享某个本地分支，你需要把它推送到一个你拥有写权限的远程仓库。
你创建的本地分支不会因为你的写入操作而被自动同步到你引入的远程服务器上，
你需要明确地执行推送分支的操作。

如果你有个叫 ``serverfix`` 的分支需要和他人一起开发，可以运行
``git push (远程仓库名) (分支名)`` ：

::

    $ git push origin serverfix
    Counting objects: 20, done.
    Compressing objects: 100% (14/14), done.
    Writing objects: 100% (15/15), 1.74 KiB, done.
    Total 15 (delta 5), reused 0 (delta 0)
    To git@github.com:schacon/simplegit.git
     * [new branch]      serverfix -> serverfix

也可以运行 ``git push origin serverfix:serverfix`` 来实现相同的效果，它的意思
是“上传我本地的 serverfix 分支到远程仓库中去，仍旧称它为serverfix分支”。通过
此语法，你可以把本地分支推送到某个命名不同的远程分支：若想把远程分支叫作
``awesomebranch`` ，可以用 ``git push origin serverfix:awesomebranch`` 来推送数据。

接下来，当你的协作者再次从服务器上获取数据时，他们将得到一个新的远程分支
``origin/serverfix`` ，并指向服务器上 ``serverfix`` 所指向的版本：

::

    $ git fetch origin
    remote: Counting objects: 20, done.
    remote: Compressing objects: 100% (14/14), done.
    remote: Total 15 (delta 5), reused 0 (delta 0)
    Unpacking objects: 100% (15/15), done.
    From git@github.com:schacon/simplegit
     * [new branch]      serverfix    -> origin/serverfix

值得注意的是，在 ``fetch`` 操作下载好新的远程分支之后，你仍然无法在本地编辑
该远程仓库中的分支。换句话说，在本例中，你不会有一个新的 ``serverfix`` 分支，
有的只是一个你无法移动的 ``origin/serverfix`` 指针。

如果要把该远程分支的内容合并到当前分支，可以运行 ``git merge origin/serverfix`` 。
如果想要一份自己的 ``serverfix`` 来开发，可以在远程分支的基础上分化出一个新的
分支来：

::

    $ git checkout -b serverfix origin/serverfix
    Branch serverfix set up to track remote branch refs/remotes/origin/serverfix.
    Switched to a new branch "serverfix"

这会切换到新建的 ``serverfix`` 本地分支，其内容同远程分支 ``origin/serverfix`` 
一致，这样你就可以在里面继续开发了。

**跟踪远程分支**

从远程分支 ``checkout`` 出来的本地分支，称为 *跟踪分支* (tracking branch)。
跟踪分支是一种和某个远程分支有直接联系的本地分支。在跟踪分支里输入 ``git push`` ，
Git会自行推断应该向哪个服务器的哪个分支推送数据。同样，在这些分支里运行
``git pull`` 会获取所有远程索引，并把它们的数据都合并到本地分支中来。

在克隆仓库时，Git通常会自动创建一个名为 ``master`` 的分支来跟踪 ``origin/master`` 。
这正是 ``git push`` 和 ``git pull``
一开始就能正常工作的原因。当然，你可以像上例那样随心
所欲地设定为其它跟踪分支，比如 ``origin`` 上除了 ``master`` 之外的其它分支：
``git checkout -b [分支名] [远程名]/[分支名]`` 。如果你有1.6.2以上版本的Git，
还可以用 ``--track`` 选项简化：

::

    $ git checkout --track origin/serverfix
    Branch serverfix set up to track remote branch refs/remotes/origin/serverfix.
    Switched to a new branch "serverfix"

**删除远程分支**

如果不再需要某个远程分支了，比如搞定了某个特性并把它合并进了远程的 ``master`` 分支
（或任何其他存放稳定代码的分支），可以用这个非常无厘头的语法来删除它：
``git push [远程名] :[分支名]`` 。如果想在服务器上删除 ``serverfix`` 分支，
运行下面的命令：

::

    $ git push origin :serverfix
    To git@github.com:schacon/simplegit.git
     - [deleted]         serverfix


4. 服务器上的Git
------------------

7. 自定义Git
----------------

9. Git内部原理
-----------------
