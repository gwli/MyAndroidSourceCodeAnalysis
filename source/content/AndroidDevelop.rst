nvidia resource
===============

#. `how to build android <https://wiki.nvidia.com/wmpwiki/index.php/V:Booting/JellyBean_Cardhu_rel-16>`_   %RED% on 12.04. when installing build prerequits.  there is missing linux lib  linux-libc-dev:i386 .     libc6-dev depends on this.  
 #. | mobile tools | https://wiki.nvidia.com/wmpwiki/index.php/Tools | 
   OS image build  , https://wiki.nvidia.com/wmpwiki/index.php/V:Booting,
%RED% some cmd there is sudo, but I use sudo -i, so some config will be put on the /root, So I should reconfig ssh, and gitconfig %ENDCOLOR%
#. `nvidia mobile-development zone <https://developer.nvidia.com/category/zone/mobile-development>`_ 
#. `修改android_native_app_glue,简化android的纯C/C++编程 <http://www.klayge.org/2011/12/13/%E4%BF%AE%E6%94%B9android_native_app_glue%EF%BC%8C%E7%AE%80%E5%8C%96android%E7%9A%84%E7%BA%AFcc%E7%BC%96%E7%A8%8B/>`_  通过修改程序入口点来实现的。
current issues
==============

   
.. ::
 
    in Eclipse, you set which debugerserver you use: system, apk bundle, user path, you can find at debug configration for debugger page.
    gdbserver is in /system/bin/  by default. you can start it by manuallly. for example:
    gdb --attach 8000  pid
   

   
.. ::
 
   C:\NVPACK\OpenCV-2.4.2-Tegra-sdk\apk>adb
   C:\NVPACK\OpenCV-2.4.2-Tegra-sdk\apk>adb install OpenCV_2.4.2_binary_pack_tegra3.apk
   980 KB/s (4584082 bytes in 4.567s)
           pkg: /data/local/tmp/OpenCV_2.4.2_binary_pack_tegra3.apk
   Success
   
   C:\NVPACK\OpenCV-2.4.2-Tegra-sdk\apk>adb install OpenCV_2.4.2_manager.apk
   665 KB/s (384443 bytes in 0.564s)
           pkg: /data/local/tmp/OpenCV_2.4.2_manager.apk
   Success
   C:\NVPACK\OpenCV-2.4.2-Tegra-sdk\apk>
   

这是因为没有生成对应lib库，或者lib库的名字不对，lib库的名字是通过项目属性上的lib name. 纯NDK的开发，则是google通过glue把C++转化java代码执行。
   
.. ::
 
   Shutting down VM
   W/dalvikvm( 9870): threadid=1: thread exiting with uncaught exception (group=0x40ad0930)
   E/AndroidRuntime( 9870): FATAL EXCEPTION: main
   E/AndroidRuntime( 9870): java.lang.RuntimeException: Unable to start activity ComponentInfo{com.nvidia.InstancedTessellation/android.app.NativeActivity}: java.lang.IllegalArgumentException: Unable to find native library: instancedtessellation
   E/AndroidRuntime( 9870):        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2180)
   E/AndroidRuntime( 9870):        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2230)
   E/AndroidRuntime( 9870):        at android.app.ActivityThread.access$600(ActivityThread.java:141)
   E/AndroidRuntime( 9870):        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1234)
   E/AndroidRuntime( 9870):        at android.os.Handler.dispatchMessage(Handler.java:99)
   E/AndroidRuntime( 9870):        at android.os.Looper.loop(Looper.java:137)
   E/AndroidRuntime( 9870):        at android.app.ActivityThread.main(ActivityThread.java:5039)
   E/AndroidRuntime( 9870):        at java.lang.reflect.Method.invokeNative(Native Method)
   E/AndroidRuntime( 9870):        at java.lang.reflect.Method.invoke(Method.java:511)
   E/AndroidRuntime( 9870):        at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:793)
   E/AndroidRuntime( 9870):        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:560)
   E/AndroidRuntime( 9870):        at dalvik.system.NativeStart.main(Native Method)
   E/AndroidRuntime( 9870): Caused by: java.lang.IllegalArgumentException: Unable to find native library: instancedtessellation
   E/AndroidRuntime( 9870):        at android.app.NativeActivity.onCreate(NativeActivity.java:181)
   E/AndroidRuntime( 9870):        at android.app.Activity.performCreate(Activity.java:5104)
   E/AndroidRuntime( 9870):        at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1080)
   E/AndroidRuntime( 9870):        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2144)
   E/AndroidRuntime( 9870):        ... 11 more
   W/ActivityManager(  513):   Force finishing activity com.nvidia.InstancedTessellation/android.app.NativeActivity
   
   



   
