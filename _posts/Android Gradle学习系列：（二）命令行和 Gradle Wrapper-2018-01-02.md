---
title: Android Gradle学习系列：（二）命令行和 Gradle Wrapper
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - Android Studio
date: 2018-01-2 19:36:21
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

平时的 Android 开发中，通过界面上的 *Run* 按钮来打包和安装 apk，这一些列动作，主要是靠 Gradle 命令来完成的。基本上，我们不太去直接使用命令行来进行打包。但是实际上，我们还是需要了解和学习这一过程。*<u>本文使用的 Gradle 版本是 3.3， 没有特别指明的话，本系列后续篇幅中 Gradle 均为此版本</u>*。

## 2.1、tasks 命令

在 Terminal 中使用该命令，你将会看到这样的输出信息：

```
> gradle tasks
Parallel execution with configuration on demand is an incubating feature.
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'gradle'.
components - Displays the components produced by root project 'gradle'. [incubating]
dependencies - Displays all dependencies declared in root project 'gradle'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gradle'.
dependentComponents - Displays the dependent components of components in root project 'gradle'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'gradle'. [incubating]
projects - Displays the sub-projects of root project 'gradle'.
properties - Displays the properties of root project 'gradle'.
tasks - Displays the tasks runnable from root project 'gradle'.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL

Total time: 0.607 secs
```

这里列出了一些项目可以允许的 task 列表，分为了两组任务（当然，你可以为任务配置分组信息，这里可能会有更多的分组），一是构建配置任务*（*Build Setup tasks*）*，参考 2.2；一是帮助任务*（*Help tasks*）* 这些可以帮助你掌握项目的一些信息，比如依赖、属性、包括的子项目。

## 2.2、init 和 wrapper

假如你想从零开始一个项目，你可以试试

```
$ gradle init
```

它可以帮助你生成一些文件，包括以下文件和目录：

```
+ gradle 目录，包括 wrapper 配置
graldew
gradlew.bat
build.gradle 文件，项目的根 build 文件
settings.gradle 项目的设置文件，常规的就是项目的组织和配置信息
```



使用 wrapper 这个命令，可以帮助你生成 gradle 目录、gradlew 以及 gradlew.bat 文件，这样一来，需要团队协作的项目可以统一 Gradle 的版本（构建环境）。

```
> gradle wrapper
```



## 2.3、多任务执行

Gradle 使用有向无环图来决定一条任务执行的路径，任务之间可以有依赖关系。你可以同时执行多个任务，只需要用空格分割多个任务就OK，并且任务仅被执行一次。例如 build.gradle 文件的内容是这样的：

```
task compile {
    doLast {
        println 'compiling source'
    }
}

task compileTest(dependsOn: compile) {
    doLast {
        println 'compiling unit tests'
    }
}

task test(dependsOn: [compile, compileTest]) {
    doLast {
        println 'running unit tests'
    }
}

task dist(dependsOn: [compile, test]) {
    doLast {
        println 'building the distribution'
    }
}
```

OK， 你可以看得出来，任务之间的依赖性是这样的，如下图所示。

