《Go Web编程》读书笔记
==========================

第一章：Go语言环境配置
----------------------------

**源码安装**

::

    hg clone -u release https://code.google.com/p/go
    cd go/src
    ./all.bash


运行all.bash后，出现“ALL TESTS PASSED”字样时才算安装成功。

然后设置以下几个环境变量

::

    export GOROOT=$HOME/go
    export GOBIN=$GOROOT/bin
    export PATH=$PATH:$GOBIN


**第三方工具安装**

GVM是第三方开发的Go语言多版本管理工具，类似ruby里面的rvm工具。使用非常方便，安装GVM使用如下命令。

::

    base < <(curl -s https://raw.github.com/moovweb/gvm/master/binscripts/gvm-installer)


安装完成后，如下安装go：

::

    gvm install go1.0.3
    gvm use go1.0.3


执行完上面的命令之后，GOPATH、GOROOT等环境变量会自动设置好，这样就可以直接使用了。


GOPATH与工作空间
^^^^^^^^^^^^^^^^^^^^^^^^

go命令依赖一个重要的环境变量：$GOPATH

GOPATH允许多个目录，当有多个目录时，请注意分隔符，多个GOPATH的时候，Windows系统是分号，Linux系统是冒号，当有多个GOPATH时，默认将go get的内容放在第一个目录下。

$GOPATH目录约定有三个子目录：

- src：存放源代码（比如：.go、.c、.h、.s等）
- pkg：编译后生成的文件（比如：.a）
- bin：编译后生成的可执行文件


**获取远程包**

Go语言有一个获取远程包的工具 ``go get`` ，目前 ``go get`` 支持多数开源社区（例如：github、googlecode、bitbucket、Launchpad）。

::

    go get github.com/astaxie/beedb

通过这个命令可以获取相应的源码，对应的开源平台采用不同的源码控制工具，例如，github采用git，googlecode采用hg，所以要想获取这些源码，必须先安装相应的源码控制工具。

``go get`` 本质上可以理解为：首先通过源码工具clone代码到src目录，然后执行 ``go install`` 。


Go语言命令
^^^^^^^^^^^^^^

**go build**

这个命令主要用于测试编译。在包的编译过程中，若有必要，会同时编译与之相关联的包。

- 如果是普通包，当你执行 ``go build`` 之后，不会产生任何文件。如果你需要在 ``$GOPATH/pkg`` 下生成相应的文件，则要执行 ``go install`` 。
- 如果是main包，当你执行 ``go build`` 之后，就会在当前目录下生成一个可执行文件。如果你需要在 ``$GOPATH/bin`` 下生成相应的文件，需要执行 ``go install`` ，或者使用 ``go build -o 路径/a.exe`` 。
- 如果某个项目文件夹下有多个文件，而你只想编译某个文件，就可在 ``go build`` 之后加上文件名