.. ::
 
   $NDKROOT  ---> C:\NVPACK\android-ndk-r8b
   build command :   $NDKROOT/ndk-build
   build path:  user_apps,victmy_app/jni/
   this path can be retrieved by properties, you can find command line on this page.
   



.. csv-table:: 

   `NDK <AndroidNativeDevelopmentKit>`_  , VisualStudioIDE ,


 包安装分析
================

 `android-package <http://benno.id.au/blog/2007/11/18/android-package>`_  分析了，apk是安装在/data/app/  下面，安装好之后都会在/data/data/有对应的目录，并且其lib都是链接到了/data/app-lib下面，/data/app-asec 是放经过加密的密钥的。当然有些软件也会自定义自己的位置。并且生成的那些.dex是放在/data/dalvik-cache/下面的。    就像`dexformat.html <http://www.retrodev.com/android/dexformat.html>`_  能够反向工程，并且能够猜想验证这样的东东才叫study,而自己只是搜索一下现成的而己。而这种猜想加验证才是反向工程与hack的真正水平。例如直接从十六进制从中字符串来猜测其结构。例如cuda的反向功能就是这样进行的，只输入一条指令，然后看其输出指令，这样来看其二进制。常用编码方式也就那几种，再加上大小字节序的问题，都是可以有办法猜想验证的。

.. csv-table:: 

   data/app-private ,  3rd-party protected aps ,
   /system/app , System apps that come pre-installed with Rom ,
   data/system/packages.list   , package list and path ,
   data/system/packages.xml ,  配制信息 , 

android SDK
===========

android 采用类似于微内核的方式，都所有东西都是最终android自身来调度，android的虚机就是那个核，所有界面，后台，以及进程之间通信都都是在各在队列堆栈中，例如activity就是一个栈活动，至于是完全有操作系统来决定，每个activity只是设置一下自己的部局，所有需要的动作事件，自己实现其接口就行了，并且对于资源本身的调用也都进行优话，尽可能的动态可配置。activity,就相当于form,而view就是过去的控件，并且在java中对于线程的使用，直接实现runable接口就可以了。

intent，就是数据与action的结合体，并且各个intent还要进行分组分类，例如home,launcher,default,main,其实intent就像一个消息队列，就像Qos往不同的队列里放回调函数，并且采用最长的匹配的方法来进行查找。 

android 继承了linux的特点，所有底层应用程序都是有命令行的，你可以直接用命令来驱动，基本可以控制应用程序的方方面面。这个是与windows程序的最大的不同。所以linux下的大部分程序都可以当做管道进行复用的。并且android的开发并且有点像linux的驱动开发，但是把那些东西简化了，利用一个XML把所有的一切注册与管理起来了。所以它有两种机制，一种是通过代码直接执行，而另一种那就是通过XML注册出来消息循环机制，来进行自己消息调度算法，就像MFC中的那消息的循环一样。

APK本身就是一个.zip的压缩包。你可以利用一般的工具来打开的。并且它的结构也很简单，就像apt-get 包一样，你可以使用任意解压软件去打开看，无非三部分，manifest来配制应用程序，例如消息的处理等等，asset来存放数据，一般游戏的数据会放在里，lib会存放本地需要的库，res存放字符串值，以及GUI的布局。MEFA-INF来存放签名验证等等。以及Jar包在那里。

