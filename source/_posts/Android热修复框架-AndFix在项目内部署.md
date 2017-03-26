---
title: 'Android热修复框架: AndFix在项目内部署'
date: 2017-03-26 18:42:43
tags: [Android,Java,热修复]
---

# 介绍

**AndFix** [Github: AndFix](https://github.com/alibaba/AndFix.git)（当前版本0.5.0）是一个通过Native层替换Java类函数指针，达到修改程序流程，实现热修复的框架。  
App使用该框架可以达到无声修复bug代码内bug的效果，用户不需要重新安装app。  
AndFix经测试可以在开启MultiDex的项目内使用，但仅能新增/修改Java内的方法，新增类/修改类属性不会产生修改，可以说局限性还是比较大。在项目中应用AndFix需要谨慎使用，多测试，因为一不小心App就可能会Crash。

## 目标： 将AndFix框架部署到Android项目中。本文会带你处理遇到的坑。

本文使用Android Studio，非Android Studio项目请自行解决。

1. 在build.gradle内添加 
```gradle
dependencies{
	compile 'com.alipay.euler:andfix:0.5.0@aar'
}
```

2. 这时候会遇到一个问题：如果你运行了APP，可能发现代码执行System.loadLibrary()时，会报so库找不到（如果你的项目中没有包含所有平台的so库），异常名字叫 java.lang.UnsatisfiedLinkError。此时需要在build.gradle 内添加ndk声明，指定你的项目只支持哪些平台的so库。
```gradle
    defaultConfig {
        ndk {
            // 设置支持的 SO 库构架
            abiFilters 'armeabi', 'armeabi-v7a'//, 'arm64-v8a', 'x86', 'x86_64', 'mips', 'mips64'
        }
......
```
 现在Gradle Sync一下，完成后就成功引入了andFix。

3. 添加一个简单的热修复工具类作为尝试。
```java
final public class PatchHelper {
    private static PatchManager sPatchManager;

    private PatchHelper() {
    }

    public static void init(Context applicationContext, String appVersion) {
        if (null != sPatchManager) {
            throw new IllegalStateException("already inited");
        }
        sPatchManager = new PatchManager(applicationContext);
        sPatchManager.init(appVersion);
        sPatchManager.loadPatch();
    }

    public static boolean addPatch(String patchPath) {
        if (null == sPatchManager) {
            throw new IllegalStateException("need init");
        }
        try {
            sPatchManager.addPatch(patchPath);
            return true;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return false;
    }
}

```
 你需要在Application初始化的时候，也就是onCreate()里，执行一下PatchHelper.init(getApplicationContext());   
 需要动态修复的时候，将apatch文件下载到手机存储内，执行PatchHelper.addPatch(apatch文件路径)即可完成修复。  
 
 ***本文不会讲解如何使用apkpatch工具生成apatch文件并传给app的过程。请自行解决。***   

 自己在一个Android项目中做了一些热修复实验，在实验过程中发现一些问题：

 1) 无法修改Activity中的方法，会报出错误。说是调用父类方法出java.lang.NoSuchMethodException了。  
 2) 无法修改带有System.loadLibrary(）调用的方法，会抛出java.lang.UnsatisfiedLinkError。较莫名其妙。

 最后自己尝试修改了一个简单类的方法行为（里面基本没啥东西，也没有继承自某个父类），成功了。

 ***个人感觉目前这个热修复框架局限性还是比较大的，感觉也不太成熟，较容易崩溃。投放到生产环境前需要进行大量的测试覆盖（尤其是新的patch要在测试机上过一遍），且仅仅建议作为紧急用途使用。***   

4. 上面编写的工具类对PatchManager 进行了封装。浏览一下源码很容易发现它也仅仅是为com.alipay.euler.andfix.AndFixManager类封装操作一个工具类。  

 这个PatchManager 类实际上来说，写的并不太好。一个很简单的多进程APP场景就令PatchManager显得笨拙：它会在addPatch时将你的apatch文件拷贝到app的fileDir内。若多个进程都要更新这个apatch，addPatch方法将会拷贝多次该文件。  
 一个简单的解决方案就是自己重新实现一个PatchManager。移除掉拷贝文件到fileDir的画蛇添足操作，将apatch直接下载到fileDir。为它添加单例/多进程更新广播/网络加载等实用功能。具体不细讲。  
 ****另外****，AndFix热修复可能直接导致用户的APP Crash掉，并且有可能是每次开启都会Crash，这就很尴尬了。除部署严格测试和发布灰度包以外，还可以考虑在Thread.setDefaultUncaughtExceptionHandler()中处理异常，根据抛出异常的情况自己清空本次更新（或者提示用户手动清除也行）。挽回一点用户体验。

# 总结
AndFix部署到项目的过程虽有一些坑，总体较为简单。AndFix能火线救急，但是局限性也较大，杀伤力也很大（本来不崩溃的应用可以用AndFix强行弄崩溃了）。在不同Android适配上据说也有不小的坑（未验证过）。两个字：****慎用****。