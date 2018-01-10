---
title: Eclipse项目转Android Studio碰到的一些问题
date: 2016-11-02 14:07:28

tags:
- Android Studio
- Eclipse
- Gradle
- Android

categories:
- Android Studio


---
*<p align=""> ![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/preface.png) </p>*
> 前言

之前，组内有一个 Android 项目，由于历史原因，一直是在 `Eclipse` 上开发的。最近，要对项目进行全面更新。首先，就是要把项目从 Eclipse 上移到 `Android Studio（下面都简称 AS）` 上来。这两天，我负责把项目进行转换的工作，该项目依赖的 *lib* 工程比较多，虽然之前下面也在用 `AS`，但是都是全过程的再用，从新建项目到调试发布，没有转过，过程中还是遇到了一些蛋疼琐碎的问题，然后就想把这些问题和解决方法放在一起。`如果能帮到你，荣幸之至`。

<!-- more -->
从 Eclipse 导出项目到导入 AS，我会用自己的示例来大概演示一下，后面会放上实际项目迁移过程中遇到的问题和解决方法。
<br/>
# 1、Eclipse导出项目 #
<br/>
- 在 `Eclipse` 中打开你的项目，然后 *File -> Export* 或者在右键菜单中找到 *Export* ，打开导出项目弹出框。如下图，选中 *Generate Gradle build files*，点击 *Next*。

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/steps-1.png)

- 直接点击 *Next*，进入下一步操作

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/steps-2.png)

- 在该界面选择你要生成 *Gradle build files* 的项目，包括主工程和 *lib* 工程，选中后，直接点击 *Next*

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/steps-3.png)

- 在这一步，勾选覆盖已有文件这一项，点击 *Finish*，完成生成

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/steps-4.png)

<br/>
# 2、导入Android Studio #
<br/>
完成上面的步骤，在导入 `AS` 之前，我们来看一下都生成了什么东西，在工程目录外层生成了如下文件，见下图。 `build.gradle` 是属于 Project （项目）的文件，`gradle` 文件夹下面描述了所用的 Gradle 信息。

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/ori-add.png)

而刷新 Eclipse 项目，在每个工程中，可以看到增加了一个 `build.gradle` 文件，如下图。

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/lib-add.png)

接下来，我们就需要把项目导入 `AS` 了。打开你的 `AS`，选择下图中的选项

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/as.jpg)

然后我们可以从下图中看到整个项目的目录结构，基本上每个工程的目录结构和在  Eclipse 中是一样的结构，这和你通过 AS 去新建一个项目是不一样的，但这并不妨碍我们。

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/as-project-structure.png)

*`Note： 这里强烈建议先参照问题列表里的第一个问题，先修改Gradle 和 Android Plugin 版本后（也就是修改上图中 1 和 2 两个文件），再导入，省去不少麻烦`*

<br/>
# 3、问题列表（Issue list） #
<br/>
项目导入`Android Studio`后，一般总会问题多多，就本项目为例，大概有下面几个。。。。而且有的问题会反复出现，改到想哭。。。。

`PS： 所有报错信息中的路径和名称类似的都用 {.....} 或者 ‘xxxx’ 代替，具体每个人的项目路径和名称都不一样，到时候涉及到这些的时候，还需要具体对待`
<br/>
## 3.1、Gradle 和 Android Plugin 版本 ##
<br/>
通过 Eclipse 导出的项目，因为 adt 的原因，默认的 *Gradle* 和 *Android Plugin* 版本都比较低，需要提升下版本。但是，因为伟大的`墙`的存在，可能会下载不下来（出现下图的情况，一直停留在building）。

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/as-gradle-building.png)