并且其gen,src,res,asset目录都是可以指定在利用ANT，在命令行宏，或者在PENTAK中来指定。
Workflow
========

   
.. ::
 
   ##SDK bootup to activity
   	com.example.hellojni.HelloJni.onCreate() Line 39	Java
    	android.app.Activity.performCreate() Line 5104	Java
    	android.app.Instrumentation.callActivityOnCreate() Line 1080	Java
    	android.app.ActivityThread.performLaunchActivity() Line 2144	Java
    	android.app.ActivityThread.handleLaunchActivity() Line 2230	Java
    	android.app.ActivityThread.access$600() Line 141	Java
    	android.app.ActivityThread$H.handleMessage() Line 1234	Java
    	android.os.Handler.dispatchMessage() Line 99	Java
    	android.os.Looper.loop() Line 137	Java
    	android.app.ActivityThread.main() Line 5039	Java
    	java.lang.reflect.Method.invokeNative()	Java
    	java.lang.reflect.Method.invoke() Line 511	Java
    	com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run() Line 793	Java
    	com.android.internal.os.ZygoteInit.main() Line 560	Java
    	dalvik.system.NativeStart.main()	Java
   ###########
   # NDK
   ###########
          [0x67A703DC] Java_com_example_hellojni_HelloJni_stringFromJNI(JNIEnv * env, jobject thiz) Line 12	C++
    	[0x4085E794] libdvm.so!dvmPlatformInvoke()	C++
    	[0x4088D924] libdvm.so!dvmCallJNIMethod(unsigned int const*, JValue*, Method const*, Thread*)()	C++
    	[0x4088FA80] libdvm.so!dvmResolveNativeMethod(unsigned int const*, JValue*, Method const*, Thread*)()	C++
    	[0x40867BE8] libdvm.so!dvmJitToInterpNoChain()	C++
    	[0x40867BE8] libdvm.so!dvmJitToInterpNoChain()	C++
   ##########
   # NDK static load C++lib
   ##########
   	com.example.hellojni.HelloJni.<clinit>() Line 87	Java
    	java.lang.Class.newInstanceImpl()	Java
    	java.lang.Class.newInstance() Line 1319	Java
    	android.app.Instrumentation.newActivity() Line 1054	Java
    	android.app.ActivityThread.performLaunchActivity() Line 2097	Java
    	android.app.ActivityThread.handleLaunchActivity() Line 2230	Java
    	android.app.ActivityThread.access$600() Line 141	Java
    	android.app.ActivityThread$H.handleMessage() Line 1234	Java
    	android.os.Handler.dispatchMessage() Line 99	Java
    	android.os.Looper.loop() Line 137	Java
    	android.app.ActivityThread.main() Line 5039	Java
    	java.lang.reflect.Method.invokeNative()	Java
    	java.lang.reflect.Method.invoke() Line 511	Java
    	com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run() Line 793	Java
    	com.android.internal.os.ZygoteInit.main() Line 560	Java
    	dalvik.system.NativeStart.main()	Java
   
   
   


 *Android的命令行开发*
建立工程，更新，以及建立sdk,avd,等等都通过 android这个脚本来进行。所有的命令工具都在[android-SDK-XXXX]/tools下。整个过程如下，eclipse 可以直接使用ant 来进行管理，但如果C/C++的这种NDK 开发，就要用makefile了，如果pentaK 来做，那就要用到，来管理JAVA这一套的话，就是用到build文件了。
#. `Android开发片段–命令行安装，卸载，启动，程序（AM,PM） <http://hi.baidu.com/liujianzhang85/item/f3ffdde0e70ea8326dabb8db>`_  maybe I can use this.
#. ` Android系列之Android 命令行手动编译打包详解 <http://www.cnblogs.com/jk1001/archive/2010/08/05/1793216.html>`_  this site is resourceful for android. I can visit this frequently. 
#. `android managing Projects from the command line <http://developer.android.com/tools/projects/projects-cmdline.html>`_  
#. `AIDL,Android Interface Definition Language <http://developer.android.com/guide/components/aidl.html>`_  提供给其他应用程序的接口定义,可以用在进程间通信，就像RPC一样。

