---
title: go mod download踩坑
layout: post
---

### 前情提要

公司的项目是一个大仓包含着若干个微服务,而打包镜像到k8s时又是分开打包的,这就会出现一个问题:如果每次运行时都执行一遍`go mod tidy`下载依赖的话,每次打包都会下载巨量的依赖,只有部分是有用的,而且每次都下载构建速度又过于慢,所以决定要给他弄个缓存,虽然最后还是完成了,但是其中还是有不少值得注意的.

### 案件重演

由于构建镜像是分步的,所以第一时间想到的是在构建镜像的镜像里先下载依赖包.

```Dockerfile

COPY project /project

WORKDIR /project
RUN go mod tidy

```

这样看上去好像也能完成任务(虽然我没试过这样到底行不行,但是理论上是可行的),但是有更加优雅的方法.

可以使用 `go mod download` 命令来根据go.mod下载所有的依赖,所以下载依赖的可以是这样:

```Dockerfile

COPY project/go.mod /project/go.mod

WORKDIR /project
RUN go mod download -x # -x 表示输出下载的信息

```

这样依赖就下载到镜像中了,然后我们再提交一次,发现`go mod tidy`还是下载了一堆的依赖(正常来说本地有缓存这个命令是不会输出下载依赖的),是哪里出错了吗?

我在网上搜索了许多的内容,都没有答案.

于是我把目光放回了构建二进制的日志里,我发现对比没有预先下载依赖的tidy,有预先下载镜像的输出的下载依赖会更少一点,也就是说,download不会下载所有的依赖?

我去翻了一下go的ctl工具的help:

```bash
[~] # go mod download --help
usage: go mod download [-x] [-json] [-reuse=old.json] [modules]
```

好像也没有特别有用的信息,在网上也没有找到关于download的信息,于是我去问了一下deepseek,ai倒是给了我一个可能有用的答案:

```bash
go mod download -x all
```

为什么这个命令在go的ctl工具里没有提示呢?但是我还是先试试,没想到确实成了,`go mod tidy`执行之后确实没有输出了,构建也成功了.

### 复盘

为什么要使用`go mod download -x all`才可以呢?我在网上根本搜不到相关的信息,于是我又看了go的ctl的help信息,发现有这么一段:

```
With no arguments, download applies to the modules needed to build and test
the packages in the main module: the modules explicitly required by the main
module if it is at 'go 1.17' or higher, or all transitively-required modules
if at 'go 1.16' or lower.

如果没有参数，则 download 适用于构建和测试所需的模块
主模块中的包：主模块明确需要的模块
module （如果它是 'go 1.17' 或更高版本），或者所有传递必需的 modules
如果为 'Go 1.16' 或更低。
```

这里就更加表明了,默认就是会下载构建所需的依赖,但是为什么呢?
