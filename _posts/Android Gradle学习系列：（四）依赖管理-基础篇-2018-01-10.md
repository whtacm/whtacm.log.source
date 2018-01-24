---
title: Android Gradle学习系列：（四）依赖管理-基础篇
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - Android Studio
date: 2018-01-10 12:35:02
---

>前言

Gradle 的核心就是依赖管理。在 Gradle 中，我们熟知的是第三方依赖包。除此之外，任务、项目都可以是相互依赖的。本篇将对 Gradle 的依赖管理进行介绍。
<!--- more --->
<br/>

# 1、什么是依赖管理
<br/>

简单来说，两个方面。首先，知道哪些东西是构建需要的，并且能够找到这些东西，这些被称之为依赖。其次，构建和上传项目的输出，这些产出的东西称之为 *publications*。

多数项目都需要一些辅助，比如 JDBC 驱动来链接数据库。这些文件形成了项目的依赖，并且依赖也可能有依赖，依赖的依赖称之为传递依赖*（transitive dependencies）*。

<br/>

# 2、声明依赖

<br/>

依赖的声明十分简单，例如在构建脚本中声明依赖：

```
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

这里，声明了 *Hibernate core 3.6.7.Final* 来编译项目的源码。*Hibernate core* 和它的依赖会在运行时被需要。还有任何 junit  4.0 及以上的版本来在项目的测试中参与编译。同时，该声明告诉 Gradle 在 *Maven central* 库中下载需要的依赖包。

<br/>

# 3、依赖配置

<br/>

一个配置就是指定的依赖和构件的集合。配置有三种用途：

- 声明依赖，插件使用配置来声明需要的子项目或外部构件，在任务执行时使用
- 解析依赖，插件使用该配置信息来寻找（下载）
- 暴露构件，定义供其他项目使用的构件

例如，对于 [Java 库插件](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph) 有下面几个标准的配置定义：

- implementation，声明了项目编译的依赖，但不是项目暴露的 api
- runtimeClasspath，运行时类路径
- apiElements






<br/>

# 4、外部依赖

<br/>

声明的依赖有很多种，其中一种是外部依赖 *（external dependency）*。添加外部依赖，可以在 *dependencis* 中添加：

```
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
}
```

外部依赖的标识使用 *group*、*name* 和 *version* 属性。也可以采取 *group:name:version* 简写的方式。

<br/>

# 5、仓库

<br/>

Gradle 会在仓库中寻找外部依赖。仓库就一个文件集合，通过  *group*、*name* 和 *version* 进行组织。

默认的，Gradle 不会定义任何仓库。在使用外部依赖前，你必须定义至少一个仓库。

例如，使用 *Maven central* 仓库：

```
repositories {
    mavenCentral()
}
```

或者，使用 *JCenter* 仓库：

```
repositories {
   jcenter()
}
```

或者，使用 *remote Maven* 仓库：

```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

或者，使用 *remote Ivy* 目录：

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

或者，使用 *本地 Ivy* 目录：

```
repositories {
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```

当然，你可以同时使用多个仓库。Gradle 会在仓库中依次寻找依赖，直到找到依赖包。

<br/>

# 6、发布构件

<br/>

依赖配置还可以发布文件，这称之为 *发布构件（publication artifacts）* 或简称为 *构件*。

你只需要告诉 Gradle 构件发布的位置，额外的工作会由插件完成。你只需要为 *uploadArchives* 任务添加一个仓库。

例如，发布到 *Ivy* 仓库：

```
uploadArchives {
    repositories {
        ivy {
            credentials {
                username "username"
                password "pw"
            }
            url "http://repo.mycompany.com"
        }
    }
}
```

当运行 *gradle uploadArchives* ，Gradle 会构建并上传你的 Jar 包。同时还会生成并上传 ivy.xml 文件。

也可以上传至 Maven 仓库，语法稍有不同。需要先引入 Maven 插件。Gradle 会生成并上传 pom.xml 文件。

```
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```

<br/>

# 进一步阅读

-----

- [Chapter 25, *Dependency Management*](https://docs.gradle.org/current/userguide/dependency_management.html)

- [Chapter 32, *Publishing artifacts*](https://docs.gradle.org/current/userguide/artifact_management.html)

- [`Project.configurations{}`](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:configurations(groovy.lang.Closure))

- [`Project.repositories{}`](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:repositories(groovy.lang.Closure))

- [`Project.dependencies{}`](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:dependencies(groovy.lang.Closure))

  ​