android 源码分析
====================

源码编译后产生三个文件，ramdisk.img,system.img, userdata.img 这个三个文件都是用gzip与cpio生成的，同样也是可以用它们来打开这些系统，并挂截文件系统，`方法见此 <http://ibbs.91.com/thread-68601-1-1.html>`_ , system,与usredata 分别挂载在/system,/userdata形成的。
#. `android filesystem  <http://hi.baidu.com/zhupan19851230/item/057470f4227442c6531c26d4>`_   根据分类以及需要来进行修改，并且进行裁减，其实就是内核裁减的过程。

直接通过看源码编译好的文件看看下面都有什么功能，并且如何才能把bash这些东西都移植到android上去。这样还可以让自己有点声望。 同时要把`为Android安装BusyBox —— 完整的bash shell <http://www.cnblogs.com/xiaowenji/archive/2011/03/12/1982309.html>`_  把一些功能强大的东西都移植上来。   
#. `android-scripting <http://code.google.com/p/android-scripting/>`_  use various script perl/tcl on android.
 
How to remote debug java and native code
========================================

   *`JDWP 命令调试 <http://www.apkbus.com/android-3455-1-1.html>`_  想要到在java代码中断到什么位置，那就要设置dalvikvm -agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=y -cp /sdcard/foo.jar Foo 的启动参数。每一个apk都要启动一个dalvikvm.  `Controlling the Embedded VM <http://www.netmite.com/android/mydroid/dalvik/docs/embedded-vm-control.html>`_ 
   
.. ::
 
   on target:
   
   am start -D -n com.example.hellojni/.HelloJni android.intent.category.LAUNCHER
   am start -D -n com.nvidia.devtech.hdrdemo/com.nvidia.devtech.NativeHDR.NativeHDR android.intent.category.LAUNCHER
   gdbserver :1234 --attach PID
   
   on host:
   adb forward tcp:1234 tcp:1234
   gdb app_process
   set solib-search-path        // 这里指的是你的obj/local/armebi/  远程调试的程序可以没有符号表，但是本地的必须有。它们具有相同地址，gdb可以通过这些进行查询。
   target remote :1234
   
   adb forward tcp:1235 jdwp:PID
   jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=1235
   
   
   run-as $PACKAGE_NAME    lib/gdbserver   +$DEBUG_SOCKET --attach $PID
   
   adb forward tcp:$DEBUG_PORT localfilesystem:$DATA_DIR/$DEBUG_SOCKET
   socket could use like this.
   
   run-as 相当于linux中runuser命令。
   
   

 
Application analysis
====================

 there are many android example :*//sw/devrel/Handheld/playpen/lbishop/android/NvCameraBurstExample/*

.. csv-table:: 

   NewtonScradle ,  Kwattk3Study ,  NativeGlobe , NvCameraBurstExample , `cocos2d-x <Cocos2Dx>`_  ,  `EBook <ElectronicBook>`_   ,     
   ` ogre3D  <OgreAndroid>`_  ,  `libpng <LibPng>`_  , `bullet physics  <http://bulletphysics.org/wordpress/>`_  , `Unreal <UnrealEngine>`_  ,  ,GlowBall ,
   `大海 <SeaWave>`_  ,  `HDR <HDRSample>`_ 




See also
========

#. `安桌开发入门经典 <http://wenku.baidu.com/view/a30efd1d59eef8c75fbfb36a.html>`_  

#. ` Android-Terminal-Emulator <http://www.oschina.net/p/android-terminal-emulator>`_  how to use the linux
#. `command development for android <http://blog.csdn.net/feifei454498130/article/details/6638276>`_  adb
#. `老贾的博客 <http://blog.csdn.net/blogercn/article/details/7449091>`_  
#. `手机UI自动化测试工具NativeDriver VS Robotium <http://www.51testing.com/?uid-116228-action-viewspace-itemid-241063>`_  这个做者是谁
#. ` Android开发之旅：android架构 <http://www.cnblogs.com/skynet/archive/2010/04/15/1712924.html>`_  框架图

