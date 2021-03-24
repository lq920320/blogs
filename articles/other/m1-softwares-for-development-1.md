## M1 安装日志（一）：基础开发软件列表

> M1 兼容软件查询网站：https://doesitarm.com/

没错，年初的时候，我换电脑了。开发效率确实提升了不少，我之前的电脑开机一次 3 分钟，而且打字的时候太卡啦！终于，不用等那么久了。不过，话又说回来，毕竟，这么贵！！！

这次就先列一下开发相关的一些软件列表，对于基本的前后端开发还是没啥问题的，后续可能会继续跟进一些运维软件的安装。

#### 1、HomeBrew

Homebrew是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。这个在安装过程中可能会遇到一些问题，不过通过百度是可以解决的。比如，安装 `git` 的时候缓存不足之类的。

#### 2、git

通过 homebrew 安装即可：

```shell
$ brew install git
```
git-gui 的安装是不兼容的，所以在暂时只能用命令行：

```shell
$ brew install git-gui
```

#### 3、IntelliJ IDEA

本身就支持，直接下载安装就行

#### 4、VS code

直接下载安装即可。

#### 5、MySQL

原生支持。但是 `workbench` 在我的电脑上安装并不兼容，于是我只能暂时使用 `DataGrip` 代替。

#### 6、openjdk

可以下载 `zulu` 社区的 `openjdk` 版本，下载地址：

```
https://www.azul.com/downloads/zulu-community/?os=macos&architecture=arm-64-bit&package=jdk
```

#### 7、NodeJS

直接下载安装即可。

#### 8、Maven 以及 Gradle

`maven` 下载安装包即可，`gradle` 的兼容性未知，我也没试。但是，`gradle` 的项目在 IDEA 中打开，IDEA 是集成了 `gradle` 的，可以正常打开运行。

#### 9、golang

直接最新版。开发工具，当然是 `VS code`！

#### 10、文本编辑工具：Sublime Text 、 Typora 和 金山软件

总之，基本的文本编辑都是兼容的。

#### 11、浏览器：Chrome 和 FireFox

直接安装就行。

#### 12、Postman

直接下载安装。

-------

这是一条分割线。

截止到现在，以上软件基本上可以满足开发工作的需求，像 `Redis`、`Kafka`、`ElasticSearch` 这些，大家可以考虑自行安装，都是支持的。除了一些类似虚拟机，`Docker` 之类的我还没去尝试和探索，据说是不兼容，以后有机会再试试吧。