![图2-1](https://docs.gradle.org/current/userguide/img/commandLineTutorialTasks.png)

<p align = "center"> `图2-1` </p>

当执行 gradle dist compile ，执行 dist 和 compile 任务时，在控制台你可以看到如下信息：

```
> gradle dist compile
Parallel execution with configuration on demand is an incubating feature.
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 0.611 secs

```

如果，你去掉 dist 里的参数，即去掉对 compile 和 test 对依赖，在执行该命令，会得到一下的输出信息。这样进一步验证了 Gradle 在执行多任务时候按照任务在命令行里列出的先后顺序执行，当一个任务已经执行过了，在后面列出来了也会被跳过不执行。

```
> gradle dist compile
Parallel execution with configuration on demand is an incubating feature.
:dist
building the distribution
:compile
compiling source

BUILD SUCCESSFUL

Total time: 0.784 secs
```



## 2.4、排除任务的 -x 选项

你可以排除某个任务不被执行，只需要使用 -x 选项并后接任务名。想要排除多个任务，你要多次使用 *-x 任务名* 的方式来进行操作。比如：

```
> gradle dist -x compile -x compileTest
Parallel execution with configuration on demand is an incubating feature.
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL

Total time: 0.606 secs
```



## 2.5 -\-continue 选项 

Gradle 在执行任务失败时会默认的中止构建，这样一来会使整个构建工作完成，但是会把一些可能会失败的问题隐藏起来。为了能够在单次构建的执行中发现更多的问题，你可以使用 *—continue* 选项。

使用该选项，可以执行所有依赖任务没有失败的任务，在构建完成后，失败信息会被报告。

如果任务失败，基于安全的考虑，依赖于它的后续任务将不被执行。例如，在*图2-1*中，*test* 依赖的任务如果执行失败， *test* 任务将停止执行。

## 2.6、任务名缩写 

在执行任务时，你不必写出完整的任务名。你只需要提供足够的信息能够唯一确定该任务就OK了。例如，在上面的例子中，你可以使用 *di* 代替 *dist*。

另外，你还可以对驼峰式的名字中进行缩写。比如，使用 *compTest* 甚至是 *cT* 来替代 *compileTest*。

## 2.7、选择 build  

当使用 *gradle* 时，默认会查找当前目录的 *build.gradle* 文件。你可以使用 *-b* 选项来选择 build 文件。例如：

```
> gradle -q -b subdir/myproject.gradle hello
```

该命令会选择 *subdir* 目录下的 *myproject.gradle* 文件并执行 *hello* 任务。

或者，使用 *-p* 选项来指定项目目录。对于多项目构建使用 *-p* 选项而非 *-b* 选项。

```
> gradle -q -p subdir hello
```

该命令使用 *subdir* 目录下的 *build.gradle* 文件并执行 *hello* 任务。另外，*如果没有该文件会报错*。

## 2.8、强制执行 -\-rerun-tasks 选项

许多任务，特别是 Gradle 自身提供的，都支持增量构建 *（incremental builds）*。此类任务是否需要执行，取决于最近一次执行后输入输出是否有改变。在构建过程中，你可以看到一些被标记了 *UP-TO-DATE* 的任务，它们是不需要再次被执行的。

有时候，你可能需要执行所有的任务，忽略一些 up-to-date 检查。使用 *-\-rerun-tasks* 可以帮助你完成该项工作。

这一步会强制那些有必要执行的任务被执行。这有点类似 *clean* 操作，唯一的不同的在于它不会删除 build 后生成的 output。

## 2.9、 获取 build 信息

该部分，针对2.1中的帮助任务进行一些介绍。

### 2.9.1、项目清单

运行 *gradle projects* 命令会列出项目的所有子项目信息。下面是一个使用 projects 命令获取的信息：

```
> gradle projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'task'
+--- Project ':app' - The shared API for the application
\--- Project ':robin.test'

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :app:tasks

BUILD SUCCESSFUL

Total time: 0.723 secs
```

如果在项目的 *build.gradle* 中指定了描述属性（如下），也会在 Terminal 中显示。

```
description = 'The shared API for the application'
```

### 2.9.2、任务清单 tasks命令 

在开始的时候，我们已经介绍了该命令。此处会展开一点讲一下。例如，我们有下面这个 build.gradle 文件：

````
task clean {
	description = 'Deletes the build directory (build)'
    group = 'build'

}

task libs {
	description = 'Builds the JAR'
    group = 'build'

} 

task dists {
   description = 'Builds the distribution'
    group = 'build'
}

task test {
	description = 'Test task'
    group = 'build'
     doLast {
        println('do during execution phrase')
    }
}
````

执行命令后：

```
> gradle -q tasks
------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build tasks
-----------
clean - Deletes the build directory (build)
dists - Builds the distribution
libs - Builds the JAR
test - Test task

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]
....
```

*Build tasks* 中的任务前三个是 Gradle 自身提供的，而 *test* 是我们定义的，我们通过添加 *group* 分组的方式将它们放在了一起。这些任务叫做可见任务*（visible tasks）*。

使用 *-\-all* 选项，你可以获取更多的任务信息。包括项目下的那些没有进行分组的任务，即隐藏任务*（hidden tasks）* 。

### 2.9.3、任务使用详情

使用 *gradle help --task someTask* 可以获取指定任务或多个任务的信息。

```
> gradle -q help --task libs
Detailed task information for libs

Paths
     :api:libs
     :webapp:libs

Type
     Task (org.gradle.api.Task)

Description
     Builds the JAR

Group
     build
```

这些信息包括完整任务路径、任务类型、可能的命令行选项以及描述信息等。

### 2.9.4、依赖清单

*gradle dependencies* 命令可以获取选中的项目依赖信息，默认是现在根项目*（Root project）*的。你可以通过指定 *项目:dependencies* 的方式来添加子项目。例如：

```
> gradle -q dependencies api:dependencies webapp:dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

No configurations

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

compile
\--- org.codehaus.groovy:groovy-all:2.4.10

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3

------------------------------------------------------------
Project :webapp - The Web application implementation
------------------------------------------------------------

compile
+--- project :api
|    \--- org.codehaus.groovy:groovy-all:2.4.10
\--- commons-io:commons-io:1.2

testCompile
No dependencies
```

由于依赖报告可能会很大，因此通过配置来简化报告，该操作可使用 *—configuration* 选项：

```
> gradle -q api:dependencies --configuration testCompile

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3
```

### 2.9.5、项目buildscript依赖清单

```
> gradle buildEnvironment 
```

### 2.9.6、深入依赖

通过该命令可以深入指定的依赖包，例如：

```
> gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
org.codehaus.groovy:groovy-all:2.4.10
\--- project :api
     \--- compile
```

### 2.9.7、项目属性清单

```
> gradle -q api:properties

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

allprojects: [project ':api']
ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@12345
asDynamicObject: DynamicObject for project ':api'
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle
```

## 2.10、-\-profile选项

*—profile* 选项可以帮助你记录构建过程中的一些时间信息，并会生成一份报告位于 *`build/reports/profile* 目录。

profile 会汇总配置阶段和任务执行的时间和详情。并按时间长短从大到小排序，还将指示任务是否被跳过。

如果项目中有 *buildSrc* 目录，还将在 *buildSrc/build* 目录下还将生产一个 profile 文件。

*PS： profile 文件主要帮助你来进行 build 阶段的优化工作，找出瓶颈，使用合适的方法去解决它。*

## 2.11、空运行 Dry Run

有时候你可能对于命令行中的任务的执行顺序会感兴趣，但是又不想任务被真正执行。这时候，你可以使用 *-m* 选项。例如:

```
> gradle -m clean dists
:clean SKIPPED
:dists SKIPPED

BUILD SUCCESSFUL

Total time: 0.66 secs
```

<br>

# 3、Gradle Wrapper

之前说到 Android 项目基于 Gradle 以及 Plugin 进行构建，不同的项目的 Gradle 环境（版本）不同，同一个项目可能需要在一个 team 中进行开发，需要 Gradle 环境保持一致。因此，这就产生了 Gradle Wrapper。Wrapper 是一个管理 Gradle 版本的工具，这有点类似于 Node 的 NVM，可以安装、卸载 Node 版本，也可以切换 Node 版本。

## 3.1、使用 Wrapper 进行 build

假如项目配置了 Wrapper，你可以在项目根目录下执行下列命令：

```
> ./gradlew
Downloading https://services.gradle.org/distributions/gradle-3.3-bin.zip
..............................................................................................................................................................
Unzipping /User/xxxxxxxx/.gradle/wrapper/dists/gradle-3.3-bin/64bhckfm0iuu9gap9hg3r7ev2/gradle-3.3/bin/gradle

BUILD SUCCESSFUL

Total time: 57.793 secs
```

每个 Wrapper 都会指定一个 Gradle 版本，因此，当你执行命令的时候。首先它会去下载该版本然后去 build。

>IDEs
>
>如果通过 wrapper 把项目导入 IDE 中去，可能会提示你使用 'all' 发布。这能够给 IDE 提供更完整的代码去build 文件。

使用 Wrapper，不仅免去了 Gradle 的下载工作，还保证了版本的一致。这样保证你所有的历史 build 的可靠。命令和 *gradle* 相似。

在 *$USER_HOME/.gradle/wrapper/dists* 目录下，你能够找到这些 *Gradle* 发布。

## 3.2、添加Wrapper

在上一节中，我们可以通过 *wrapper* 来为项目添加 Wrapper。默认的，它使用当前的 Gradle 版本的 'bin' 发布（文件体积最小），当然，你可以通过 *-\-gradle-version* 来指定版本。类似 Android Studio 这类的开发工具使用 'all' 发布。你可以使用 *-\-distribution-type* 来指定发布的类型，使用 *-\-gradle-distribution-url* 来指定发布的下载地址。如果不指定版本或者 url ，则使用默认的配置。

```
> gradle wrapper --gradle-version 2.0
:wrapper

BUILD SUCCESSFUL

Total time: 0.759 secs
```

 或者在 *build.gradle* 中配置一个 Wrapper 任务来指定版本，然后执行该任务：

```
task wrapper(type: Wrapper) {
    gradleVersion = '2.0'
}
-----------------------------------------------
> gradle wrapper
:wrapper

BUILD SUCCESSFUL

Total time: 0.668 secs
```

执行完后，在项目下会添加一些文件：

```
  -gradlew
  -gradlew.bat
  +gradle/wrapper/
    -gradle-wrapper.jar
    -gradle-wrapper.properties
```

之后，你应该使用 *gradlew* 命令 build 项目，使用方式和 *gradle* 一样 。如果你想更换版本，只需要修改 *gradle-wrapper.properties* 文件即可。

## 3.3、配置

默认的，Android 项目在创建时，已经配置了 wrapper。当然，你也可以通过 *gradle wrapper* 命令行来添加 Wrapper。Wrapper 位于项目根目录下的 *gradle/wrapper* 下面，里面有一个 *jar* 包还有一个配置文件 *gradle-wrapper.properties* ：

```
#Sun May 01 11:23:33 CST 2016
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-3.3-all.zip
```

这里面有个五个配置项，但是前面四个是两两组合使用的。*distributionBase + distributionPath* 指定了该版本的 Gradle 在此目录下存放，如果没有就查找有没有下载 *zip* 文件。*zipStoreBase + zipStorePath* 指定了 *zip* 文件存放的位置，如果该文件不存在就去 *distributionUrl* 地址去下载。这几个地址均是可配的，当然一般来说是不需要动的。

***唯一一个需要说明的是：***通常卡在 build 时，可能是由于下载 Gradle 时造成的。你可以去国内的站点去下载 Gradle，然后放在 *dists* 目录或者修改 *distributionUrl* 为 *file\:///your_file_path/gradle-x.x-all.zip*，然后进行 build。

## 3.4、下载认证

这一部分主要针对哪些私有受保护的服务器，可能需要 HTTP Basic 认证。在 *.gradle/gradle.properties* 文件中添加一下配置信息：

```
systemProp.gradle.wrapperUser=username
systemProp.gradle.wrapperPassword=password
```

然后修改 *gradle/wrapper/gradle-wrapper.properties* 文件的 *distributionUrl* 属性，加入认证信息。

```
distributionUrl=https://username:password@somehost/path/to/gradle-distribution.zip
```

更多信息可以查看 [*Section 12.3, “Accessing the web via a proxy”*](https://docs.gradle.org/current/userguide/build_environment.html#sec:accessing_the_web_via_a_proxy)

## 3.5、验证发布文件

你可以下载 *.sha256* 文件对下载的发布进行验证是否被篡改。或者在 *gradle-wrapper.properties* 文件中使用 *distributionSha256Sum* 进行验证：

```
distributionSha256Sum=371cb9fbebbe9880d147f59bab36d61eee122854ef8c9ee1ecf12b82368bcf10
```

## 3.6、Unix 文件权限

Unix 系统的文件权限问题时很头痛的，这会带来很多问题。根据官网的建议，Unix 系统你应该时刻使用 *sh gradlew* 这样的命令。

<br/>

# 4、最后

本篇介绍了关于 Gradle 的一些基本知识，还有命令的一些基础使用，了解了 Wrapper 是什么，能干什么。下一章，我们来谈谈 Build 环境的配置。