#. `Android 根文件系统分析 <http://www.hzlitai.com.cn/article/ARM11/SYSTEM/1756.html>`_  similar with the linux, just do some simplification.
#. `android developer offical web <http://developer.android.com/index.html>`_  
#. `Android内核剖析 <http://www1.huachu.com.cn/read/readbook.asp?bookid&#61;1000001806>`_  quickly read it
#. `manifest for Java lib <https://github.com/cocos2d/cocos2d-x/blob/master/cocos2dx/platform/android/java/AndroidManifest.xml>`_  
#. `adt official web  <http://tools.android.com/download/adt-20-preview>`_  
#. `TextView usage sample <http://vaero.blog.51cto.com/4350852/782595>`_  
#. `解析Android消息处理机制：Handler/Thread/Looper &#38; MessageQueue <http://blog.csdn.net/thl789/article/details/6601558>`_  
#. `Android View.post(Runnable ) <http://www.cnblogs.com/akira90/archive/2013/03/06/2946740.html>`_  
#. `Android系统进程Zygote启动过程的源代码分析 <http://blog.csdn.net/luoshengyang/article/details/6768304>`_   Zygote use /system/bin/app_process

#. `genymotion <http://www.genymotion.com/features/>`_  采用虚拟机做的仿真器，速度超快
#. `android的NPK格式与APK格式文件的区别 <http://zhidao.baidu.com/question/209125243.html>`_  
#. `proguard <http://developer.android.com/tools/help/proguard.html>`_  protect from revise engine

thinking
========



*android testing*
Android testing is using injection technique to test, and the google will give an unique signature for each application for secruity. 

for the testing and debug, there is open debug signature which is same around the world. 

currently, Most of the application on Android can be delete the signature except the QQ which has add the antispy feature.

-- Main.GangweiLi - 21 Oct 2012


*`StackFrame <http://www.oschina.net/code/explore/android-2.2-froyo/com/android/hit/StackFrame.java>`_  the model of stack.  calling the function is based on stack. so stack model is important. the stack of java for android is like the tcl stackframe.

-- Main.GangweiLi - 29 Oct 2012


*sign of APK*
property, Ant-builder中要配置那些keystore之类东西就是天于签名的。其实就是`数字签名 <Work.DigitalSignature>`_ 

-- Main.GangweiLi - 08 Feb 2013


default android sdk use own build.xml to build app. when you need build, you need android update project  manually.

-- Main.GangweiLi - 27 Feb 2013


*apk default location*
/data/local/tmp/, it seems that there is no home direcotory for android. instead of it, the home dir should be /data. the default dir of *cd* will to go to */data/*. 
there is no file browser in android. and there is multi-user directory currently.

-- Main.GangweiLi - 04 Mar 2013


*how to decompile apk* you can search apk-tool to do this.http://www.cnblogs.com/lovelili/archive/2011/10/19/2217259.html

-- Main.GangweiLi - 25 Sep 2013



-- Main.GangweiLi - 14 Oct 2013


   *android*是用的linux内核，所以底层文件系统以及.ko什么都是与linux一样的。并且对于对于也都支持mount文件系统的命令。例如在perhes 中会使用
.. ::
 
   adb shell su -c "mount -o remout,rw /dev/block/platform/sdhci-tegra.3/by-name/APP /system" 
   并且修改已经打开的文件，系统就会检测到，并且已经修改，就会发生产生一个中断，来更新内容。
   
   并且正是由于计算机的这种分层透明性，所以出了问题，你不能直接说明他们之间的原由，特别是当发现不可思议的事情发生时，就像quadD与安装与perfHues放在一个块，并且perfHes引起了OS的异常，而又OS异常导致了它不能工作。
   


-- Main.GangweiLi - 29 Oct 2013


*安卓APP的机制主要是类与类之间的通信主要靠intent.

-- Main.GangweiLi - 20 Mar 2014

