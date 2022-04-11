layout: post
title: "编译WebAssembly版本的FFmpeg（ffmpeg.wasm）：（1）准备"
date: 2022-04-11 20:20:20 -0000
categories: 前端 音视频

# **编译WebAssembly版本的FFmpeg（ffmpeg.wasm）：（1）准备**

![img](https://wdoc-76491.picgzc.qpic.cn/MTY4ODg1MTM0NjExMTM4Nw_618614_14G0me1I5fpNj9w6_1649647915?w=640&h=360)

> 作者：[Jerome Wu](https://jeromewus.medium.com/?source=post_page-----ed12bf4c8fac-----------------------------------)
>
> 原文链接：[Build FFmpeg WebAssembly version (= ffmpeg.wasm): Part.1 Preparation](https://itnext.io/build-ffmpeg-webassembly-version-ffmpeg-js-part-1-preparation-ed12bf4c8fac)
>
> 译者：[Yodoxu](https://github.com/Yodonicc)
>
> 2020年8月更新：教程已更新，可在MacOS中运行。

在这一部分中，你将了解到：

1. 这个系列的背景

2. 如何用Docker构建原生的FFmpeg（以及在MacOS中不使用docker）。

### 本系列的背景

这个系列的文章旨在为以下目的服务：

1. 为那些想学习如何使用Emscripten将C/C++库编译成JavaScript的人提供指南（希望是目前最有用、最详细的指南）

2. 个人笔记

### 为什么是FFmpeg？

FFmpeg是一个免费的开源项目，由一套庞大的软件库和程序组成，用于处理视频、音频、其他多媒体文件和流。(来自维基百科)

它是一个有用的库，没有一个JavaScript库具有完全相同的功能。如果你在谷歌上搜索 "ffmpeg.js"，你会发现很少有与我们将要建立的库完全相同的现有库。

- ffmpeg.js: https://github.com/Kagami/ffmpeg.js

- videoconverter.js: https://github.com/bgrins/videoconverter.js

这些库很好，在大多数情况下都可以使用，但它们存在以下问题。

1. FFmpeg和Emscripten的版本都已经过时了。

2. 多年来没有积极维护。(Kagami/ffmpeg.js在2020年4月继续其开发)

我考虑过也许可以接管其中一个仓库，但由于这些年变化太大，我决定从头开始，同时写了这个系列的教程，帮助人们学习如何在现实（工程）世界的C/C++库中使用Emscripten。

### 如何用Docker构建原生FFmpeg

首先，我们需要从FFmpeg的仓库中克隆源代码，由于主分支（master）正在开发中，我们最好选择一个特定的版本来编译。

在我写这个文章的时候，FFmpeg的最新稳定版本是n4.3.1，所以我们将在文章中使用这个版本。

```
	# Use depth=1 to download only the latest version
	git clone --depth 1 --branch n4.3.1 https://github.com/FFmpeg/FFmpeg
```

在完成克隆版本库后，是时候用GCC构建以确保它的工作。

实际上，如果你很着急的话，你可以跳过这一部分，但根据我的经验，最好先熟悉一下库的构建系统。

构建和安装FFmpeg的说明可以在版本库根目录下的INSTALL.md中找到。

```
Installing FFmpeg:
1. Type `./configure` to create the configuration. A list of configure options is printed by running `configure -help`.`configure` can be launched from a directory different from the FFmpeg sources to build the objects out of tree. To do this, use an absolute path when launching `configure`, e.g. `/ffmpegdir/ffmpeg/configure`.

2.Then type `make` to build FFmpeg. GNU Make 3.81 or later is required.

3.Type `make install` to install all binaries and libraries you built.

NOTICE
- - -
	- Non system dependencies (e.g. libx264, libvpx) are disabled by default.
```

由于我们不需要实际安装FFmpeg，只需要步骤1和2。

有两种构建方式，一种是原生方式，需要你安装软件包（如emsdk，Node.js）。大多数时候，它是有效的，但有时你可能会面临错误，由于包的版本和操作系统的变化而难以解决。另一种方法是使用Docker，它提供了一个稳定和静态的构建环境。我们强烈建议使用Docker，因为它可以节省你安装（和删除）软件包的时间。

我不会在这里介绍如何安装软件包，但由于我把脚本分成`build.sh`和`build-with-docker.sh`，你可以自己安装所有的软件包并运行`build.sh`。

> 为了确保本教程能够达到最大的环境覆盖率（支持更多的操作系统），我使用Github Actions来测试它在Linux和MacOS上是否有效。对于Linux用户，我将使用Docker方式`/build-with-docker.sh`来构建。对于MacOS用户，由于Github Actions不支持Docker，我将使用本地方式`/build.sh`进行构建。

现在，让我们创建一个名为`build.sh`的文件，内容如下。

```
#!/bin/bash -x
# gcc 8 is used in this tutorial, other versions may fail
./configure --disable-x86asm
make -j
```

要以本地方式构建，你只需要运行命令：

```
$ bash build.sh
```

要用Docker构建，创建一个名为`build-with-docker.sh`的文件，内容如下。

```
#!/bin/bash -x
docker pull gcc:8
docker run \
  -v $PWD:/usr/src \
  gcc:8 \
  sh -c 'cd /usr/src && bash build.sh'
```

然后运行命令：

```
bash build-with-docker.sh
```

> `--disable-x86asm`是必须的，因为我们不打算使用x86装配功能。
>
> 根据你的网速和电脑的硬件规格，可能需要10~30分钟才能完成编译。
>
> 在编译过程中看到大量的警告是正常的，因为gcc 9引入了更多的限制条件。

它应该需要一些时间来编译本地的FFmpeg。如果一切正常，你应该可以用下面的命令运行`ffmpeg`。

```
$ ./ffmpeg
```

你应该能看到类似于这样的结果:

> *ffmpeg version n4.3.1 Copyright © 2000–2019 the FFmpeg developers**
>
> **built with gcc 8 (GCC)*
>
> configuration: — disable-x86asm
>
> *libavutil 56. 22.100 / 56. 22.100*
>
> libavcodec 58. 35.100 / 58. 35.100
>
> *libavformat 58. 20.100 / 58. 20.100*
>
> libavdevice 58. 5.100 / 58. 5.100
>
> *libavfilter 7. 40.101 / 7. 40.101*
>
> libswscale 5. 3.100 / 5. 3.100
>
> *libswresample 3. 3.100 / 3. 3.100*
>
> Hyper fast Audio and Video encoder
>
> *usage: ffmpeg [options] [[infile options] -i infile]… {[outfile options] outfile}…*
>
> 
>
> Use -h to get full help or, even better, run ‘man ffmpeg’*

你可以在这里访问Github库，了解它的工作细节：https://github.com/ffmpegwasm/FFmpeg/tree/n4.3.1-p1

并随时在这里下载构建代码：https://github.com/ffmpegwasm/FFmpeg/releases/tag/n4.3.1-p1

现在我们已经完成了准备工作，让我们继续**编译WebAssembly版本的FFmpeg（ ffmpeg.wasm）：（2）用Emscripten编译**，开始用Emscripten编译FFmpeg。😃

注：**特别感谢技术指导dazhao(赵达)对本文翻译的审阅指正**。