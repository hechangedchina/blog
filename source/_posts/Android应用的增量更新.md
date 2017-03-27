---
title: Android应用的增量更新
date: 2017-03-27 14:12:24
tags: [Java, Android, 增量更新]
---

# 介绍

Android应用更新版本时需要下载apk文件进行安装，而现在的apk都比较大，一次下载一个完整的apk包太冤枉。因此增量更新应运而生。

****原理很简单****：通过linux工具bsdiff计算新老apk文件的差异，将差异记录为一个体积较小的patch包。通过bspatch工具（和bsdiff配套的）将老apk文件与这个patch包合并为新apk文件并安装，达到增量更新的目的。  

****由此可得方案****：应用在做版本更新操作时获取到了版本号有更新，就将本地的老apk包的MD5校验值与版本号提交给服务器。服务器检验md5和版本号后，把使用bsdiff生成的patch包下载地址与最新版本apk的MD5校验值传回给应用。  
手机端应用程序下载到patch后，使用***内置的bspatch工具***将自己本地旧的apk文件与patch包合并为新版本apk文件，计算校验值正确便开始安装。整个流程若校验值错误，则下载全量包安装。

## 目标： 编写一个demo，实现通过读取磁盘上的patch文件实现自我更新。

1. 首先实验一下bsdiff/bspatch工具是否能够正常工作，达到预期效果。  

 本人在Win10使用了Linux子系统功能，自带了一个Ubuntu14.04，直接用命令安装bsdiff。

 ```bash
➜  ~ sudo apt-get install bsdiff
[sudo] password for administrator:
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  bsdiff
0 upgraded, 1 newly installed, 0 to remove and 98 not upgraded.
Need to get 14.5 kB of archives.
After this operation, 69.6 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu/ trusty/universe bsdiff amd64 4.3-15 [14.                                                                                                                                                             5 kB]
Fetched 14.5 kB in 10s (1,335 B/s)
Selecting previously unselected package bsdiff.
(Reading database ... 33465 files and directories currently installed.)
Preparing to unpack .../bsdiff_4.3-15_amd64.deb ...
Unpacking bsdiff (4.3-15) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Setting up bsdiff (4.3-15) ...
➜  ~ bsdiff
bsdiff: usage: bsdiff oldfile newfile patchfile
➜  ~ bspatch
bspatch: usage: bspatch oldfile newfile patchfile
 ```

 我们可以看到bsdiff/bspatch都正常安装了。

 接下来，这边生成了两个Android apk，old.apk和new.apk。new.apk相对old.apk修改了一点代码。  
 我们这里把它们的md5都算出来。

 ```bash
➜  app git:(master) ✗ ll *.apk
-rwxrwxrwx 1 root root 22M Mar 27 09:56 new.apk
-rwxrwxrwx 1 root root 22M Mar 27 09:32 old.apk
➜  app git:(master) ✗ md5sum old.apk
045382b701ce372aecd5bb87ee1e526d  old.apk
➜  app git:(master) ✗ md5sum new.apk
cbb1afdbc32e4d1c62c4d38674a6a3a9  new.apk
 ```

 然后用bisdiff 通过old/new.apk文件 打出一个patch

 ```bash
➜  app git:(master) ✗ bsdiff old.apk new.apk patch
➜  app git:(master) ✗ ll patch
-rwxrwxrwx 1 root root 3.5M Mar 27 13:52 patch
 ```

 打patch包过程时间稍微有点长，十几秒左右。可以看到打出来的patch包只有3.5M。

 接下来实验一下将old.apk与patch包合并，能否得到刚才的new.apk

 ```bash
➜  app git:(master) ✗ bspatch old.apk out.apk patch
➜  app git:(master) ✗ md5sum out.apk
cbb1afdbc32e4d1c62c4d38674a6a3a9  out.apk
 ```

 可以看到这里out.apk的MD5校验值和之前的new.apk是一样的。这个工具通过了实验，若能将它应用到项目，每次只用下载很少的数据就能完成版本更新。

