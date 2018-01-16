---
title: Android Gradle学习系列：（二）命令行和 Gradle Wrapper
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - Android Studio
date: 2018-01-2 11:36:21
---

>前言

本篇将介绍一些Gradle相关知识、Gradle Wrapper 以及命令行工具的使用。
<!--- more --->
<br/>



# 1、.gradle 目录

在介绍 Gradle 命令行工具以及 Wrapper 之前，还是想说一下关于 Gradle 的一些其他一些相关知识。这里主要想讲一点关于 *.gradle* 目录的内容，对于 Android 开发者来讲，应该多少都有了解。首先我们来看一下这个文件夹，在 Mac 系统下它可能是隐藏的，你可以通过 Android Studio -> Preference -> Build,Execution… -> Build Tools -> Gradle 面板，勾选 *“Use local gradle distribution”* 填入 “/Users/xxxx/.gradle”，然后点击选择文件目录查看。

或者在 Terminal 中输入以下命令 ：

```
defaults write com.apple.finder AppleShowAllFiles -bool true
```

然后重启 Finder 就OK了。

现在，我们来看一下  *.gradle* 目录有哪些东西：

```
+ caches 
+ daemon 
+ native
+ workers
+ wrapper
gradle.properties
```

一般情况下，主要的目录和文件有这么几个。

- *caches* 目录下存放了下载的依赖包，你可以通过 Android Studio -> Preference -> Build,Execution… -> Build Tools -> Gradle 面板，勾选 *“Use offline”* 来减少重复下载依赖包，加快 build 或者 sync 过程。此外，你还是向这里添加你自己的依赖包
- *daemon* 目录存放 Gradle 守护进程的输入日志信息，以版本号文件夹的形式进行分类。
- *wrapper* 目录存放不同版本的 Gradle 包，Gradle Wrapper 会下载和使用这里的 Gradle 进行项目构建。由于敏感的原因，有时候会出现长时间的 building ，这可能是由于下载相应的 Gradle 太慢或者没响应造成的，建议去国内的站点下载相应的 Gradle 版本的 .zip文件，放在这个 wrapper/dists 目录下面，再次打开项目进行构建
- *gradle.properties* 文件，这一个 Gradle 全局配置文件，可以在这里配置守护、jvmargs、全局字段（比如keystore的信息）等
- 其他，比如 *native* 目录、worker 目录


OK，关于 Gradle 而言，关于 .gradle 目录基本要了解的内容就这么多，相比于 Maven 这类的依赖管理的工具来说，就是有一个配套的仓库（caches）来存放使用的依赖。多出来的一个重要内容就是多了一个自身版本的管理器，这个我们在第三部分来说。


<br/>

# 2、Gradle 命令行

平时的 Android 开发中，通过界面上的 *Run* 按钮来打包和安装 apk，这一些列动作，主要是靠 Gradle 命令来完成的。基本上，我们不太去直接使用命令行来进行打包。但是实际上，我们还是需要了解和学习这一过程。

<br>

# 3、Gradle Wrapper

之前说到 Android 项目基于 Gradle 以及 Plugin 进行构建，不同的项目的 Gradle 环境（版本）不同，同一个项目可能需要在一个 team 中进行开发，需要确定 Gradle 环境保持一致。因此，这就产生了 Gradle Wrapper。它是一个管理



