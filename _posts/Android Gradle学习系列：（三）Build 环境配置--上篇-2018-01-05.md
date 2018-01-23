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

一个 Gradle 项目可以由一个根项目和若干子项目组成。从构建的过程（生命周期）来看，有三个阶段：初始化、配置和执行。初始化阶段决定哪些项目参与构建，配置阶段进行项目对象的配置工作，最后是任务的执行阶段。

按需配置模式仅会对参与构建任务的项目进行配置，以便能够降低配置阶段的开销。该模式今后会成为默认的模式开启，该特性不能保证每次构建都能正确运行。多项目构建应该进行解耦。在按需配置模式下，项目应该按如下要求配置：

- 根项目必须进行配置，支持一些通用的配置（ *allprojects* 或者 *subprojects* 脚本块）
- 执行构建的目录下的项目应该进行配置，但是仅在 Gradle 没有任何任务执行时。当项目按需配置时，默认任务会被正确执行
- 支持标准的项目依赖且相关项目进行配置。如果项目 A 依赖项目 B，那么构建 A 会引起这两个项目的配置
- 支持任务路径进行任务的依赖并引起相关项目的配置。例如： *someTask.dependsOn(":someOtherProject:someOtherTask")*
- 在命令行中通过任务路径执行的任务会引起相关项目的配置。例如： 构建 *projectA:projectB:someTask* 会引起项目 projectB 的配置

该特性可以通过配置启用，在下一篇中介绍。


<br/>
# 3、日志
<br/>
## 3.1、日志级别
<br/>
日志的重要性无需赘述。有六种日志级别，常见的有两种：*QUIET* 和 *LIFECYCLE*。
日志级别见下面的表格：

| 级别        | 用途   |
| --------- | ---- |
| ERROR     | 错误信息 |
| QUIET     | 重要信息 |
| WARNING   | 警告信息 |
| LIFECYCLE | 进度信息 |
| INFO      | 信息   |
| DEBUG     | 调试信息 |

<br/>
你可以使用命令行选项来决定输出的日志级别：

| 选项           | 输出日志级别             |
| ------------ | ------------------ |
| 无            | LIFECYCLE 及更高      |
| -q 或 --quiet | QUIET 及更高          |
| -w 或 --warn  | WARN 及更高           |
| -i 或 --info  | INFO 及更高           |
| -d 或 --debug | DEBUG 及更高 (全部日志信息) |

<br/>
Stacktrace 命令行选项：

| 选项                     | 意义                  |
| ---------------------- | ------------------- |
| 无                      | 不会输出构建错误的栈信息，内部错误除外 |
| -s 或 --stacktrace      | 简要的栈信息打印，推荐方式       |
| -S 或 --full-stacktrace | 完整的栈信息打印            |

<br/>
## 3.2、打印日志信息
<br/>
简单的方式是使用标准输出，该日志级别为 *QUIET*。例如在 *build.gradle* 中：

```
println 'A message which is logged at QUIET level'
```



Gradle 在构建脚本中提供了 *logger* 属性，它是 *Logger* 的实例。例如在 *build.gradle* 中：

```
logger.quiet('An info log message which is always logged.')
logger.error('An error log message.')
logger.warn('A warning log message.')
logger.lifecycle('A lifecycle info log message.')
logger.info('An info log message.')
logger.debug('A debug log message.')
logger.trace('A trace log message.')
```



或者使用 *SLF4J* 来打印信息：

```
import org.slf4j.Logger
import org.slf4j.LoggerFactory

Logger slf4jLogger = LoggerFactory.getLogger('some-logger')
slf4jLogger.info('An info log message logged using SLF4j')
```



<br/>

## 3.3、外部工具/库的日志

<br/>

Gradle 内部使用 Ant 和 Ivy。因此有两套日志系统。然后把它们的日志输出定向到自己的日志系统来。当然，其他的工具也会有一些标准输出。默认情况，Gradle 会将标准输出定向到 *QUIET* 级别，标准错误定向到 *ERROR* 级别。 这些行为都是可配的。项目对象提供了 *LoggingManager* ，来帮助你把标准输出进行定向配置。

例如在 *build.gradle* 中配置标准输出捕获：

```
logging.captureStandardOutput LogLevel.INFO
println 'A message which is logged at INFO level'
```



在任务执行阶段改变标准输出、错误的日志级别，任务也提供了  *LoggingManager* ：

```
task logInfo {
    logging.captureStandardOutput LogLevel.INFO
    doFirst {
        println 'A task message which is logged at INFO level'
    }
}
```



Gradle 和很多 Java 日志工具库提供集成，例如 Jakarta、Log4j 。可以去参考下。

<br/>

## 3.4、改变日志内容

<br/>

这一部分主要是考虑到你可能有一些定制化的要求。比如日志格式、内容等等。你可以使用 [*Gradle.useLogger(java.lang.Object)*](https://docs.gradle.org/current/dsl/org.gradle.api.invocation.Gradle.html#org.gradle.api.invocation.Gradle:useLogger(java.lang.Object)) 方法进行日志输出的替换。这可以定义在构建脚本、初始脚本或者通过嵌入API。需要注意的时，它会禁用 Gradle 的默认输出。

使用初始脚本改变任务执行和构建完成时的日志，在 *init.gradle* 中：

```
useLogger(new CustomEventLogger())

class CustomEventLogger extends BuildAdapter implements TaskExecutionListener {

    public void beforeExecute(Task task) {
        println "[$task.name]"
    }

    public void afterExecute(Task task, TaskState state) {
        println()
    }
    
    public void buildFinished(BuildResult result) {
        println 'build completed'
        if (result.failure != null) {
            result.failure.printStackTrace()
        }
    }
}
```



使用 *-I* 选项指定初始脚本

```
> gradle -I init.gradle build
[compile]
compiling source

[testCompile]
compiling test source

[test]
running unit tests

[build]

build completed
3 actionable tasks: 3 executed
```



自定义的 logger 可以执行以下几个监听器接口。

- [`BuildListener`](https://docs.gradle.org/current/javadoc/org/gradle/BuildListener.html)
- [`ProjectEvaluationListener`](https://docs.gradle.org/current/javadoc/org/gradle/api/ProjectEvaluationListener.html)
- [`TaskExecutionGraphListener`](https://docs.gradle.org/current/javadoc/org/gradle/api/execution/TaskExecutionGraphListener.html)
- [`TaskExecutionListener`](https://docs.gradle.org/current/javadoc/org/gradle/api/execution/TaskExecutionListener.html)
- [`TaskActionListener`](https://docs.gradle.org/current/javadoc/org/gradle/api/execution/TaskActionListener.html)

<br/>

# 4、缓存
<br/>

缓存的好处，主要的考虑是加快数据的读取或者恢复任务。构建缓存也是一个缓存机制，重复使用构建的输出。缓存通常保存在本地或远程服务器，并且还要一定的检车机制，避免再次生成工作。

任务输出缓存使用了 *up-to-date* 检测。

默认情况，构建缓存是禁用的。两种方式可以开启缓存

- 使用 *-\-build-cache* 命令行选项，但只对本次构建使用
- 在 *gradle.properties* 文件中添加 *org.gradle.caching=true* 配置，每次都会有效，除非显式的使用 *-\-no-build-cache*

构建缓存存放在 Gradle 用户 home 目录，该位置可配。

<br/>