这种情况，如果你没有翻墙的话，你可以改成你本机上 `Android Studio` 上已经成功运行项目中使用的 *Gradle* 和 *Android Plugin* 版本，*Gradle* 版本修改在项目的根目录下的 `gradle/wrapper/gradle-wrapper.properties` 文件中，打开已成功运行的项目中的该文件可以看到类似内容
```
#Mon Nov 30 13:15:11 EET 2015
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-2.10-all.zip
```
最后一行就是*Gradle* 版本，把刚刚导入的项目 *Gradle* 版本替换成这个版本。仅仅是这样，项目还可能不能好好的编译，有可能遇到下面两个问题：
```
> Could not resolve all dependencies for configuration ':classpath'.
   > Could not resolve com.android.tools.build:gradle:0.12.+.
     Required by:
         :client.android.studio:unspecified
      > ...
```
，或者
```
> Could not resolve all dependencies for configuration ':classpath'.
   > Could not resolve com.android.tools.build:gradle:0.12.+.
     Required by:
         :client.android.studio:unspecified
      > No cached version of com.android.tools.build:gradle:0.12.+. available for offline mode.
```
都是找不到该 *Android Plugin* 版本，前者是联网下载不到报错，后者是在设置成 `Offline work` 时找不到本地版本报错信息。我们也可以改成成功运行的项目的 *Android Plugin* 版本，这个版本设置在整个项目根目录下的 `build.gradle` 文件中, 修改 `classpath` 这一行最后的版本。
```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.12.+'
    }
}
.......
```
<br/>
## 3.2、配置合适的 buildToolsVersion ##
<br/>
`Eclipse` 为项目中每个工程单独生成一个 `build.gradle` 文件，这个文件包括以下内容（里面加了一些注释信息）：
```
// 指示是主工程还是 Module，这个自动生成的值已经被废弃了，
// 主工程要替换为'com.android.application'，
// 其他的要替换为'com.android.library'
// 这里'android'表示是主工程

apply plugin: 'android'

dependencies {
	// 引用的 jar 包和位置
    compile fileTree(dir: 'libs', include: '*.jar')
	// 引用的 Module
    compile project(':library')
}

android {
	// sdk 版本
    compileSdkVersion 21
	// buildTools 版本
    buildToolsVersion "25.0.0"

	// 代码和资源文件目录
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        // Move the tests to tests/java, tests/res, etc...
        instrumentTest.setRoot('tests')

        // Move the build types to ...
        debug.setRoot('build-types/debug')
        release.setRoot('build-types/release')
    }
}
```
其中，`buildToolsVersion "25.0.0"` 是 ADT 为你生成时设置的，一定要结合你使用的 JDK 版本改成适合的 buildTools 版本，比如，我用的 JDK1.7，就会报下面的错误。
```
Error:java.lang.UnsupportedClassVersionError: com/android/dx/command/Main : Unsupported major.minor version 52.0
```
<br/>
## 3.3、找不到 Java 类 ##
<br/>
下面的这个信息是因为某个 Module 中使用到了 `sun.misc.BASE64Encoder` 类，而在 Eclipse 的项目中，加入了 *JRE System Library*（其中的 rt.jar 包含 `sun.misc.BASE64Encoder` 类），在 AS 中也引用到了 `rt.jar`，但没有这这一块的代码。去网上找到需要的jar包（不建议直接使用安装的 jdk 目录下面的 rt.jar，因为实在是太大啦。居然有 60+M。。。）放到 Module 下的 libs 下 sync 下项目就行了。 
```
{....}/Basic64Util.java:7: error: package sun.misc does not exist
import sun.misc.BASE64Encoder;
               ^
```
<br/>
## 3.4、AndroidManifest.xml 文件合并 ##
<br/>
```
{....}/AndroidManifest.xml:70:9-39 Error:
Attribute application@theme value=(@style/MyTheme) from AndroidManifest.xml:70:9-39
is also present at [{....}:unspecified] AndroidManifest.xml:27:9-40 value=(@style/AppTheme).
Suggestion: add 'tools:replace="android:theme"' to <application> element at AndroidManifest.xml:65:5-465:19 to override.

See http://g.co/androidstudio/manifest-merger for more information about the manifest merger.

:{....}:processDebugManifest FAILED

```
项目原来在 Eclipse 中时，每个 lib 工程一开始都是个可以单独运行的 App，每个都带有一个完整的 AndroidManifest.xml 文件（即都有入口的 Activity）。在 Eclipse 中打包 *apk* 时，以主工程的  AndroidManifest.xml 为准，即所有的 Activity、Service 等都要在主工程中注册，lib 工程的 AndroidManifest.xml 会被忽略掉。`AS `这边似乎是对所有的 Module 的 AndroidManifest.xml 进行了合并处理，这边运行直接报错。解决方法是打开主工程的  AndroidManifest.xml 文件，在*manifest* 节点增加：

