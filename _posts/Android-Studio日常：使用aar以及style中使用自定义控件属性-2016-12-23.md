---
title: Android Studio日常：本地仓库引用方式使用 aar 以及 style 中使用自定义控件属性
date: 2016-12-23 10:22:13

tags:
- Java
- Android
- Android Studio
- aar
- style

categories:
- Android

---

> 前言 

昨天项目里遇到了两个小问题，都是基本的使用问题：
- 本地仓库引用方式使用 aar；
- style 中使用自定义控件的属性

索性就把找到的解决方法写下来。


<!--- more --->
<br>

# 1、本地引用 aar

<br>

昨天，项目里想找一个好一些的*RatingBar*控件，公司无法正常登陆 `GitHub`，就在手机上查了几个库。都不太理想，不是依赖的包版本太高，一般的问题都是尺寸不能灵活的定义，定小了就会产生不良反应。最后找到了一款符合要求的库，不是图片替换方式的，尺寸能够随意设定。不过限制就是它是一个只支持多边形的 [*RatingBar*](https://github.com/Amagi82/FlexibleRatingBar)，另外就是不同于其他项目，没法通过一般引用第三方库的方式使用，类似 `build.gradle` 中这样写：

```
compile 'io.reactivex:rxjava:1.1.0'
```

在上面却是这样的一句话:

```
(currently getting this set up with maven/jcenter. will update when available)
```

o(╯□╰)o ，不过所幸的是library代码可以下下来，然后把它打成了 `aar` 文件放在主工程 `Module` 里来用了。本来以为和 `jar` 文件一样的使用方式就行了，放到 `libs` 目录下面，然后 `build.gradle` 中这样写：

```
dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    compile fileTree(dir: 'libs', include: '*.aar')
	....
}
```
结果一运行，就报了类似下面的错误：

```
only jar-type local dependencies supported.....
```

哦漏，不行啊。改变方式，于是乎变成本地仓库引用的方式，下面这样。。。。

```
//添加本地仓库，libs目录作为仓库的地址
repositories {
    flatDir {
        dirs 'libs'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    //compile fileTree(dir: 'libs', include: '*.aar')

    //添加本地aar依赖
    compile(name: 'flexibleratingbar', ext: 'aar')
}
```

OK，完美应用。。。。

<br>


# 2、style 中使用自定义控件的属性

<br>
就刚才的控件使用，项目里用到的地方比较多，项目主页上给出了使用方式：

```
<amagi82.flexibleratingbar.FlexibleRatingBar
  android:id="@+id/flexibleRatingBar"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:numStars="6"
  android:rating="1.5"
  android:stepSize="0.5"
  app:colorFillOff="@android:color/transparent"
  app:colorFillOn="@color/tealA400"
  app:colorFillPressedOff="@android:color/transparent"
  app:colorFillPressedOn="@color/tealA200"
  app:colorOutlineOff="@color/teal800"
  app:colorOutlineOn="@color/teal900"
  app:colorOutlinePressed="@color/teal500"
  app:polygonVertices="7"
  app:polygonRotation="0"
  app:strokeWidth="5dp"/>
```

这一大推的属性，任谁 *Command + C/V* 的一路下来也要吐了，那我们就把它抽出去一些写成复用的 `style`，然后在使用的 `layout文件` 里就变成了：

```
  <amagi82.flexibleratingbar.FlexibleRatingBar
            android:id="@+id/ratingBar"
            style="@style/ratingbar_style"/>
```

`style.xml` 文件里是这样：

```
<resources xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto">
	
	<style name="ratingbar_style">
       ...
        <item name="android:rating">0</item>
        <item name="android:numStars">5</item>
        <item name="android:stepSize">0.5</item>
        <item name="android:clickable">true</item>
        <item name="app:colorFillOff">@color/trans</item>
        <item name="app:colorFillOn">@color/little_yellow</item>
        <item name="app:colorFillPressedOff">@color/trans</item>
        <item name="app:colorFillPressedOn">@color/little_yellow</item>
        <item name="app:colorOutlineOff">@color/little_yellow</item>
        <item name="app:colorOutlineOn">@color/little_yellow</item>
       ...
	</style>
</resources>
```

一运行，报错属性找不到。。。后来找了下这个问题，说是命名空间改成包名的，这样：

```
<resources xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res/amagi82.flexiblratingbar">
	
	...
</resources>
```

再运行，还是报错属性找不到。。。，然后继续找，变成了下面这样：


```
<resources xmlns:android="http://schemas.android.com/apk/res/android">
	<style name="ratingbar_style">
      ...
        <item name="amagi82.flexiblratingbar:colorFillOn">@color/little_yellow</item>
        <item name="amagi82.flexiblratingbar:colorFillPressedOff">@color/trans</item>
        <item name="amagi82.flexiblratingbar:colorFillPressedOn">@color/little_yellow</item>
      ...
	</style>
</resources>
```

再次运行，继续报错属性找不到。。。泪奔 ┭┮﹏┭┮，没法子继续找，最后成了下面这样：

```
<resources xmlns:android="http://schemas.android.com/apk/res/android">
	<style name="ratingbar_style">
      ...
        <item name="com.demo.app:colorFillOn">@color/little_yellow</item>
        <item name="com.demo.app:colorFillPressedOff">@color/trans</item>
        <item name="com.demo.app:colorFillPressedOn">@color/little_yellow</item>
      ...
	</style>
</resources>
```

运行，通过 :-D，其中 `com.demo.app` 为主工程 `Module` 即应用的包名
