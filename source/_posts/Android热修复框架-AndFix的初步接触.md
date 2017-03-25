---
title: 'Android热修复框架: AndFix的初步接触'
date: 2017-03-25 14:31:05
tags: [Android,Java,热修复]
---

# 介绍

**AndFix** [Github: AndFix](https://github.com/alibaba/AndFix.git)（当前版本0.5.0）是阿里推出的Android热修复方案。  
市面上方案较多，选择它是因为它的效果比目前市面上其他的修复方案要好（修复方案比较不是本文重点，本文仅做实践记录）。  
AndFix方案只能对代码进行热修复实现修复代码bug，而无法热更新资源。

## 目标：在AndFix提供的Demo中使用AndFix，试验热修复Demo中的方法。

# Demo配置

1. 从Github下载AndFix源码。

2. 打开Android Studio 2.3，File->New->Import Project导入AndFix项目。

3. File->New->Import Project导入AndFixDemo项目。（在AndFix项目中sample文件夹中）。

4. AndFixDemo导入完成后，项目应该会报错。因为AndFixDemo中还没有添加AndFix库。

5. 切换到AndFix项目，打开命令行，输入gradle assembleRelease编译AndFix。编译完成后在项目的build/output/aar目录找到andfix-release.aar。

6. 将andfix-release.aar拷贝到AndFixDemo项目内的app/libs文件夹（若没有该文件夹，请新建）。

7. 打开AndFixDemo项目的app/build.gradle文件，添加以下代码，
```gradle
repositories {
    flatDir {
        dirs 'libs'
    }
}

dependencies {
    compile(name: 'andfix-release', ext: 'aar')
}

```
然后Sync一下项目。

这时候报了个错：

```
Error:Execution failed for task ':app:processDebugManifest'.
> Manifest merger failed : uses-sdk:minSdkVersion 8 cannot be smaller than version 9 declared in library [:andfix-release:] C:\Users\anonymous\.android\build-cache\5887af3b2772a1bf81b83033920436ff6841c8de\output\AndroidManifest.xml
	Suggestion: use tools:overrideLibrary="com.alipay.euler.andfix" to force usage
```

把app/build.gradle中 android { defaultConfig { minSdkVersion从8 改为9，重新Sync。 错误消失，项目Demo配置完成。

## 上面说的是本地编译&添加方法，也可以通过在build.gradle中加入以下代码引入andfix库：
```gradle
dependencies {
	compile 'com.alipay.euler:andfix:0.5.0@aar'
}
```


# 热修复使用

打开Android设备，运行Android Studio编译AndFixDemo安装App并运行。 

App运行后，会执行

```java
        Log.e(TAG, A.a("good"));
        Log.e(TAG, "" + new A().b("s1", "s2"));
        Log.e(TAG, "" + new A().getI());
```

相关类：

```java
public class O {
    public String s = "s";

    public O(String s) {
        this.s = s;
    }

    @Override
    public String toString() {
        return s;
    }
}
.......
.......
public class A {
    static int i = 10;
    private static O o = new O("a");
    String s = "s";

    public static String a(String str) {
        Log.i("euler", "fix error");
        return "a";
    }

    public int b(String s1, String s2) {
        Log.i("euler", "fix error");
        Log.i("euler", o.s);
        return 0;
    }

    public int getI() {
        return i;
    }
}

```

可以看到log输出为：

```
03-25 08:07:39.193 20868-20868/com.euler.andfix D/euler: inited.
03-25 08:07:39.194 20868-20868/com.euler.andfix D/euler: apatch loaded.
03-25 08:07:39.206 20868-20868/com.euler.andfix I/euler: fix error
03-25 08:07:39.206 20868-20868/com.euler.andfix E/euler: a
03-25 08:07:39.206 20868-20868/com.euler.andfix I/euler: fix error
03-25 08:07:39.206 20868-20868/com.euler.andfix I/euler: a
03-25 08:07:39.206 20868-20868/com.euler.andfix E/euler: 0
03-25 08:07:39.206 20868-20868/com.euler.andfix E/euler: 10
```

***现在开始！！！***
目标：尝试修改类A的行为。

#### 1. 在AndFix项目根目录找到tools文件夹，解压里面的apkpatch-1.0.3.zip，得到文件夹apkpatch-1.0.3，执行一下apkpatch.bat。
```
X:\apkpatch-1.0.3>apkpatch.bat
ApkPatch v1.0.3 - a tool for build/merge Android Patch file (.apatch).
Copyright 2015 supern lee <sanping.li@alipay.com>

usage: apkpatch -f <new> -t <old> -o <output> -k <keystore> -p <***> -a <alias> -e <***>
 -a,--alias <alias>     alias.
 -e,--epassword <***>   entry password.
 -f,--from <loc>        new Apk file path.
 -k,--keystore <loc>    keystore path.
 -n,--name <name>       patch name.
 -o,--out <dir>         output dir.
 -p,--kpassword <***>   keystore password.
 -t,--to <loc>          old Apk file path.

usage: apkpatch -m <apatch_path...> -k <keystore> -p <***> -a <alias> -e <***>
 -a,--alias <alias>     alias.
 -e,--epassword <***>   entry password.
 -k,--keystore <loc>    keystore path.
 -m,--merge <loc...>    path of .apatch files.
 -n,--name <name>       patch name.
 -o,--out <dir>         output dir.
 -p,--kpassword <***>   keystore password.

X:\apkpatch-1.0.3>
```
Demo的MainApplication模块实现了载入patch并应用的功能：
```java
public class MainApplication extends Application {
    private static final String TAG = "euler";

    private static final String APATCH_PATH = "/out.apatch";
    /**
     * patch manager
     */
    private PatchManager mPatchManager;

    @Override
    public void onCreate() {
        super.onCreate();
        // initialize
        mPatchManager = new PatchManager(this);
        mPatchManager.init("1.0");
        Log.d(TAG, "inited.");

        // load patch
        mPatchManager.loadPatch();
        Log.d(TAG, "apatch loaded.");

        // add patch at runtime
        try {
            // .apatch file path
            String patchFileString = Environment.getExternalStorageDirectory()
                    .getAbsolutePath() + APATCH_PATH;
            mPatchManager.addPatch(patchFileString);
            Log.d(TAG, "apatch:" + patchFileString + " added.");
        } catch (IOException e) {
            Log.e(TAG, "", e);
        }

    }
}
```
以上代码是AndFix的***使用方法！！！***

apkpatch工具可以为你生成一个patch文件，按照代码中描述，将apkpatch生成的patch文件放到/sdcard/out.apatch目录就可以了。  
App启动后会将patch载入，并更新app。

#### 2. 生成keystore，并为apk签名。
1) 切到AndFixDemo项目，在Android Studio 中点击Build->Generated Signed Apk， 生成一个keystore命名为key.keystore保存在AndFixDemo项目根目录。  
我这里密码是123456，alias是key，信息全部是乱写的。

2) 使用Build->Generated Signed Apk编译出一个apk文件，重命名为old.apk。就放在app目录。

3) 改类A为