> xmlns:tools="http://schemas.android.com/tools"

在*application* 节点中增加：

> tools:replace="android:theme"

这里可以一次添加多个属性，用*,* 分割。例如*tools:replace="android:theme, android:icon, android:label"*。

后来，程序发现了一个问题，就是安装*apk* 后，出现了多个 App 图标，把除主工程之外的所有 lib 工程中的 AndroidManifest.xml 中的那些入口 Activity 都去掉。
<br/>
## 3.5、.9 资源文件 ##
<br/>
```
:{....}:bundleDebugAAPT err(Facade for 304835512) : No Delegate set : lost message:ERROR: 9-patch image {....}/build/intermediates/bundles/debug/res/drawable-hdpi-v4/skin_common_btn_red_unpressed.9.png malformed.
AAPT err(Facade for 304835512) : No Delegate set : lost message:       Frame pixels must be either solid or transparent (not intermediate alphas).
AAPT err(Facade for 304835512) : No Delegate set : lost message:       Found at pixel #1 along top edge.
....
```
这个问题比较蛋疼， Eclipse 转 AS 的时候报出了很多这种问题，主要原因是 *.9* 文件不符合 AS 的规范，但是在  Eclipse 可以使用（比如，有的*.9* 图片，拉伸部分，就是那道黑线没出来，可以在 AS 中打开该图片进行操作），但是大多数的时候都是没办法，只能找你们的 UI 重新画。。。这也是我遇到的问题里，唯一是单靠程序猿自己不一定能搞定的事情，当然有特长的除外啦。
<br/>
## 3.6、 PNG 资源文件 ##
<br/>
```
:{....}:mergeDebugAndroidTestResourcesAAPT err(Facade for 1698576962) : No Delegate set : lost message:libpng error: Not a PNG file
 FAILED

FAILURE: Build failed with an exception.
```
碰到这个问题的时候，简直是坑，从字面意思上来看就是碰到了一个*非 PNG 文件*的错误，一时摸不着头脑。后来找到问题的原因是在 Eclipse 中你可以把一个*.jpg* 文件通过修改后缀的方式，改为*.png* 文件使用。但是， AS 中这样做是不被认可的。只能说资源文件的问题，大部分是两个标准的问题，AS 更加严苛，当然这也是好的。不过问题的关键是报错信息里没有给出来是哪一个资源文件。。。。解决方法是根据报错信息的工程 Module 去资源文件里找，双击图片，在右侧可以看到该图片实际上的格式，找出来不是 PNG 格式的图片转换一下就 OK 了。。。。

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/png-err.png)


```
{....}/res/drawable-hdpi/bell_50.758620689655px_1183394_easyicon.net.png: Error: '.' is not a valid file-based resource name character: File-based resource names must contain only lowercase a-z, 0-9, or underscore
:nari.mip.console:mergeDebugResources FAILED

FAILURE: Build failed with an exception.
```
这个问题比较简单，资源文件命名不合符 AS 的规范，改掉就OK了。。
<br/>
## 3.7、编译死机 ##
<br/>
执行编译后，下面提示条停在 `Gradle Build Running` 就不动了，机器直接死机。。。。最后 *OOM* 了。。。基本上遇到的报错信息大概下面两种
```
:{....}: {"kind":"error","text":"UNEXPECTED TOP-LEVEL ERROR:","sources":[{}]}
AGPBI: {"kind":"error","text":"java.lang.OutOfMemoryError: **GC overhead limit exceeded**","sources":[{}]}
```
可以在 *Users/{username}/.gradle/gradle.properties* 文件中添加这个参数：