2. 如何将它应用到项目呢？按照刚才的实验，可以发现合包需要bspatch工具。我们需要做的是将bspatch加入到Android代码中。

 一个显而易见的方案就是将bspatch的源码（地址：[bsdiff](http://www.daemonology.net/bsdiff/)）使用ndk编译为so文件，通过jni调用。这样有点麻烦，不过我们可以方便一点：  
 有个叫SmartAppUpdate（[Github: SmartAppUpdate](https://github.com/cundong/SmartAppUpdates)）的项目，它已经把bspatch的源码加入到jni内了。只要下载它并编译，就可以在应用内嵌入bspatch，实现增量更新了。

 配置好NDK，在SmartAppUpdates的目录内执行ndk-build：

 ```bash
\ApkPatchLibrary\app\src\main\jni>ndk-build
[arm64-v8a] Compile        : ApkPatchLibrary <= com_cundong_utils_PatchUtils.c
[arm64-v8a] SharedLibrary  : libApkPatchLibrary.so
[arm64-v8a] Install        : libApkPatchLibrary.so => libs/arm64-v8a/libApkPatchLibrary.so
[x86_64] Compile        : ApkPatchLibrary <= com_cundong_utils_PatchUtils.c
[x86_64] SharedLibrary  : libApkPatchLibrary.so
[x86_64] Install        : libApkPatchLibrary.so => libs/x86_64/libApkPatchLibrary.so
[mips64] Compile        : ApkPatchLibrary <= com_cundong_utils_PatchUtils.c
[mips64] SharedLibrary  : libApkPatchLibrary.so
[mips64] Install        : libApkPatchLibrary.so => libs/mips64/libApkPatchLibrary.so
[armeabi-v7a] Compile thumb  : ApkPatchLibrary <= com_cundong_utils_PatchUtils.c
[armeabi-v7a] SharedLibrary  : libApkPatchLibrary.so
[armeabi-v7a] Install        : libApkPatchLibrary.so => libs/armeabi-v7a/libApkPatchLibrary.so
[armeabi] Compile thumb  : ApkPatchLibrary <= com_cundong_utils_PatchUtils.c
[armeabi] SharedLibrary  : libApkPatchLibrary.so
[armeabi] Install        : libApkPatchLibrary.so => libs/armeabi/libApkPatchLibrary.so
[x86] Compile        : ApkPatchLibrary <= com_cundong_utils_PatchUtils.c
[x86] SharedLibrary  : libApkPatchLibrary.so
[x86] Install        : libApkPatchLibrary.so => libs/x86/libApkPatchLibrary.so
[mips] Compile        : ApkPatchLibrary <= com_cundong_utils_PatchUtils.c
[mips] SharedLibrary  : libApkPatchLibrary.so
[mips] Install        : libApkPatchLibrary.so => libs/mips/libApkPatchLibrary.so

\ApkPatchLibrary\app\src\main\jni>
 ```

 到这一步，我们可以得到名为libApkPatchLibrary.so的库，通过com.cundong.utils.PatchUtils模块来调用。

 ```java
public class PatchUtils {

	/**
	 * native方法 使用路径为oldApkPath的apk与路径为patchPath的补丁包，合成新的apk，并存储于newApkPath
	 * 
	 * 返回：0，说明操作成功
	 * 
	 * @param oldApkPath 示例:/sdcard/old.apk
	 * @param newApkPath 示例:/sdcard/new.apk
	 * @param patchPath  示例:/sdcard/xx.patch
	 * @return
	 */
	public static native int patch(String oldApkPath, String newApkPath,
			String patchPath);
}
 ```

 被调用的相应jni代码为

 ```c
/*
 * Class:     com_cundong_utils_PatchUtils
 * Method:    patch
 * Signature: (Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)I
 */
JNIEXPORT jint JNICALL Java_com_cundong_utils_PatchUtils_patch(JNIEnv *env,
		jobject obj, jstring old, jstring new, jstring patch) {

	char * ch[4];
	ch[0] = "bspatch";
	ch[1] = (char*) ((*env)->GetStringUTFChars(env, old, 0));
	ch[2] = (char*) ((*env)->GetStringUTFChars(env, new, 0));
	ch[3] = (char*) ((*env)->GetStringUTFChars(env, patch, 0));

	__android_log_print(ANDROID_LOG_INFO, "ApkPatchLibrary", "old = %s ", ch[1]);
	__android_log_print(ANDROID_LOG_INFO, "ApkPatchLibrary", "new = %s ", ch[2]);
	__android_log_print(ANDROID_LOG_INFO, "ApkPatchLibrary", "patch = %s ", ch[3]);

	int ret = applypatch(4, ch);

	__android_log_print(ANDROID_LOG_INFO, "ApkPatchLibrary", "applypatch result = %d ", ret);

	(*env)->ReleaseStringUTFChars(env, old, ch[1]);
	(*env)->ReleaseStringUTFChars(env, new, ch[2]);
	(*env)->ReleaseStringUTFChars(env, patch, ch[3]);

	return ret;
}
 ```

 其实applypatch函数就是bspatch代码的main函数，它这里改了个名字。

 现在，我们在SmartAppUpdates的Demo项目中测试加入App中的bspatch代码是否能够正常工作：
 把刚才的old.apk和patch包拷贝到手机的/sdcard/目录，并将如下代码加到App的onCreate()方法内。
 ```java
        PatchUtils.patch(Environment.getExternalStorageDirectory().getPath() + "/old.apk",
                Environment.getExternalStorageDirectory().getPath() + "/out.apk",
                Environment.getExternalStorageDirectory().getPath() + "/patch");
 ```

 运行APP后，按照预期/sdcard/目录会生成out.apk。通过adb shell在手机内执行命令：

 ```bash
/sdcard $ md5sum out.apk
cbb1afdbc32e4d1c62c4d38674a6a3a9  out.apk
 ```

 这里可以看到在app程序内嵌入的bspatch也成功的通过了老apk和patch包生成了新的apk，生成的out.apk的MD5值与new.apk是一致的。测试通过。

3. 自我更新
 新建一个Android项目，名字叫PatchDemo。它可以通过patch包实现自我更新。代码为：

 ```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                patch();
            }
        });
    }

    private void patch() {
        String outPath = Environment.getExternalStorageDirectory().getPath() + "/new.apk";
        if (PatchUtils.patch(this, Environment.getExternalStorageDirectory().getPath() + "/update.patch", outPath)) {
            Toast.makeText(this, "updating !", Toast.LENGTH_LONG).show();
            startActivity(new Intent(Intent.ACTION_VIEW).setDataAndType(Uri.fromFile(new File(outPath)),
                    "application/vnd.android.package-archive"));
        } else {
            Toast.makeText(this, "cannot update", Toast.LENGTH_LONG).show();
        }
    }
}
 ```

 PatchUtils代码改为：
 ```java
public class PatchUtils {
    static {
        System.loadLibrary("ApkPatchLibrary");
    }

    /**
     * native方法 使用路径为oldApkPath的apk与路径为patchPath的补丁包，合成新的apk，并存储于newApkPath
     * <p>
     * 返回：0，说明操作成功
     *
     * @param oldApkPath 示例:/sdcard/old.apk
     * @param newApkPath 示例:/sdcard/new.apk
     * @param patchPath  示例:/sdcard/xx.patch
     * @return
     */
    public static native int patch(String oldApkPath, String newApkPath,
                                   String patchPath);

    /**
     * 从patch更新自身，生成到newApkPath
     *
     * @param context
     * @param patchPath
     * @return 成功返回true
     */
    public static boolean patch(Context context, String patchPath, String newApkPath) {
        return 0 == patch(context.getPackageResourcePath(), newApkPath, patchPath) && new File(newApkPath).exists();
    }
}
 ```

 在layout上放一个按钮，点击按钮会调用patch过程。从/sdcard/update.patch路径读取patch包，生成new.apk，然后调用安装。

 ```java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.administrator.patchdemo.MainActivity">

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="Button" />
</RelativeLayout>
 ```

 编译生成app-debug-old.apk，然后修改 android:text="Button" 为 android:text="NEW Button"， 再次编译生成 app-debug-new.apk。

 使用 bsdiff 生成patch包，

 ```java
➜  bsdiff app-debug-old.apk app-debug-new.apk update.patch
 ```

 将update.patch放到/sdcard路径，安装app-debug-old.apk，运行APP，点击button按钮。若一切正常，会弹出APP安装界面，安装后运行APP发现Button按钮的标题已经从Button修改成NEW Button了。

4. ****这样就结束了吗？Naive!****

 你可以尝试不放入update.patch文件，会发现点击button的时候应用直接没了。因为bspatch工具是个命令行工具，它对文件未找到的异常的处理就是直接exit()把进程终结。 更新个增量包，自己先退出了，这样搞肯定是不行的，所以还需要把SmartAppUpdates项目的c代码中bspatch乱结束进程的代码给改了。至于怎么改代码就见仁见智了。

# 总结
 增量更新这个东西原理很简单，效果也很不错，节省流量也十分方便，而且部署到项目内也不困难，只是需要用户手动点击安装（非Root），做好足够的测试就可以在项目中运用它了。
