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



# 1、写在前面

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

- *caches* 目录下存放了下载的依赖包，你可以通过 Android Studio -> Preference -> Build,Execution… -> Build Tools -> Gradle 面板，勾选 *“Use offline”* 来减少重复下载依赖包，加快 build 或者 sync 过程。*daemon* 目录





<br/>

# 2、命令行



<br>

# 3、Gradle Wrapper