> org.gradle.jvmargs=-XX:-UseGCOverheadLimit 

该问题可以参考 [java.lang.OutOfMemoryError：GC overhead limit exceeded填坑心得](http://www.cnblogs.com/hucn/p/3572384.html)。
```
AGPBI: {"kind":"error","text":"UNEXPECTED TOP-LEVEL ERROR:","sources":[{}]}
AGPBI: {"kind":"error","text":"java.lang.OutOfMemoryError: Java heap space","sources":[{}]}
AGPBI: {"kind":"error","text":"\tat com.android.dx.util.Bits.makeBitSet(Bits.java:38)","sources":[{}]}
```
主要是修改 AS 的参数进行优化，增大运行内存。在 *Users/{username}/.gradle/gradle.properties* 文件中添加：
> org.gradle.daemon=true
> org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
> org.gradle.parallel=true
> org.gradle.configureondemand=true

第一个是开启守护进程，第二个是更改 jvm 的参数，第三个是开启并行任务，第四个是按需配置开关。关于优化的问题可以参考下面这个文章 [Android Studio 运行、编译卡死的解决办法](http://blog.csdn.net/cswhale/article/details/51028242)。
<br/>
## 3.8、jar包冲突 ##
<br/>
项目之前用的是 Eclipse，用到了一些 github 上的第三方库，不过是用引用 jar包的方式，转到 AS 的过程中，这些文件都放在各个 lib 工程下的*libs* 文件目录中去了，而每个 lib 工程的*build.gradle* 文件都有这么一句：
```
dependencies {
	compile fileTree(dir: 'libs', include:'*.jar')
}
```
因为项目中有自己的 jar 包和第三方的 jar 包，并且各个 lib 工程应用的 jar 包有重复，sync 的时候不会报错，但是生成 apk 的时候就会报出问题：
```
> Execution failed for task ':{...}:transformResourcesWithMergeJavaResForDebug'.
> com.android.build.api.transform.TransformException: com.android.builder.packaging.DuplicateFileException: Duplicate files copied in APK META-INF/maven/com.alibaba/fastjson/pom.properties
```
，或者报引用我们自己的核心业务 jar 包冲突的报错（因为核心业务比较敏感，只能提供 jar 包方式），类似：
```
....Lxxxxx.xxx.xx alread added.....
```
解决方法，有下面三个：

- 方式一：创建 jar Module

新建一个 Module，如下图，选择 *Import .JAR/.AAR Package* 的方式

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/jar-module.png)

然后在需要引用的地方，在其 *build.gradle* 文件中添加依赖：
```
dependencies {
	compile project（'：{project name}')
}
```
- 方式二：provided 方式

我们打开 *Project Structure* 对话框，然后在 *Dependencies* 选项卡，在下拉菜单中可以看到这样的几项：

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/dependencies.png)

主要讲两种，一种是 *compile*，一种是 *provided*。前者是参与编译并在生成 apk 时打入包；而后一种则是只在编译时引用，但并不打入 apk 中。而我们的 jar 包冲突就是因为在生成 apk 的时候检测到有多个重复的 jar 包造成的。因为多个 lib 工程都用到了自己的核心业务 jar 包和第三方 jar 包，而且是 *compile* 的方式，自然就会出问题了。

