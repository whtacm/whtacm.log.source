---
title: Android Gradle学习系列：（五）Gradle 插件开发及 Android Gradle 插件
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - Android Studio
date: 2018-01-13 19:55:42
---

>前言

在之前，我们提到 Android 项目的构建需要 Android Gradle 插件来完成。每一个 Gradle 插件就是来帮助任务进行构建的，在你的 *build.gradle* 文件中应用你的插件并编写构建 DSL *（domain specific language）*脚本。本章将给你一个直观的插件编写例子，以及 Android Gradle 插件的相关介绍。

<!--- more --->

<br/>

# 1、插件开发
<br/>

插件代码可以放在下面几个地方，或者说，使用一个插件有三种方式：

- Build script，把插件源代码放在构建脚本 *build.gradle* 中。插件会被自动编译，并加入到构建脚本的 classpath 中。缺点是对外部构建脚本*（build script）*不可见，不能复用。
- buildSrc 项目，把插件源代码放在 *rootProjectDir/buildSrc/src/main/java* 目录下。需要编译和测试才可用，本项目构建的构建脚本都可以使用。然而，对于外部构建不可用。
- 独立项目，打包发布成一个 JAR 包的方式。可以被多个构建复用。



本示例中为第二种方式。

<br/>

## 1.1、创建项目

<br/>

创建插件工程并切换到项目目录下：

```
$ mkdir greeting-plugin
$ cd greeting-plugin
```

生成插件代码目录：

```
$ mkdir -p buildSrc/src/main/java/org/example/greeting
```

<br/>

## 1.2、创建插件

<br/>

在刚创建的目录下生成 *GreetingPlugin* 类，该类必须实现 *Plugin* 接口。当插件被工程使用时，Gradle 会生成一个 plugin 类实例并调用实例的 *aplly(T)* 方法。

***buildSrc/src/main/java/org/example/greeting/GreetingPlugin.java***

```
package org.example.greeting;

import org.gradle.api.Plugin;
import org.gradle.api.Project;

public class GreetingPlugin implements Plugin<Project> {
    public void apply(Project project) {
        project.getTasks().create("hello", Greeting.class, (task) -> { 
            task.setMessage("Hello");
            task.setRecipient("World");                                
        });
    }
}
```

其中，生成了一个命名为 hello 的 Greeting 类型的新任务实例，并为之设置了默认值。

这就是一个真实的插件代码，并且 Project 对象是全部 Gradle API 的访问入口，通过它可以完成在构建脚本所做的同样工作。在本例中，在目标项目中创建了一个 hello 任务。

接下来，为 GreetingPlugin 使用的任务类型生成 java 类。依旧在改包路径下添加 Greeting 类。

***buildSrc/src/main/java/org/example/greeting/Greeting.java*** 

```
package org.example.greeting;

import org.gradle.api.DefaultTask;
import org.gradle.api.tasks.TaskAction;

public class Greeting extends DefaultTask {
    private String message;
    private String recipient;

    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }

    public String getRecipient() { return recipient; }
    public void setRecipient(String recipient) { this.recipient = recipient; }

    @TaskAction
    void sayGreeting() {
        System.out.printf("%s, %s!\n", getMessage(), getRecipient()); 
    }
}
```

该类中有一些属性以及 getter/setter 类。在 sayGreeting 方法中指定了当任务运行后输出的配置消息。

<br/>

## 1.3、使用插件

<br/>

有了上面的工作，现在就实现了一个简单的插件了。下面，你可以使用插件（当然，比较low一点，只能打印一个输出语句）。在 build.gradle 文件中使用 apply 关键字在项目中使用插件：

```
apply plugin: org.example.greeting.GreetingPlugin
```

这样，当前的 Project 实例将使用改插件，并在构建中添加一个 hello 任务。

使用 *gradle hello* 命令来执行 hello 任务来验证插件的正确性：

```
$ gradle hello
:buildSrc:compileJava
:buildSrc:compileGroovy UP-TO-DATE
:buildSrc:processResources UP-TO-DATE
:buildSrc:classes
:buildSrc:jar
:buildSrc:assemble
:buildSrc:compileTestJava UP-TO-DATE
:buildSrc:compileTestGroovy UP-TO-DATE
:buildSrc:processTestResources UP-TO-DATE
:buildSrc:testClasses UP-TO-DATE
:buildSrc:test UP-TO-DATE
:buildSrc:check UP-TO-DATE
:buildSrc:build
:hello
Hello, World!
```

