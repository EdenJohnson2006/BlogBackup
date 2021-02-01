---
title: macOS上Hexo写作环境的搭建
date: 2020-06-27 01:27:12
tags: HEXO
categories: 科技
---

因为之前全盘装了黑苹果，所以自然对macOS下的环境有一些要求。

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627163435.png)

然后我的动态博客因为[数据删除]跑路而死亡了。

（我一定要问候服务商的Mother

然后我想起来我的GitHub学生账号

于是就想着用Hexo建博客。

好吧，废话不说，我们开始吧。

# 开端：Homebrew的安装。

如果你了解过Windows下的安装，你肯定会疑惑：en？Homebrew？不应该装Git和NodeJS？

您先安静，然后打开Homebrew的网站：https://brew.sh/index_zh-cn

然后往中间看

看到

> macOS（或 Linux）缺失的软件包的管理器

这一行字了吗？

我们就用Homebrew装Git和NodeJS。

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627171542.png)

主页中已经有代码了，直接复制到终端就好了。

> 警告：因为一堵墙树在你面前，所以你的速度可能会掉到100b左右甚至出现fatal: the remote end hung up unexpectedly fatal: early EOF问题 所以你需要足够的耐心多试几次。

> ~~我才不会告诉你我试过10次，2天才安装完成。~~

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627180538.png)

macOS自带Git，所以你只需要安装NodeJS就好了。

> 当然我的黑苹果是macOS Catalina 10.15.6 开发者预览，如果有装macOS 10.13 High Sierra的朋友我也不清楚具体情况，并且我翻源代码也有看见没有Git的提示，所以请各位自行检查，谢谢。

# 接续：NodeJS的安装

> ```
> brew install node
> ```

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627180815.png)

安装完成后，最好切换npm的源，否则你又会体验装Homebrew时候的绝望了

> ```
> npm config set registry https://registry.npm.taobao.org
> ```

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627181439.png)

> ```
> npm config get registry
> ```

如果返回如下：

> https://registry.npm.taobao.org

那么你就可以进行下一步了。

> ```
> npm install -g cnpm --registry=https://registry.npm.taobao.org
> ```

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627182701.png)

# 终了：Hexo及相关插件的安装

> ```
> cnpm install -g hexo
> ```

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627182904.png)

以及大多数人用Hexo都是要Git到某些地方作小站的，所以安装Git插件也是有必要的。

> （前提是Git前你需要建议一个工作目录，但是这不是本篇的主题，这边建议到 [https://chitang	.tk/2020/06/26/Hexo%E5%8D%9A%E5%AE%A2%E6%8A%98%E8%85%BE%E7%AC%94%E8%AE%B0/](https://chitang.tk/2020/06/26/Hexo博客折腾笔记/) 了解）
>
> （又商业互吹）

> ```
> >npm install hexo-deployer-git --save
> ```

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627184427.png)

然后你就可以Start Blogging With Hexo（官方语）了

# Some Advice：macOS下写Hexo博客的几个实用程序

## 1、VSCode

为什么选VSCode呢？因为VSCode可以添加文件夹，并且也有一定MarkDown支持，所以比较适合管理配置文件，简单写写文章之类的。

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627185128.png)

## 2、Typora

Typora就是专业的MarkDown编辑器了，有实时预览，也可以同时添加文件夹，所以我写文章主要会用Typora。

> Typora 在OS X还是Beta版本，但是我这几天用下来，并没有出现问题，所以还是推荐了过来。

![img](https://cdn.jsdelivr.net/gh/MEMZSONBILI/PicGoBed@master/images/20200627185159.png)

就写到这么多。关于MarkDown和其他Hexo事宜可以另找Windows文章。

（因为都是一个Hexo）

------

没了。



<!--  当然不可能完的。这个文章到建站之后才想起来写，中间黑苹果因为蓝牙驱动待机问题多次重启，我当时还以为之前的执行记录都没了，直到我看见了这个：恢复于... 才把这期文章保了下来。 -->