```java
public class A {
    static int i = 20; //从10修改到20
    private static O o = new O("b"); //从"s" 修改为 "b"
    String s = "s";

    public static String a(String str) {
        Log.i("euler", "fix success"); //从 "error" 修改为 "success"
        return str; //从"a" 改为 str
    }

    public int b(String s1, String s2) {
        Log.i("euler", "fix success"); //从 "error" 修改为 "success"
        Log.i("euler", o.s);
        return 0;
    }

    public int getI() {
        return i;
    }
}
```

4) 再使用Build->Generated Signed Apk编译出一个apk文件，重命名为new.apk。就放在app目录。

5) 把
```bat
@set /p APK_PATCH=请输入apkpatch.bat路径名：
%APK_PATCH% -f app/new.apk -t app/old.apk -o out.apatch -k key.keystore -p 123456 -a key -e 123456
```
存到文件patch.bat，放在AndFixDemo项目根目录。

6) 执行patch.bat
```
X:\project\AndFixDemo>patch
请输入apkpatch.bat路径名：X:\apkpatch-1.0.3\apkpatch.bat

X:\project\AndFixDemo>X:\apkpatch-1.0.3\apkpatch.bat -f app/new.apk -t app/old.apk -o out.apatch -k key.keystore -p 123456 -a key -e 123456
add modified Method:Ljava/lang/String;  a(Ljava/lang/String;)  in Class:Lcom/euler/test/A;
add modified Method:I  b(Ljava/lang/String;Ljava/lang/String;)  in Class:Lcom/euler/test/A;

X:\project\AndFixDemo>
```
工具告诉我们已经做好两个方法的patch了。  
这里apkpatch为我们生成了一个文件夹，里面放了几个文件。

