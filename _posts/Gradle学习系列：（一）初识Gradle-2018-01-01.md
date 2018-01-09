---
title: Gradle学习系列：（一）初识Gradle
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - Android Studio
date: 2018-01-01 20:23:31
---

>前言

从 Eclipse 转 Android Studio 已经有很长一段时间了，期间也尝试过从把公司的项目在两个 IDE 之间进行过转换，还是有一些坑和心得体会的。我们都知道 Android Studio 是 Google 力推的开发 IDE ，其中主要的特性之一是项目的构建是基于Gradle 的。一直以来都在使用各种 gradle.build 文件，对其中的一些东西还是掌握的不够。加之最近在看 Android 和 Gradle 的官方文档，索性就整合成一个系列。

<!--- more --->

<br/>

# 1、Gradle介绍

<br/>

Gradle 是一个基于 JVM 的构建工具，功能类似于 Ant 或着 Maven 。它使用基于 Groovy 的特定领域语言（DSL）来声明项目的一些配置，不同于 Maven 使用 XML 进行项目配置。因此，它需要 JDK 的支持，这意味着它能运行于目前大多数操作系统之上；另一方面，其核心是一个基于依赖编程的语言 *（ [We said earlier that the core of Gradle is a language for dependency based programming](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:project_evaluation).）*。

<br/>

## 1.1、Gradle安装