解决方式就是，假如一个 jar 包被多次引用依赖，其中一个使用 *compile*，其他都使用 *provided*。特别注意的是要去掉 *compile fileTree(dir: 'libs', include:'*.jar')* 这一句，它会把 *libs* 目录下的所有 jar 包以 *compile* 方式引用。我的方式是，把需要打入到 apk 中的所有 jar 包都放在 *libs* 目录下，在 *build.gradle* 中使用：
```
dependencies {
	compile fileTree(dir: 'libs', include:'*.jar')
}
```
而在其他的 lib 工程中，把自己编译时需要的 jar 包放在 *libs* 目录下，在 *build.gradle* 使用：
```
dependencies {
	provided files('libs/xxx.xxx.xx1.jar')
	provided files('libs/xxx.xxx.xx2.jar')
	...
}
```
其他的方式可以参考这个：[AndroidStudio中多个Module依赖同一个jar的解决方案](http://blog.csdn.net/u013134391/article/details/51538511)

- 方式三：上传 maven 库

这种是最麻烦（对于本项目来讲的话，核心业务 jar 包比较少）的方式了，一开始想到的是按照类似 v4 包这样的思路，因为在 Eclipse 项目中，lib 工程的*libs* 目录中是有 v4 包的，因此，当你迁移到 AS 上来的时候 v4 包实际上也存在冲突的问题，那么一般的解决方法是对其进行用下面的引用依赖方式：
```
dependencies {
	compile 'com.android.support:support-v4:21.+'
}
```
这样的方式是不会出现冲突的问题的，还有就是用这种方式引用的其他第三方包，一般 jar 包来自 *sdk* 或者是*Users/{username}/.gradle/caches* 下面的缓存（没有的就从网络下载），另外就是*gradle* 可以使用本地*maven* 库，如果你本机没有存在的*maven* 库，默认的位置是*Users/{username}/.m2/repository*，但是直接把 jar 包放在这些目录下面是不行的，因为除了 jar 包，还有其他的一大堆文件。万幸的是，我们可以编写 gradle *upload* task 来上传到本地库（当然也可以上传到远程库），然后以上面的这种方式引用。

首先，将自己的核心业务 jar 包源码单独放到一个 Module 工程里去，然后配置 *maven* 库地址：
```
repositories {
    mavenCentral()
    maven { url "maven respository URL" }//依个人情况而定
}
```
添加 *maven* 发布插件:
```
apply plugin: 'maven-publish'
apply plugin: 'maven'
```
定义属性以及上传目标仓库地址:
```
def MAVEN_LOCAL_PATH ='$(your private maven repository URL)'
def ARTIFACT_ID = 'xxx'
def VERSION_NAME = '1.0.0-SNAPSHOT'
def GROUP_ID = 'xxx.xx'
```
最后，在其 *build.gradle* 文件中编写 *upload* 部分：
```
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url:MAVEN_LOCAL_PATH ){
				//这里，我们上传到本地库
                //authentication(userName: “$(user_name)”, password:“$(passport)”)
            }
        pom.project {
        groupId GROUP_ID
        artifactId ARTIFACT_ID
        version VERSION_NAME
        packaging 'aar'
            }
        }
    }
}
```
 sync一下，会看到出现 *upload* 任务。最后，双击执行该任务：

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/upload-task.png)

最后，按照引用 v4 包的方式进行引用依赖就 OK 了~~
<br/>
## 3.9、Note/Warning ##
<br/>
其他的是一些 *Note/Warning* ，主要是 *Permission* 重复的警告信息，手动修改或者忽略。。。
<br/>
# 4、其他 #
<br/>
另外，可能是我们项目中的问题了，因为原来的一些 App 中的模块，比如*消息*、*新闻* 等等的，都是单独的 lib 工程，转到 AS 中成了一个个 Module 了，这样的后果就是编译一次非常的慢，比在 Eclipse 中还慢，你敢想象吗。。。。。造成的原因是因为 Module 太多了，造成 gradle 的 task list 非常多，每一个都非常的耗时。你可以打开你的 *Gradle Projects* 看一下，下图是 *Phoenix* 的 task list 的一部分：

![](http://ofdub8np7.bkt.clouddn.com/2016/11/02/gradle-tasks.png)

除了 *root* ，每一个 Module 都会有一堆的 task 来执行，如果你只有一两个  Module 可能还没有什么，假如你有十几个的时候，你想一下。。。。
所以，尽量还是减少无谓的多个 Module 的方式对代码进行分块了。。 
<br/>