```
X:\project\AndFixDemo>dir out.apatch
 驱动器 X 中的卷是 SSD
 卷的序列号是 A64D-1BE1

 X:\project\AndFixDemo\out.apatch 的目录

2017/03/25  19:54    <DIR>          .
2017/03/25  19:54    <DIR>          ..
2017/03/25  19:54             1,316 diff.dex
2017/03/25  19:54             2,846 new-28ef61a629487761bedf0f2f0c4ac17f.apatch
2017/03/25  19:54    <DIR>          smali
               2 个文件          4,162 字节
               3 个目录 66,200,182,784 可用字节

X:\project\AndFixDemo>
```
其中new-28ef61a629487761bedf0f2f0c4ac17f.apatch这个文件就是起作用的。

我们先安装old.apk，然后运行。 日志如下：
```
03-25 08:07:39.193 20868-20868/com.euler.andfix D/euler: inited.
03-25 08:07:39.194 20868-20868/com.euler.andfix D/euler: apatch loaded.
03-25 08:07:39.206 20868-20868/com.euler.andfix I/euler: fix error
03-25 08:07:39.206 20868-20868/com.euler.andfix E/euler: a
03-25 08:07:39.206 20868-20868/com.euler.andfix I/euler: fix error
03-25 08:07:39.206 20868-20868/com.euler.andfix I/euler: a
03-25 08:07:39.206 20868-20868/com.euler.andfix E/euler: 0
03-25 08:07:39.206 20868-20868/com.euler.andfix E/euler: 10
```
他现在是这么个提示。

我们把new-28ef61a629487761bedf0f2f0c4ac17f.apatch上传到手机的/sdcard/out.apatch路径里

```
X:\project\AndFixDemo>adb push out.apatch\new-28ef61a629487761bedf0f2f0c4ac17f.apatch /sdcard/out.apatch
[100%] /sdcard/out.apatch

X:\project\AndFixDemo>
```

然后再开app，发现log变了
```
03-25 08:14:54.119 27167-27167/com.euler.andfix D/euler: inited.
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A.s flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 1 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A.i flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 9 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A.o flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 9 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: replace_5_1: -1354706580 , -1354706580
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A_CF.s flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 1 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A_CF.i flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 9 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A_CF.o flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 9 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: replace_5_1: 1927159936 , 1927159936
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A_CF.s flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 1 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A_CF.i flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 9 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: modify com.euler.test.A_CF.o flag:
03-25 08:14:54.128 27167-27167/com.euler.andfix D/AndFix: setFieldFlag_5_1: 9 
03-25 08:14:54.128 27167-27167/com.euler.andfix D/euler: apatch loaded.
03-25 08:14:54.139 27167-27167/com.euler.andfix I/euler: fix success
03-25 08:14:54.139 27167-27167/com.euler.andfix E/euler: good
03-25 08:14:54.139 27167-27167/com.euler.andfix I/euler: fix success
03-25 08:14:54.139 27167-27167/com.euler.andfix I/euler: a
03-25 08:14:54.139 27167-27167/com.euler.andfix E/euler: 0
03-25 08:14:54.139 27167-27167/com.euler.andfix E/euler: 10

```
可以看到里面的两个方法调用的确修改成功了，但是改动的两个静态变量的影响没有到andFix里面。  

通过观察日志可以做出猜测：里面有两个replace，应该代表修改了两个方法名字。

用命令把/sdcard/out.apatch给删掉，发现log依旧是修复过的。框架应该已经把修改应用到app里了。

# 总结
AndFix项目提供了一个Demo，Demo已经实现了patch的代码流程。  
本文的实验修改了Demo中的类，并用工具通过比较新老app生成更新patch。  
可以看到AndFix框架读取到更新patch后成功的修复了类内的一个类方法和对象方法，而对类变量修改却无效。  
若在项目内部署了这样一个热更新框架，的确能够实现远程无声修bug。

## 若想应用AndFix到生产环境，还需要
1) 大概了解内部原理机制
2) 编写测试代码测试它的修改生效范围
3) MultiDex对它的影响
4) 代码混淆
## 不过这篇文章的目标已经达成了。