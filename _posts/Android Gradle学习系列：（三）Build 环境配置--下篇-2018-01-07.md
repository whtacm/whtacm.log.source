---
title: Android Gradle学习系列：（三）Build 环境配置—下篇
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - Android Studio
date: 2018-01-7 19:55:23
---

>前言

有了上一篇的介绍工作，本篇将正式介绍 Gradle Build（构建） 环境配置的相关内容了。
<!--- more --->
<br/>

# 1、配置 Build 环境
<br/>

Gradle提供了一系列的选项，让你能够更容易的对执行构建的 Java 进程进行配置。通过 *GRADLE_OPTS* 或者 *JAVA_OPTS* 的本地环境配置，JVM 内存设置、Java home，开启守护*（daemon）*进程等操作，如果这些统统可以在版本管理系统*（VCS）*中进行控制，这将有利于团队在一个一致的开发环境中工作。设置这样一个环境只需要简单的进行 *gradle.properties* 文件进行配置。有多处文件可以进行配置，最终配置的生效会按下面的顺序（即同一个配置如果出现多个地方，最后一个位置生效）。

- 项目目录中的 *gradle.properties* 文件

- gradle 用户目录下的  *gradle.properties* 文件，即 *$User/.gradle/* 目录下

- 系统配置，例如在命令行中设置 *-Dsome.property* 

当设置这些属性时，你需要注意的是 Gradle 需要 Java JDK 或 JRE  7 或更高的版本。

下面的这些属性可以用来进行 Gradle 构建环境的配置：

>org.gradle.daemon

当设置为 *true* 时，Gradle 守护进行将在执行构建的时候被使用。从 Gradle 3.0 开始，默认是开启的，这也是官方所推荐的方式。

>org.gradle.java.home

指定 Gradle 构建进程的Java home。值可以是 jdk 或者 jre 的位置，然而，这取决于你构建的内容，***设置成 jdk 会更加保险一点***。如果没有设置将会使用默认。

> org.gradle.jvmargs

指定守护进程使用的 jvmargs。该设置对于内存调整特别有用。

>org.gradle.configureondemand

新的 *incubating mode* 在配置项目时增加了 Gradle 的可选择性。只有配置相关的项目能够在多项目中加速构建过程。

> org.gradle.parallel

配置该属性，Gradle 决定是否允许并行模式。

>org.gradle.workers.max

配置 Gradle 可以使用的最大 worker 数，默认是处理器数 *（number of processors）*。

>org.gradle.logging.level

设置日志级别，例如 quiet 、warn 、lifecycle 、info 或者 debug。该值对大小写不敏感。

>org.gradle.debug

该属性设置为 *true* 时，Gradle 会允许在构建时开启远程调试，端口为 *5005*。这等价于使用 JVM 命令行 *-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005* ，这回在调试点将虚拟机挂起。

>org.gradle.daemon.performance.enable-monitoring

是否开启对守护进程内存使用情况的监视。

>org.gradle.caching

设置为 *true*，Gradle 将尝试重用之前构建的输出。

>org.gradle.console

设置 Gradle 使用不同类型的控制台，候选值为 *plain、auto或者 rich*。
<br/>
# 1.1、Forcked Java Process
<br/>
许多的设置（例如 Java 版本和最大堆）只能在启动新的 JVM 进程时使用。这意味着在解析 *gradle.properties* 文件后，Gradle 必须要启动一个单独的 JVM 进程来执行构建任务。当运行守护进程时，一旦 JVM 运行就可以在每一次守护构建运行时被重复使用。假如没有守护进程，那么新的 JVM 将被启动，除非 Gradle 开始脚本启动的 JVM 有相同的参数。

每次构建执行的 JVM 启动都需要很大的开销，这也是为什么无论你是设置 *`org.gradle.java.home* 还是 *org.gradle.jvmargs*，我们都强烈建议你使用 Gradle 守护。


<br/>
# 2、Gradle 属性和系统属性
<br/>

Gradle 提供了多种方式来为你的构建添加属性设置。使用 *-D* 选项你可以来为运行 Gradle 的 JVM 传递系统属性。 *gradle* 命令中的 *-D* 选项和 *java* 命令的 *-D* 选项有着相同的效果。

当然，你也可以使用属性文件来为你的项目对象添加属性设置。你可以在 Gradle 的用户 home 目录或项目目录下添加一个 *gradle.properties* 文件。对于多项目构建来说，你可以在任意子项目目录下放置 *gradle.properties* 文件。这些文件中设置的属性可以通过项目对象被访问到。用户 home 目录中的属性文件会覆盖掉项目目录中的属性文件。

同时，你还可以通过 *-P* 命令行选项直接为你的项目对象添加属性。

Gradle 还可以引用系统属性或者环境变量来设置项目属性。当你没有持续集成服务器的管理员权限，还要必须设置那些明显不可见的属性（典型的是处于安全考虑）时十分有用。在这个时候，你不能使用 *-P* 选项，也不能改变系统级别的配置文件。正确的策略是更改持续集成构建工作的配置，添加那些可与预期模式匹配的环境变量。这些对于系统的普通用户不可见*[1]*。

如果环境变量的命名类似 *ORG_GRADLE_PROJECT_prop=somevalue*，那么 Gradle 将会为项目对象设置一个 *prop* 属性，它的值是 *somevalue*。Gradle 还支持系统属性进行如此设置，只是它支持这样的命名模式 *org.gradle.project.prop*。

你可以在 *grdle.properties* 文件中进行系统属性的设置。如果属性名前缀为 *systemProp*，例如 *systemProp.propName*，这样一个 **属性名*（去掉前缀）*-值** 对就是一个系统属性。在多项目构建中，除了根目录，其他的子项目中的 *systemProp* 属性会被忽略。这意味着，只有根项目的 *gradle.properties* 文件中那些有 *systemProp* 前缀的属性会被检查。

例子，gradle.properties文件：

```
gradlePropertiesProp=gradlePropertiesValue
sysProp=shouldBeOverWrittenBySysProp
envProjectProp=shouldBeOverWrittenByEnvProp
systemProp.system=systemValue
```

build.gradle 文件：

```
task printProps {
    doLast {
        println commandLineProjectProp
        println gradlePropertiesProp
        println systemProjectProp
        println envProjectProp
        println System.properties['system']
    }
}
```

*gradle -q -PcommandLineProjectProp=commandLineProjectPropValue -Dorg.gradle.project.systemProjectProp=systemPropertyValue printProps* 命令输出：

```
> gradle -q -PcommandLineProjectProp=commandLineProjectPropValue -Dorg.gradle.project.systemProjectProp=systemPropertyValue printProps
commandLineProjectPropValue
gradlePropertiesValue
systemPropertyValue
envPropertyValue
systemValue
```
<br/>
## 2.1、项目属性检查
<br/>
在构建脚本中，你可以使用变量名的方式访问项目属性。如果属性不存在，会抛出异常并造成构建失败。假如构建脚本以来一些用户设置的备选属性，例如在 *gradle.properties* 文件中，那么你需要在访问前进行一些存在性检查。你可以是使用 *hasProperty('propertyName')* 方法，该方法会返回 *true* 或 *false*。


<br/>
# 3、代理设置
<br/>

通过标准的 JVM 系统属性可以配置 HTTP 或者 HTTPS 代理（例如，下载依赖包）。你可以直接配置在构建脚本中，例如使用 *System.setProperty('http.proxyHost', 'www.somehost.org')* 设置 HTTP 代理 host。或者，在 *gradle.properties* 文件中。当然，也可以在项目根目录中。

例子，在 gradle.properties 文件中配置 HTTP：

```
systemProp.http.proxyHost=www.somehost.org
systemProp.http.proxyPort=8080
systemProp.http.proxyUser=userid
systemProp.http.proxyPassword=password
systemProp.http.nonProxyHosts=*.nonproxyrepos.com|localhost
```

在 gradle.properties 文件中配置 HTTPS：

```
systemProp.https.proxyHost=www.somehost.org
systemProp.https.proxyPort=8080
systemProp.https.proxyUser=userid
systemProp.https.proxyPassword=password
systemProp.https.nonProxyHosts=*.nonproxyrepos.com|localhost
```
<br/>
## 3.1、NTLM 认证

<br/>

如果代理需要进行 NTLM 认证，你可能不但要提供用户名和密码还要提供认证域名。对于 NTLM 代理你有两种方式提供域名：

- 设置 *http.proxyUser* 系统属性，提供类似 *domain/username* 的值

- 通过 *http.auth.ntlm.domain* 系统属性来提供认证域名


<br/>
------
[1] Jenkins、Teamcity、Bamboo还有其他的 CI 系统提供此类功能









