---
title: Android Gradle学习系列：（三）Build 环境配置--上篇
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - Android Studio
date: 2018-01-5 12:14:11
---

>前言

俗话说，工欲善其事，必先利其器。 接下来的，将对 Gradle Build 环境配置进程介绍。在正式的开始介绍之前，先对一些内容做些介绍，比如 Gradle 守护进程，为什么要使用守护进程以及进程的开启、状态、停止等。
<!--- more --->
<br/>

# 1、守护进程
<br/>
这是维基上对于守护进程的定义是这样的：

> 守护进程是一段运行在后台的计算机程序，并不在交互用户的直接操作下。

Gradle 运行在 JVM 上，并且使用一些支持库，这需要不少的初始化时间。因而，有时候看上去启动有些慢。解决的方法就是使用 Gradle *Daemon* ：一个常驻的后台进程，能够更快的执行构建任务。这样做可以避免启动进程的开销，还可以利用缓存，能够把一些项目数据放在内存中。开启守护与否在构建的操作上没什么不同。只是简单的配置你是否想开启它，其他的由 Gradle 进行处理，这些对于用户都是透明的。
<br/>
## 1.1、为什么使用守护进程
<br/>

守护进程是一个常驻进程，除了避免创建进程的开销，还能对于项目结构、文件、任务以及更多信息缓存在内存中。

原理非常简单：就是通过复用技术来加快构建速度。然而，好处是巨大的，按官方的数据是减少15~75%的构建时间。另外，推荐使用 *-\-profile* 选项来查看使用守护究竟做了什么。

从 Gradle 3.0 开始，守护是默认开启的。

另外需要注意的是，如果在容器这类的环境中进行持续集成构建，由于不能复用进程，使用守护进程会影响性能，需要关闭守护进程。

<br/>

## 1.2、守护进程状态

<br/>
使用 *-\-status* 选项可以获取守护进程以及其状态信息

例如：

```
  PID VERSION                 STATUS
28411 3.0                     IDLE
34247 3.0                     BUSY
```

目前，Gradle 只能链接对应版本的守护进程信息。后面，Gradle 会解决这一问题，会展现所有 Gradle 版本的守护信息。

<br/>

## 1.3、禁用守护进程

<br/>

守护进程默认是启用的，当然你也可以通过设置来禁用守护进程。常用的是修改 Gradle 用户 home 目录下的 *gradle.properties* 文件，添加以下信息：

```
org.gradle.daemon=false
```

注意：启动守护进程，所有的构建都能够获取性能的提升，无论 Gradle 是什么版本。

> 持续集成
>
> 从 Gradle 3.0开始，官方推荐无论是开发者的机器还是持续集成服务器都默认启用守护进程。然而，如果你怀疑守护会造成持续集成不稳定，你可以禁用该特性。

<br/>



## 1.4、停止守护进程

<br/>

守护进程是一个后台进程。尽管，它会监控内存使用情况，并会在进程空闲且系统内存紧张的情况下停止进程。但是，你可能还是会想要显式的去停止它，此时，你需要 *-\-stop* 命令。

这会终止所有该 Gradle 版本的所有守护进程。如果你装了 JDK，请使用 *jps* 命令来验证守护是否已停止。这些进程名为 *GradleDaemon*。

<br/>

## 1.5、FAQ

<br/>

### 1.5.1、禁用守护进程

<br/>

两种推荐方式：

- 通过环境变量：为环境变量 *GRADLE_OPTS* 添加标志 *-Dorg.gradle.daemon=false* 
- 通过属性文件：修改 *${USERS}/.gradle/gradle.properties* 文件，添加 *org.gradle.daemon=false*

这两种方式都可以。多数选择第二种。

一旦这样，守护进程将不会启动，除非显式的使用 *-\-daemon* 选项。

*-\-daemon* 和  *-\-no-daemon* 选项会在当前的构建中启用或禁用守护。这一选项具有最高优先级。
<br/>

### 1.5.2、机器上为什么有多个守护进程

<br/>
为什么不使用已有的守护进程，而是创建新的进程有几个原因。基本的准则是当没有空闲进程或匹配的进程时就会创建新的进程。Gradle 会杀死那些空闲时间超过 3 个小时的进程，因此你不必手动去清理。
- 空闲，是指目前没有执行构建或者有用的工作的进程
- 匹配，是指能够满足构建环境要求的进程。

守护进程可能不能满足构建环境的要求。例如，守护进程运行在 Java 7 运行时，要求的环境调用需要 Java 8，此时，会创建一个新的进程。此外，一旦 JVM 启动后，一些运行时属性无法去更改。例如，内存分配、默认编码、语言等等。

下面的 JVM 系统属性不可变。如果构建环境的有以下的要求，而守护进程的 JVM 值与之不同，守护进程就会不匹配。
- file.encoding
- user.language
- user.country
- user.variant
- java.io.tmpdir
- javax.net.ssl.keyStore
- javax.net.ssl.keStorePassword
- javax.net.ssl.keyStoreType
- javax.net.ssl.trustStore
- javax.net.ssl.trustStorePassword
- javax.net.ssl.trustStoreType
- com.sun.management.jmxremote


下列 JVM 属性，由启动参数控制，同样不可变。

- The maximum heap size (i.e. the -Xmx JVM argument)

- The minimum heap size (i.e. the -Xms JVM argument)

- The boot classpath (i.e. the -Xbootclasspath argument)

- The “assertion” status (i.e. the -ea argument)

另外一个问题是 Gradle 版本因素。守护进程与特定的 Gradle 运行时时耦合的。同一时刻，多个项目使用不同的 Gradle 版本是存在多个守护进程的常见原因。

<br/>

### 1.5.3、守护进程的内存分配

<br/>

如果没有为构建环境指定最大堆，守护进程使用的堆上限是 1GB。最小堆使用 JVM 的默认值。对于多数构建来说够用了。如果子项目、配置很多，想要更好的性能，需要更多的内存。

<br/>

### 1.5.4、停止守护进程

<br/>

通过操作系统或者执行 *-\-stop* 选项。

<br/>

### 1.5.5、守护进程出错

<br/>

守护进程可能会出错，这包括可能的内存泄漏或者全局状态错误。Gradle 会监控内存使用情况。一旦检测到问题，守护会结束掉当前的构建工作，并在下一次构建时重启守护进程。监控默认是启用的，当然也可以禁用：

```
org.gradle.daemon.performance.enable-monitoring = false
```

<br/>

### 1.5.6、守护进程如何加快构建速度

<br/>

守护进程会将一些信息加载到内存，然后在多次构建中复用来提升构建速度。另外一个原因是，它会缓存构建数据并进行输入输出的hash来进行增量构建。

<br/>

# 2、按需配置
<br/>




<br/>
# 3、-\-max-workers
<br/>

<br/>
# 4、日志
<br/>

<br/>
# 5、监控配置
<br/>

<br/>
# 6、缓存
<br/>

<br/>
# 7、构建输出
<br/>












