---
title: Android Gradle学习系列：（五）Gradle DSL 学习
tags:
  - Android
  - Gradle
categories:
  - Android
  - Gradle
  - DSL
 
date: 2018-01-20 8:34:23
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
- [Gradle插件开发](https://www.jianshu.com/p/3c59eded8155) ，该份文章你会对自定义插件开发有一个更好的概念上的了解，尤其是对 task 的增量构建方面，扩展的编写。
- [A simple Gradle Plugin development example](https://github.com/whtacm/GradlePlugin) ，本人在 Github 上的一个简单的插件编写示例，基于 Java 语言实现的。

<br/>

## 1.6、发布插件

<br/>

开发的插件可以发布到 Gralde Portal 上，和其他插件一样共享给其他人使用。

本人把前面那个插件工程从 buildSrc 中抽离了出来，建立了一个单独的插件工程，里面有上传的配置 buidl.gradle 文件，可以供你参考。

本人插件的源码及完整发布配置示例 [https://github.com/whtacm/GradlePluginSrc](https://github.com/whtacm/GradlePluginSrc)

在这之前，你需要到 [registration page](https://plugins.gradle.org/user/register?_ga=2.131428367.1810027140.1519609203-1754654511.1504769273) 上去注册账号并申请一个 API Key，然后配置到你的 ～./gradle/gradle.properties 文件中去。

然后编写自己的插件代码并配置相关的信息，执行:

```
$ gradle publishPlugins
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar
:publishPluginJar
:javadoc
:publishPluginJavaDocsJar
:publishPlugins
Publishing plugin org.robin.example.greeting version 1.0.2
Publishing artifact build/libs/greeting-plugin-src-1.0.2.jar
Publishing artifact build/libs/greeting-plugin-src-1.0.2-sources.jar
Publishing artifact build/libs/greeting-plugin-src-1.0.2-javadoc.jar
Publishing artifact build/publish-generated-resources/pom.xml
Activating plugin org.robin.example.greeting version 1.0.2

BUILD SUCCESSFUL

Total time: 10.851 secs
```

可以看到发布成功了，到 https://plugins.gradle.org/plugin/org.robin.example.greeting 页面上可以确认该插件上传成功并且版本一致。最后，在你的其他工程中使用该插件：

```
buildscript {
  repositories {
    maven {
      url "https://plugins.gradle.org/m2/"
    }
  }
  dependencies {
    classpath "gradle.plugin.org.robin.example.greeting:greeting-plugin-src:1.0.2"
  }
}

apply plugin: "org.robin.example.greeting"

write {
    clearAll true
    targetDir file('./target')
    generateAll true
}

bookManager {

}
```

执行 

```
$ gradle write -q
GreetingPlugin constructor-->> 
...
WriteBookTask::write-->>path: /Users/robin/android/github/Meizhi/target
```

控制台的输出表面插件运行正确，在该目录下生成了相关内容。

<br/>

## 1.7、进一步阅读

<br/>

- 使用Java Gradle Plugin 开发插件简化插件开发 [Simplifying plugin development with the Java Gradle Plugin Development Plugin](https://docs.gradle.org/4.5/userguide/java_gradle_plugin.html)
- 发布插件到 Gradle Plugin Portal [Publishing plugins to the Gradle Plugin Portal](https://guides.gradle.org/publishing-plugins-to-gradle-plugin-portal/)
- 使用扩展进行领域建模 [Modeling your domain with extensions](https://docs.gradle.org/4.5/userguide/custom_plugins.html#sec:getting_input_from_the_build)
- 插件测试 [Testing plugins](https://docs.gradle.org/4.5/userguide/test_kit.html)
- 为新任务类型添加持续构建支持 [Adding incremental build support to new task types](https://docs.gradle.org/4.5/userguide/more_about_tasks.html#sec:up_to_date_checks)


<br/>

# 2、Android Gradle Plugin

<br/>

刚接触 Android Studio 的时候，一直不太了解 build.gradle 里的那些 DSL，诸如 *buildscript、allprojects、android* 太多的标签。当然，你可以通过这份官方文档来学习 [Android Plugin DSL Reference](http://google.github.io/android-gradle-dsl/current/index.html)  。后来，直到我看了 Android Gradle Plugin的代码之后我才对这些东西有了更清晰的认识。因此，为什么要了解 Android Gradle Plugin，其实是一个 ***知其所以然*** 的过程。

<br/>

## 2.1、从 Project 开始

<br/>

我们知道，Android Stutio 中一个工程有多个模块，工程目录以及模块目录下都有一个 *build.gradle* 文件。工程目录下的构建脚本：

```
buildscript {
    repositories {
        mavenCentral()
        ……
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.1'
    }
}
```

其中，依赖里的 *classpath 'com.android.tools.build:gradle:2.3.1'* 就是使用的 Android Gradle Plugin 版本。在构建时候会下载该插件版本进行构建工作。

而 *buildscript 、dependencies*  这些 DSL 都由 Gradle 来实现，其中 buildscript 可以定位到 org.gradle.api.Project 的 void buildscript(Closure configureClosure) 方法。该脚本主要是配置 project 的类路径，并由 org.gradle.api.initialization.dsl.ScriptHandler 执行。

此外，通过查看 Project 类，你还可以看到很多的方法，比如： void allprojects(Closure configureClosure)、Map<String, ?> getProperties()。

在前面的内容中，我们看到，要实现一个插件，需要实现 Plugin 接口的 apply 方法。当你使用插件时候，apply 被调用并传递一个 Project 类的对象。通过该对象就可以使用上面的这些方法，通过 getProperties() 方法能够获取很多的相关属性和对象。这里有很多，包括配置在 *～/.gradle/gradle/properties* 中的属性信息。

<br/>

## 2.2、BasePlugin及其子类

<br/>

一般的在我们的 app 模块的 build.gradle 文件中开头需要这么写：

```
apply plugin: 'com.android.application'
```

从前面的内容，我们知道该模块使用了 ID 为 *'com.android.application'* 的插件类。那么，它的具体实现类是哪个一个呢？我们可以在 build.gradle 文件中添加：

```
dependencies {
    provided 'com.android.tools.build:gradle:2.3.1'
}
```

然后 sync 一下，就可以在 External Libraries 下面找到该包。该包目录：

```
com.android.tools.build:gradle:2.3.1@jar
	|- gradle-2.3.1.jar
		+ com.android.build.gradle
		|- META-INF
			|- gradle-plugins
				android.properties
				com.android.application.properties
				com.android.library.properties
				....
			MANIFEST.MF
		NOTICE
```

OK，根据前面的内容，显而易见的打开 com.android.application.properties 文件就知道了该实现类是 ：

```
implementation-class=com.android.build.gradle.AppPlugin
```

那么，library 模块插件实现类，你也知道了：

```
apply plugin: 'com.android.library'

implementation-class=com.android.build.gradle.LibraryPlugin
```



AppPlugin 类及 LibraryPlugin 类都继承了 *BasePlugin* 这个抽象类，并且都实现了 Plugin 接口。主要的工作是在 *BasePlugin* 中完成的，那么我们来看看这个类的主要工作。



首先，我们来看入口 apply(Project) 方法：

```
protected void apply(@NonNull Project project) {
    checkPluginVersion();// 检查插件的版本，Gradle 和 插件的版本会有一个对应的关系

    this.project = project;
    ExecutionConfigurationUtil.setThreadPoolSize(project);
    // 做一些检查的工作
    checkPathForErrors();
    checkModulesForErrors();

    ProfilerInitializer.init(project);
    threadRecorder = ThreadRecorder.get();

	// ProcessProfileWriter 记录构建的profile信息，这里会对它进行一些设置
    ProcessProfileWriter.getProject(project.getPath())
            .setAndroidPluginVersion(Version.ANDROID_GRADLE_PLUGIN_VERSION)
            .setAndroidPlugin(getAnalyticsPluginType())
            .setPluginGeneration(GradleBuildProject.PluginGeneration.FIRST);

	// 记录配置工程的时间
    threadRecorder.record(
            ExecutionType.BASE_PLUGIN_PROJECT_CONFIGURE,
            project.getPath(),
            null,
            this::configureProject);
	// 记录配置扩展的时间
    threadRecorder.record(
            ExecutionType.BASE_PLUGIN_PROJECT_BASE_EXTENSION_CREATION,
            project.getPath(),
            null,
            this::configureExtension);
	// 记录创建任务的时间
    threadRecorder.record(
            ExecutionType.BASE_PLUGIN_PROJECT_TASKS_CREATION,
            project.getPath(),
            null,
            this::createTasks);

    // Apply additional plugins
    for (String plugin : AndroidGradleOptions.getAdditionalPlugins(project)) {
        project.apply(ImmutableMap.of("plugin", plugin));
    }
}
```

其中，比较重要的是调用了configureProject、 configureExtension 以及 createTasks 这三个方法。从字面上，比较容易理解它们可能完成的工作。

我们先看 configureProject 方法：

```
private void configureProject() {
    extraModelInfo = new ExtraModelInfo(project);
    checkGradleVersion(); // 检查 Gradle 版本
    AndroidGradleOptions.validate(project);
    sdkHandler = new SdkHandler(project, getLogger());

    project.afterEvaluate(p -> {
        // TODO: Read flag from extension.
        if (!p.getGradle().getStartParameter().isOffline()
                && AndroidGradleOptions.getUseSdkDownload(p)) { // 依赖的下载
            SdkLibData sdkLibData =
                    SdkLibData.download(getDownloader(), getSettingsController());
            dependencyManager.setSdkLibData(sdkLibData);
            sdkHandler.setSdkLibData(sdkLibData);
        }
    });

    androidBuilder = new AndroidBuilder(
            project == project.getRootProject() ? project.getName() : project.getPath(),
            creator,
            new GradleProcessExecutor(project),
            new GradleJavaProcessExecutor(project),
            extraModelInfo,
            getLogger(),
            isVerbose());
    dataBindingBuilder = new DataBindingBuilder();
    dataBindingBuilder.setPrintMachineReadableOutput(
            extraModelInfo.getErrorFormatMode() ==
                    ExtraModelInfo.ErrorFormatMode.MACHINE_PARSABLE);

    // Apply the Java and Jacoco plugins.
    project.getPlugins().apply(JavaBasePlugin.class);
    project.getPlugins().apply(JacocoPlugin.class);

	// 获取 assemble 任务为之添加描述信息
    project.getTasks()
            .getByName("assemble")
            .setDescription(
                    "Assembles all variants of all applications and secondary packages.");
	
	// 为构建过程添加监听，这里主要看最后一个，即构建完成后的处理
    // call back on execution. This is called after the whole build is done (not
    // after the current project is done).
    // This is will be called for each (android) projects though, so this should support
    // being called 2+ times.
    project.getGradle()
            .addBuildListener(
                    new BuildListener() {
                        private final LibraryCache libraryCache = LibraryCache.getCache();

                        @Override
                        public void buildStarted(Gradle gradle) {}

                        @Override
                        public void settingsEvaluated(Settings settings) {}

                        @Override
                        public void projectsLoaded(Gradle gradle) {}

                        @Override
                        public void projectsEvaluated(Gradle gradle) {}

                        @Override
                        public void buildFinished(BuildResult buildResult) {
                            ExecutorSingleton.shutdown();
                            sdkHandler.unload();
                            threadRecorder.record(
                                    ExecutionType.BASE_PLUGIN_BUILD_FINISHED,
                                    project.getPath(),
                                    null,
                                    () -> {
                                    	// 对缓存进行清理
                                        PreDexCache.getCache()
                                                .clear(
                                                        FileUtils.join(
                                                                project.getRootProject()
                                                                        .getBuildDir(),
                                                                FD_INTERMEDIATES,
                                                                "dex-cache",
                                                                "cache.xml"),
                                                        getLogger());
                                        JackConversionCache.getCache()
                                                .clear(
                                                        FileUtils.join(
                                                                project.getRootProject()
                                                                        .getBuildDir(),
                                                                FD_INTERMEDIATES,
                                                                "jack-cache",
                                                                "cache.xml"),
                                                        getLogger());
                                        libraryCache.unload();
                                        Main.clearInternTables();
                                    });
                        }
                    });

	// 为任务添加监听
    project.getGradle()
            .getTaskGraph()
            .addTaskExecutionGraphListener(
                    taskGraph -> {
                        for (Task task : taskGraph.getAllTasks()) {
                            if (task instanceof TransformTask) {
                                Transform transform = ((TransformTask) task).getTransform();
                                // 这里主要是来在任务执行中复用缓存
                                if (transform instanceof DexTransform) {
                                    PreDexCache.getCache()
                                            .load(
                                                    FileUtils.join(
                                                            project.getRootProject()
                                                                    .getBuildDir(),
                                                            FD_INTERMEDIATES,
                                                            "dex-cache",
                                                            "cache.xml"));
                                    break;
                                } else if (transform instanceof JackPreDexTransform) {
                                    JackConversionCache.getCache()
                                            .load(
                                                    FileUtils.join(
                                                            project.getRootProject()
                                                                    .getBuildDir(),
                                                            FD_INTERMEDIATES,
                                                            "jack-cache",
                                                            "cache.xml"));
                                    break;
                                }
                            }
                        }
                    });
}
```

configureProject 的主要工作是下载依赖、对任务之间的缓存处理以及构建完成后的记录动作。值得注意的是这个 cache.xml 文件的路径是在工程根目录下的 ***build/intermediates/dex-cache*** 下面。除此之外，工程目录的 ***build*** 目录下存放了很多文件，比如 混淆文件、profile 报告。同时，在每个模块下面的 ***build*** 中也有大量的文件，都是在构建过程中生成的缓存等。

configureExtension 的代码：

```
private void configureExtension() {
	// 这三行类似，是分别向 project 中添加 BuildType、 ProductFlavor 
	// 以及 SigningConfig 类型的命名对象容器
    final NamedDomainObjectContainer<BuildType> buildTypeContainer =
            project.container(
                    BuildType.class,
                    new BuildTypeFactory(instantiator, project, project.getLogger()));
    final NamedDomainObjectContainer<ProductFlavor> productFlavorContainer =
            project.container(
                    ProductFlavor.class,
                    new ProductFlavorFactory(
                            instantiator, project, project.getLogger(), extraModelInfo));
    final NamedDomainObjectContainer<SigningConfig> signingConfigContainer =
            project.container(SigningConfig.class, new SigningConfigFactory(instantiator));

	// 比较重要，创建扩展
    extension =
            createExtension(
                    project,
                    instantiator,
                    androidBuilder,
                    sdkHandler,
                    buildTypeContainer,
                    productFlavorContainer,
                    signingConfigContainer,
                    extraModelInfo);

    // create the default mapping configuration.
    project.getConfigurations()
            .create("default" + VariantDependencies.CONFIGURATION_MAPPING)
            .setDescription("Configuration for default mapping artifacts.");
    project.getConfigurations().create("default" + VariantDependencies.CONFIGURATION_METADATA)
            .setDescription("Metadata for the produced APKs.");

	// 创建依赖管理器，主要管理配置中的依赖
    dependencyManager = new DependencyManager(
            project,
            extraModelInfo,
            sdkHandler);
	// 创建 ndkhandler，处理ndk相关信息
    ndkHandler = new NdkHandler(
            project.getRootDir(),
            null, /* compileSkdVersion, this will be set in afterEvaluate */
            "gcc",
            "" /*toolchainVersion*/);
            
	// 创建任务管理器，对于 app 模块，它调用 AppPlugin 的 createTaskManager 方法来创建
    // ApplicationTaskManager 对象，它里面有一个 createTasksForVariantData 方法来为变体创建任务
    taskManager =
            createTaskManager(
                    project,
                    androidBuilder,
                    dataBindingBuilder,
                    extension,
                    sdkHandler,
                    ndkHandler,
                    dependencyManager,
                    registry,
                    threadRecorder);

	// 创建变体工厂，同任务管理器一样，对于 app 模块，它调用 AppPlugin 的 createVariantFactory 方法来
	// 创建 ApplicationVariantFactory 对象，它里面有一个 createVariantData 方法来为创建变体
    variantFactory = createVariantFactory(instantiator, androidBuilder, extension);
	
	// 创建变体管理器来创建和管理变体
    variantManager =
            new VariantManager(
                    project,
                    androidBuilder,
                    extension,
                    variantFactory,
                    taskManager,
                    instantiator,
                    threadRecorder);

    // Register a builder for the custom tooling model
    ModelBuilder modelBuilder = new ModelBuilder(
            androidBuilder,
            variantManager,
            taskManager,
            extension,
            extraModelInfo,
            ndkHandler,
            new NativeLibraryFactoryImpl(ndkHandler),
            getProjectType(),
            AndroidProject.GENERATION_ORIGINAL);
    registry.register(modelBuilder);

    // Register a builder for the native tooling model
    NativeModelBuilder nativeModelBuilder = new NativeModelBuilder(variantManager);
    registry.register(nativeModelBuilder);

	// 后面这几个 whenObjectAdded 方法类似，当向容器里添加对象时候添加回调
    // map the whenObjectAdded callbacks on the containers.
    signingConfigContainer.whenObjectAdded(variantManager::addSigningConfig);

	// 同上，添加回调
    buildTypeContainer.whenObjectAdded(
            buildType -> {
                SigningConfig signingConfig =
                        signingConfigContainer.findByName(BuilderConstants.DEBUG);
                buildType.init(signingConfig);
                variantManager.addBuildType(buildType);
            });

    productFlavorContainer.whenObjectAdded(variantManager::addProductFlavor);

    // map whenObjectRemoved on the containers to throw an exception.
    signingConfigContainer.whenObjectRemoved(
            new UnsupportedAction("Removing signingConfigs is not supported."));
    buildTypeContainer.whenObjectRemoved(
            new UnsupportedAction("Removing build types is not supported."));
    productFlavorContainer.whenObjectRemoved(
            new UnsupportedAction("Removing product flavors is not supported."));

	// 生成默认值，对于 app 模块，该方法调用 ApplicationVariantFactory 的 createDefaultComponents
	// 方法，查看该方法代码只有三行，分别为signingConfigContainer添加一个名为 "debug" 的对象以及为
	// buildTypeContainer添加了名为 "debug" 和 "release" 的对象
    // create default Objects, signingConfig first as its used by the BuildTypes.
    variantFactory.createDefaultComponents(
            buildTypeContainer, productFlavorContainer, signingConfigContainer);
}
```

该方法，我们在本篇主要关心 createExtension 方法。在 app 模块中，该方法由 AppPlugin 实现：

```
@NonNull
@Override
protected BaseExtension createExtension(
        @NonNull Project project,
        @NonNull Instantiator instantiator,
        @NonNull AndroidBuilder androidBuilder,
        @NonNull SdkHandler sdkHandler,
        @NonNull NamedDomainObjectContainer<BuildType> buildTypeContainer,
        @NonNull NamedDomainObjectContainer<ProductFlavor> productFlavorContainer,
        @NonNull NamedDomainObjectContainer<SigningConfig> signingConfigContainer,
        @NonNull ExtraModelInfo extraModelInfo) {
    return project.getExtensions()
            .create(
                    "android",
                    AppExtension.class,
                    project,
                    instantiator,
                    androidBuilder,
                    sdkHandler,
                    buildTypeContainer,
                    productFlavorContainer,
                    signingConfigContainer,
                    extraModelInfo);
}
```



OK，到这里你也许意识到了，在 app 模块下的 build.gradle 文件中的 android{} 扩展名其实是在这里给定了。通过 project.getExtensions().create(…) 可以向容器中添加一个名为 ***android*** 的扩展，扩展类型为 AppExtension 。该类的传入参数包括 buildTypeContainer、productFlavorContainer、signingConfigContainer，它们中包括了构建类型、产品风味以及签名的配置等等。



最后，我们来看 createTasks 方法：

```
private void createTasks() {
    threadRecorder.record(
            ExecutionType.TASK_MANAGER_CREATE_TASKS,
            project.getPath(),
            null,
            () ->
                    taskManager.createTasksBeforeEvaluate(
                            new TaskContainerAdaptor(project.getTasks())));

    project.afterEvaluate(
            project ->
                    threadRecorder.record(
                            ExecutionType.BASE_PLUGIN_CREATE_ANDROID_TASKS,
                            project.getPath(),
                            null,
                            () -> createAndroidTasks(false)));
}
```

该方法中，我们重点看它调用的 createAndroidTasks 方法，它又调用了 VariantManager 的 createAndroidTasks ，该方法又调用了 createTasksForVariantData 方法。OK，这里该方法的实现类，对于 app 模块而言是 ApplicationTaskManager 类。

最后总结一下，configureProject、 configureExtension 以及 createTasks 这三个方法相互配合完成了工程的初始化、扩展配置的信息以及任务的创建工作。接下来，我们来了解下扩展相关的内容。

<br/>

## 2.3、BaseExtension及其子类

<br/>

从前面的介绍中，我们了解到 buidl.gradle 中的 android{} 对应于 AppExtension 类。该类是 ***BaseExtension*** 抽象类的子类，此外， LibraryExtension、TestExtension 也是 ***BaseExtension*** 的子类，它们对应于插件 com.android.library 、com.android.test。

这是一个 app 模块的 buidl.gradle 文件：

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion
    
    defaultConfig {
      ...
    }
    
    signingConfigs {
      ...
    }
    
    buildTypes {
      debug {
        ...
      }
      
      productFlavors {
        flavor1 {
          ...
        }
        flavor2 {
          ...
        }
      }
    }
    
    lintOptions {
      ...
    }
    ...
}
```

它对应于 AppExtension 类，这里我们主要看 ***BaseExtension*** 类，这里几乎可以找到一切的实现。首先，我们来看构造函数：

```
BaseExtension(
            @NonNull final Project project,
            @NonNull Instantiator instantiator,
            @NonNull AndroidBuilder androidBuilder,
            @NonNull SdkHandler sdkHandler,
            @NonNull NamedDomainObjectContainer<BuildType> buildTypes,
            @NonNull NamedDomainObjectContainer<ProductFlavor> productFlavors,
            @NonNull NamedDomainObjectContainer<SigningConfig> signingConfigs,
            @NonNull ExtraModelInfo extraModelInfo,
            final boolean publishPackage) {
        this.androidBuilder = androidBuilder;
        this.sdkHandler = sdkHandler;
        this.buildTypes = buildTypes;
        //noinspection unchecked
        this.productFlavors = (NamedDomainObjectContainer) productFlavors;
        this.signingConfigs = signingConfigs;
        this.extraModelInfo = extraModelInfo;
        this.project = project;

        logger = Logging.getLogger(this.getClass());

		// 下面的都类似，使用 instantiator.newInstance 方法来生成实例，它们都有 getter/setter 方法，
		// 会根据构建脚本 android{} 相应的内容被自动填充
        defaultConfig = instantiator.newInstance(ProductFlavor.class, BuilderConstants.MAIN,
                project, instantiator, project.getLogger(), extraModelInfo);

        aaptOptions = instantiator.newInstance(AaptOptions.class);
        dexOptions = instantiator.newInstance(DexOptions.class, extraModelInfo);
        lintOptions = instantiator.newInstance(LintOptions.class);
        externalNativeBuild = instantiator.newInstance(
                ExternalNativeBuild.class, instantiator, project);
        testOptions = instantiator.newInstance(TestOptions.class);
        compileOptions = instantiator.newInstance(CompileOptions.class);
        packagingOptions = instantiator.newInstance(PackagingOptions.class);
        jacoco = instantiator.newInstance(JacocoOptions.class);
        adbOptions = instantiator.newInstance(AdbOptions.class);
        splits = instantiator.newInstance(Splits.class, instantiator);
        dataBinding = instantiator.newInstance(DataBindingOptions.class);

		// 向 project 中添加 AndroidSourceSet 类型的容器
        sourceSetsContainer =
                project.container(
                        AndroidSourceSet.class,
                        new AndroidSourceSetFactory(instantiator, project, publishPackage));
		// 添加回调
        sourceSetsContainer.whenObjectAdded(
                new Action<AndroidSourceSet>() {
                    @Override
                    public void execute(AndroidSourceSet sourceSet) {
                        ConfigurationContainer configurations = project.getConfigurations();

                        createConfiguration(
                                configurations,
                                sourceSet.getCompileConfigurationName(),
                                "Classpath for compiling the " + sourceSet.getName() + " sources.");

                        String packageConfigDescription;
                        if (publishPackage) {
                            packageConfigDescription =
                                    "Classpath only used when publishing '"
                                            + sourceSet.getName()
                                            + "'.";
                        } else {
                            packageConfigDescription =
                                    "Classpath packaged with the compiled '"
                                            + sourceSet.getName()
                                            + "' classes.";
                        }
                        createConfiguration(
                                configurations,
                                sourceSet.getPackageConfigurationName(),
                                packageConfigDescription);

                        createConfiguration(
                                configurations,
                                sourceSet.getProvidedConfigurationName(),
                                "Classpath for only compiling the "
                                        + sourceSet.getName()
                                        + " sources.");

                        createConfiguration(
                                configurations,
                                sourceSet.getWearAppConfigurationName(),
                                "Link to a wear app to embed for object '"
                                        + sourceSet.getName()
                                        + "'.");

                        createConfiguration(
                                configurations,
                                sourceSet.getAnnotationProcessorConfigurationName(),
                                "Classpath for the annotation processor for '"
                                        + sourceSet.getName()
                                        + "'.");

                        createConfiguration(
                                configurations,
                                sourceSet.getJackPluginConfigurationName(),
                                String.format(
                                        "Classpath for the '%s' Jack plugins.",
                                        sourceSet.getName()));

                        sourceSet.setRoot(String.format("src/%s", sourceSet.getName()));
                    }
                });

		// 向容器添加item
        sourceSetsContainer.create(defaultConfig.getName());

		// 设置默认配置值
        setDefaultConfigValues();
    }
```



这里，我们关注下 buildTypes、productFlavors 、signingConfigs、defaultConfig 这一些，它们在构建脚本对应于一些对象（即带有 *{}* 的脚本块）。而不像脚本中的 compileSdkVersion、buildToolsVersion，它们只是属性。这些属性在 ***BaseExtension*** 中只有 getter/setter，而 buildTypes、productFlavors 、signingConfigs、defaultConfig 这一些还需要类似下面的方法，比如 productFlavors ：

```
public void productFlavors(Action<? super NamedDomainObjectContainer<ProductFlavor>> action) {
        checkWritability();
        action.execute(productFlavors);
 }
```

这是因为在调用 setter 方法时候，像 compileSdkVersion 一类的传递的是简单属性值，而 productFlavors 这一类的传递的是一个块（或者说是对象）。因此，需要 Action 的接口来处理该情况。

另外，在 buildTypes、productFlavors 、signingConfigs 里可以创建指定类型的命名对象（块），就像上面的 app 模块中的 buidle.gradle 中的 productFlavors：

```
productFlavors {
        flavor1 {
          ...
          nkd {
            ...
          }
        }
        flavor2 {
          ...
        }
}
```

其中，flavor1 和 flavor2 都属于 ProductFlavor 类型，但它们名字不同。这一特性由 NamedDomainObjectContainer 和其工厂类 ProductFlavorFactory 来完成。而 ProductFlavor 是一个实体类，它的构造函数类似 ***BaseExtension*** 。当然，属于 ProductFlavor 类型的 flavor1 里面可以包含了一些属性和脚本块 （比如 *ndk{}*）。



整个扩展里就是属性和脚本块的组合，它给整个项目提供了丰富的配置信息，这些信息被各种任务获取来完成如变体等配置处理。

