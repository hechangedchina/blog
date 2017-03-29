---
title: 一句话的Android增量更新框架
date: 2017-03-29 09:05:37
tags: [Java, Android, 增量更新]
---

Android应用更新要使用完整的新版本Apk安装，增量更新则是提供一个新旧版本偏差数据的patch包供应用下载，然后Android应用本地使用patch包和本地apk合成新版本apk。而patch包的体积通常都远小于新版本的apk，可以为用户节省流量和下载时间，节省时间就是**延续生命**，所以增量更新十分实用。

一些学习文章：   
  [Android应用的增量更新](https://hechangedchina.github.io/2017/03/27/Android%E5%BA%94%E7%94%A8%E7%9A%84%E5%A2%9E%E9%87%8F%E6%9B%B4%E6%96%B0/) 
  [Android 增量更新完全解析 是增量不是热修复](http://blog.csdn.net/lmj623565791/article/details/52761658)

资料里十分详细的介绍了如何在你自己的Android项目中部署增量更新功能，而实际上这个部署过程对新手来说是复杂而浪费时间的。它需要做配置NDK，并移植bsdiff/bspatch工具到Android系统，编写jni调用等麻烦事，**这是坠痛苦的**。

***I am Angry！！！ 你们这样搞是不行的！！！***

应运而生的**BigNews框架**（Github: [ha-excited/BigNews](https://github.com/ha-excited/BigNews)）为你省去了麻烦的增量更新部署过程，无需添加代码配置文件以及NDK编译，你只需要：

1. 在你项目根build.gradle添加代码：
 ```gradle
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}
 ```
2. 在你项目模块内的build.gradle添加代码，然后Gradle Sync：
 ```gradle
    dependencies {
        compile 'com.github.ha-excited:BigNews:0.1.1'
    }
 ```
3. 下载到patch文件后，你只需要写一句话，就可以合成新版本apk了。
```java
String oldApkPath = ...;
String newApkPath = ...;
String patchPath = ...;
//我就说一句话，这是坠吼的！
BigNews.make(oldApkPath, newApkPath, patchPath);
//已经弄出了一个大新。。安装包放在newApkPath路径下，随时准备升级！！
```

***简直是Too simple！！！excited！！！***

很惭愧，做了一点微小的工作， 谢谢大家。日-日
