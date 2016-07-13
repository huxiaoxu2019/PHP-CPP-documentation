# 如何安装PHP-CPP
在使用PHP-CPP来构建自己飞快的、原生的PHP扩展时，首先你不得不在系统上安装PHP-CPP库。

幸运的是，对于我们来说（那些使用Linux或者Apple环境的人），这就是小菜一碟。然而，如果你使用的是其他系统，那么你只能靠自己了（you are left on your own）。因为我们（指的是PHP-CPP的开发者），仅使用Linux系统。所以，这也没有任何理由来说明为什么这些库不应该运行在其他平台的言论，仅仅是因为它是由纯C++代码写的。因此，如果你在使用其他的系统，同时一直在设法编译它，那么请告知我们。我们来更新这些安装文档，并包含一些其他平台。

## 下载
安装的第一件事是下载源码。你既可以从我们的[下载](http://www.php-cpp.com/download)页面下载最新的发布版本，也可以从[GitHub](https://github.com/CopernicaMarketingSoftware/PHP-CPP)中下载最新的开发者版本（work-in-progress）。

为了获取最新的GitHub版本，运行如下行的命令：

```$ git clone https://github.com/CopernicaMarketingSoftware/PHP-CPP.git```

在你下载了软件之后（要么从我们的网站，要么直接从GitHub），改变当前的工作路径到PHP-CPP库的路径，然后用你熟悉的编辑器打开“Makefile”文件。

Makefile包含着编译的配置和介绍说明。在96%的情况下，默认的Makefile配置内容已经是非常适合你了，当然最好还是浏览下它，同时做一些（小小）的改变。比如，你可以改变安装路径，将要使用的编译器。

在你检查过Makefile的配置内容全部正确之后，就可以构建软件了。在PHP-CPP目录下执行下面的命令。

```$ make```

上述命令会启动编译器并且构建这些库。

## 在OSX上编译

如果你在OSX系统上编译这个软件，你可能会遇到链接（linking）和“未解决标识符（unresolved symbol）”的错误。这种情况下你需要修改Makefile文件。在Makefile的文件中的有一个叫做“LINKER_FLAGS“的选项，这个选项应该扩展，同时，应该将这个额外的标志”-undefined dynamic_lookup“加进去。

在你运行”make“之后，PHP-CPP的库已经构建完毕，剩下的事情就是将它安装到你的系统中。你可以使用”make install“命令。应该使用root权限来执行这个命令，要么使用”sudo“，要么先以root角色登陆。