控制台的输出表面 buildSrc 中的文件被当作了 Java 项目现进行了一个编译过程。然后在主构建中进行了执行。

现在构建中 hello 任务使用的是默认值，你也可以在 build 脚本中进行一个配置：

```
hello { 
    message = "Hi"
    recipient = "Gradle"
}
```

再次执行 hello 任务，并使用 -q 选项去掉一些控制台输出：

```
$ gradle -q hello
Hi, Gradle!
```

现在，自定义插件功能运行 OK。接下来，我们为插件声明一个ID，同时，当你发布插件时这样做会有帮助。

<br/>

## 1.4、声明插件ID

<br/>

通常情况下，我们都是通过 ID 来使用插件，这样比较容易记忆。

首先，生成属性文件：

***buildSrc/src/main/resources/META-INF/gradle-plugins/org.example.greeting.properties***

```
implementation-class=org.example.greeting.GreetingPlugin
```

Gradle 会根据改文件来确定 Plugin 接口的实现类。改属性文件名去掉 *.properties* 就是插件的 ID。

然后，在 build 脚本中，对下面的内容进行替换：

```
apply plugin: org.example.greeting.GreetingPlugin
```

使用插件 ID：

```
apply plugin : 'org.example.greeting'
```

<br/>

## 1.5、插件配置

<br/>

多数插件需要从构建脚本中获取一些配置。其中一个方式就是使用 extension （扩展）对象。Gradle Project 有一个相关的 [`ExtensionContainer`](https://docs.gradle.org/current/javadoc/org/gradle/api/plugins/ExtensionContainer.html) 对象，它存放所有的配置及属性。你可以在其中添加扩展来为插件提供配置。一个扩展对象其实就是一个 Java Bean。

配置可以提供一些丰富的属性，在 Android 开发中，这很常见：

```
android {
	compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion
    buildTypes {
       productFlavors{
			...
    	}
    }
   
}
```

Android Gradle Plugin 在 build 脚本中主要的就是这个 android 扩展配置。插件通过该扩展来进行构建任务的相关配置。

这里，你可以参考：

- 自定义插件编写（*官方*） [Writing Custom Plugins](https://docs.gradle.org/current/userguide/custom_plugins.html) ，该指南是基于 Groovy 语言编写的。
- [Gradle插件开发](https://www.jianshu.com/p/3c59eded8155) ，该份文章你会对自定义插件开发有一个更好的概念上的了解，尤其是对 task 的渐进式构建方面，扩展的编写。
- [A simple Gradle Plugin development example](https://github.com/whtacm/GradlePlugin) ，一个简单的插件编写示例，基于 Java 语言实现的。

<br/>

## 1.6、进一步阅读

<br/>

- 使用Java Gradle Plugin 开发插件简化插件开发 [Simplifying plugin development with the Java Gradle Plugin Development Plugin](https://docs.gradle.org/4.5/userguide/java_gradle_plugin.html)
- 发布插件到 Gradle Plugin Portal [Publishing plugins to the Gradle Plugin Portal](https://guides.gradle.org/publishing-plugins-to-gradle-plugin-portal/)
- 使用扩展进行领域建模 [Modeling your domain with extensions](https://docs.gradle.org/4.5/userguide/custom_plugins.html#sec:getting_input_from_the_build)
- 插件测试 [Testing plugins](https://docs.gradle.org/4.5/userguide/test_kit.html)
- 为新任务类型添加持续构建支持 [Adding incremental build support to new task types](https://docs.gradle.org/4.5/userguide/more_about_tasks.html#sec:up_to_date_checks)


<br/>

# 2、Android Gradle Plugin 介绍

<br/>

刚接触 Android Studio 的时候，一直不太了解 build.gradle 里的那些 DSL，诸如 *buildscript、allprojects、android* 太多的标签。当然，你可以通过这份官方文档来学习 [Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/index.html)  。后来，直到我看了 Android Gradle Plugin的代码之后我才对这些东西有了更清晰的认识。因此，为什么要了解 Android Gradle Plugin，其实是一个 ***知其所以然*** 的过程。

<br/>

## 2.1、从 Project 开始

<br/>





<br/>

## 2.2、BasePlugin及其子类

<br/>







<br/>

## 2.1、Base

<br/>