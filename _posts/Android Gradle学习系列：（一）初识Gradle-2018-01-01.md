---
title: Android Gradle学习系列：（一）初识Gradle
tags:
  - Android
  - Gradle
categories:
  - Gradle
date: 2018-01-01 20:23:31
---

>前言

从 Eclipse 转 Android Studio 已经有很长一段时间了，期间也尝试过从把公司的项目在两个 IDE 之间进行过转换，还是有一些坑和心得体会的。我们都知道 Android Studio 是 Google 力推的开发 IDE ，其中主要的特性之一是项目的构建是基于Gradle 的。一直以来都在使用各种 gradle.build 文件，对其中的一些东西还是掌握的不够。加之最近在看 Android 和 Gradle 的官方文档，索性就整合成一个系列，本系列会紧密结合 Android 开发。如需详细学习 Gradle，请阅读 [*用户指南*](https://docs.gradle.org/current/userguide/userguide.html)。

<!--- more --->

<br/>

# 1、Gradle介绍

<br/>

Gradle 是一个基于 JVM 的构建工具，功能类似于 Ant 或着 Maven 。它使用基于 Groovy 的特定领域语言（DSL）来声明项目的一些配置，不同于 Maven 使用 XML 进行项目配置。因此，它需要 JDK 的支持，这意味着它能运行于目前大多数操作系统之上；另一方面，Gradle 的核心是一个基于依赖编程的语言 *（ [We said earlier that the core of Gradle is a language for dependency based programming](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:project_evaluation).）*。

<br/>

# 2、Gradle安装

Gradle的安装，两种方式：

- 使用包管理器进行安装
- 手动安装，下载压缩包解压，需要配置环境变量

下载完，可以进行验证，这一部分请参考[*官方文档*](https://gradle.org/install/)
```
$ gradle -v
```

<br/>

# 3、Gradle的特性

Gradle 拥有众多的特性，来帮助用户进行快速构建项目，例如在性能方面：
- 增量构建*（Incremental Builds）*， 构建过的任务，当输入、输出或配置没有发生变化不会再次构建
- 任务输出缓存*（Task Output Caching）*，任务构建后的输出会被缓存起来，以便复用
- 编译器守护*（Compiler Daemon）*，Gradle 可以通过开启守护进程加快构建速度
- 并行执行*（Task Output Caching）*，Gradle 可以开启任务并行执行来获得性能的提升
- 依赖并行下载*（Task Output Caching）*

此外，还提供命令行接口进行任务执行、持续构建*（Continuous build）*，对依赖进行缓存、动态依赖、版本冲突解决，针对跨团队管理能够进行自配置构建环境（可以通过 Gradle wrapper指定 Gradle版本）。

<br/>

# 4、Android Plugin

针对不同生态，例如JVM、Android、C++、Swift、Objective-C，都有其特定的特性，这些特性主要是由相应的插件来完成，比如 Android 的 Build Tool，我们都会在每个 Android 项目的根目录下的 **build.gradle** 中看到这样的配置：

```
buildscript {
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.1'
    }
}
```
它指定了 Gradle Android Plugin 的版本信息，不同版本的插件版本有不同的特性，比如 *Instant Run* 需要 2.3.0 或更高的版本。因此，我们的Android项目的构建工作需要 Gradle 以及 Gradle Android Plugin 一起搭配来完成，两者版本的配置表可以参考 [*Android开发者官网#Update Gradle*](https://developer.android.google.cn/studio/releases/gradle-plugin.html#updating-plugin) 。通过 Android Plugin， 你可以使用到多种（功能）[*特性*](https://gradle.org/features/#android-applications)：
- 构建变体*（Build Variants）*，通过构建类型和产品风味的配置来完成
- 配置清单合并*（Manifest Merging）*，完成Manifest文件以及资源文件的自动合并
- Android 库工程*（Android Library Projects）*，支持多个工程（multi-projects）的构建
- 变体依赖管理*（Per-Variant Dependency Management）*，可以针对每个变体进行依赖声明
- 应用签名*（Application Signing）*，为应用进行自动签名
- 混淆支持*（ProGuard Support）*，轻松为应用进行代码混淆和压缩APK
- 单元测试*（Unit Testing）*，支持单元测试
- 设备功能测试*（On-device Functional Testing）*
- APK分片*（APK Splits）*，对于不同分辨率的设备或 *ABIs (Application Binary Interfaces)* 生成APK，并复用构建输出
- Multidex支持*（Multidex Support）*，避免单个 DEX 文件方法超过 65k 的限制
- NDK支持*（NDK Support）*，可以基于本地库进行 Android 开发
- 数据绑定*（Databinding）*，使用数据绑定能够自动的把数据更新到 View 上

Gradle Android Plugin 的源码可以通过在 app 工程的 *build.gradle* 中添加依赖：
```
dependencies {
        compile 'com.android.tools.build:gradle:2.3.1'
    }
```
然后在 *External Libraries* 中查看插件源码，实现部分可以参考[Android Gradle Plugin解析](http://blog.csdn.net/qq_23012315/article/details/74230956)。

其他的生态特性，本文就不在介绍，有兴趣的可以去了解 [*Gradle Features*](https://gradle.org/features/)。

<br/>

# 5、写在最后

本篇介绍了 Gradle 的核心作用以及功能特性。Android 主要是靠 Gradle 以及附加的插件 Android Plugin 共同完成项目的构建工作的。

下一篇，我们将介绍 Gradle 的一些相关知识、命令行工具 *（command-line）*和 Gradle Wrapper。

<br/>


