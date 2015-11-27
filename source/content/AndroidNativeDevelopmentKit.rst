Introduction
============

`JNI Tips <http://developer.android.com/training/articles/perf-jni.html>`_  依赖两个数据结构来传递信息，JavaVM,与JavaEnv,并且JavaEnv是线程内部，不能共享，欲共享则要JavaVM.并且还要绑定线程。`JNI protocol  <http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html>`_ 
`utf8-utf16 <http://zhidao.baidu.com/question/15626866>`_ 如果只是简单的用java call C/C++大部分工作就是利用env为接口来传递。如果想C/C++调用java间接用到了JavaVM.要用java就必须有javaVM.这个过程中两个是同时存在的。
NDK开发，通过封装把JAVA的所有消息传递都通过多线程与利用unix的管道与同步来解决。Java端使用android.app.nativeActivity. 同时也利用app_glue把消息的同步的机制给做好。因为Java本身就是多线程，NDK把所有杂事都进行转接到C/C++中，当然也可以自己实现glue这些同步的框架。例如消息的对队ALooper.事件，以及各种输入事件的处理。
JNI 是Java 到c，然后由c到c++的过程。

Hellword 调用native 真实过程
==================================

   
.. ::
 
    	[0x50F283B0] Java_com_example_hellojni_HelloJni_stringFromJNI(JNIEnv * env, jobject thiz) Line 12	C++
    	[0x40842C34] libdvm.so!dvmPlatformInvoke()	C++
    	[0x4087CEE6] libdvm.so!dvmCallJNIMethod(unsigned int const*, JValue*, Method const*, Thread*)()	C++
    	[0x4087EC16] libdvm.so!dvmResolveNativeMethod(unsigned int const*, JValue*, Method const*, Thread*)()	C++
    	[0x40854AD4] libdvm.so!dvmJitToInterpNoChain()	C++
    	[0x40854AD4] libdvm.so!dvmJitToInterpNoChain()	C++
   
   

启动时的调用过程
   
.. ::
 
   	 [0x5B5C0E40] android_main(android_app * app) Line 38	C++
    	[0x5B5C1F18] android_app_entry(void * param) Line 334	C++
    	[0x40103E70] libc.so!__thread_entry()	C++
    	[0x401039C4] libc.so!pthread_create()	C++
   

但是是谁调用了，pthread_create呢，在代码中_android_app_create

编译控制
========

#. `Application.mk与android.mk  <http://blog.csdn.net/weidawei0609/article/details/6561280>`_  

.. csv-table:: 

   android.mk ,主要控制 输入，输出 -I 以及源代码的关系 ,  ,
   application.mk ,  控制编译选项 , 例如 APP_PLATFORM=android-14 , APP_STL :=gnustl_static CPP_CFLAGS +=-Wno-error=format-security ,

   
See also
========

#. `Android.mk的用法和基础  <http://blog.csdn.net/zhandoushi1982/article/details/5316669>`_  
#. `安卓NDK综述 <http://wenku.baidu.com/view/750abfdcad51f01dc281f177.html>`_  
#. `ABI <http://zh.wikipedia.org/wiki/&#37;E5&#37;BA&#37;94&#37;E7&#37;94&#37;A8&#37;E4&#37;BA&#37;8C&#37;E8&#37;BF&#37;9B&#37;E5&#37;88&#37;B6&#37;E6&#37;8E&#37;A5&#37;E5&#37;8F&#37;A3>`_  application binary interface
#. `Android NDK offical web <http://developer.android.com/tools/sdk/ndk/index.html>`_  
#. `NativeActivity <http://developer.android.com/reference/android/app/NativeActivity.html>`_  
#. `The project either has no target set or the target is invalid. Please provide a --target to the &#39;android update&#39; command. <http://hi.baidu.com/dreamflyman/item/b1f04211e432378d88a956ab>`_  
#. `native&#95;app&#95;glue 工程完全注解  <http://wzhnsc.blogspot.com/2011/10/android-ndk-r5bsourcesandroidnativeappg.html>`_  
#. `Linux消息队列分析及应用 <http://wenku.baidu.com/view/8f71544c852458fb770b56ad.html>`_  
#. `android JNI学习之五 JNI中常用的方法  <http://lipeng88213.iteye.com/blog/1292570>`_  

#. `jniexport-and-jnicall-in-android-ndk <http://stackoverflow.com/questions/8629495/jniexport-and-jnicall-in-android-ndk>`_  these two is just Macro, dllexport,&#95;&#95;stdcall

Thinking
========



*System.loadLibrary()*
A way to invoke native method is to invoke System.LoadLibrary in a static initialization block. 并且是先声名，然后由Javah生成头文件，然后自己根据头文件写C的实现。所有java 能调用库接口是规范的，例如第一个参数都是envVar. 其实与`SWIG <http://www.swig.org/>`_ 都是一样的。然后把库放在特定的目录。并且用android.mk来推动。ndk-build 会来根据android.mk 来操作。 ndk-build 就是把make封装了一下。



并且 android update project 来生成build.xml 文件。并且谁先build,这里有一个builder,然后你可以配置builderchain. 所谓的那个toolchain应该是在这里配置的吧。 另外一点那就是MS-BUILD转化NDK-Build.

-- Main.GangweiLi - 05 Feb 2013


*NDK-GDB*
`NDK-GDB <http://www.cnblogs.com/yaozhongxiao/archive/2012/03/13/2393959.html>`_ 其实是一段脚本同时能够操作jdb与gdb来进行调试操作。而pentaK利用了NDK-GDB来简化的调试。

-- Main.GangweiLi - 08 Feb 2013


*Android.app.NativeActivity*
Android 提供了一个NativeActivity类然后再用JNI接口，把所有的消息驱动通过native_app_glue给转接过来。

ANativeActivity 是一个结构体，里面有callback,callback里把所有的onStart,onResume等提供了实现接口.  并且android_app_write_cmd,把所有java面事件，都通过JNI做一个中转给native. Main UI thread and native thread communicates via Unix pipes and mutexes to ensure proper synchronization.

-- Main.GangweiLi - 18 Feb 2013


NDK开发一个重要功能，就是把现有程序都搬到android上来。这些主依赖于NDK这种交叉编译来实现的。主要也就是移值各种库。主要是本地的android.mk的文件，然后再appliation.mk的编写。而java则是利用ant 来进行的。另外一个问题那就是能否利用maven来进行NDK的开发管理。
-- Main.GangweiLi - 20 Feb 2013


并且 一旦回到了NDK，所有的知识都又回到了再OPENGL.

-- Main.GangweiLi - 20 Feb 2013


*ndk-build*  如果出现找不到库的，就是因为编译当前的路径不对。
ndk-build -C 首先切换路径。  ndk-build就是一个make,所以你要在当前有application.mk,android.mk文件目录进行。  并且要能够手工驱动这些命令进行debug.

-- Main.GangweiLi - 27 Feb 2013


call the java lib from native development.
https://github.com/cocos2d/cocos2d-x/blob/master/cocos2dx/platform/android/java/AndroidManifest.xml

-- Main.GangweiLi - 05 Mar 2013


*download the old r8e*
download the latest url, and then change the r9 to r8e

-- Main.GangweiLi - 29 Jul 2013


ant 需要SDK 环境变量,是通过再android.bat
./lib/build.template:    <condition property="sdk.dir" value="${env.ANDROID_HOME}">
 
ndk-build 是需要环境变量，$NDKROOT,

.. csv-table:: 

   JAVA_HOME , ./templates/gradle/wrapper/gradlew.bat:echo Please set the JAVA_HOME variable in your environment to match the ,
   ANDROID_NDK_ROOT , ./build/tools/gen-platforms.sh:NDK_DIR=$ANDROID_NDK_ROOT,
    NDK_ROOT , ^ ,



-- Main.GangweiLi - 13 Nov 2013


*对于各种功能开发*
看android系统中支持这个函数与否，直接在platforms直接搜头文件就知道支持与不支持了。 并且今天也看ndk自带的doc文档。以后要经常去这些文档。并且这个下面有现成各种各样的tool脚本，例如生成gcc的脚本，是交叉编译的很好的自定义工具平台。同时把ndk-gdb-py也看了一下，知道了其是何实现管理的，直接使用socket来做重定向，所以使用NC应该也可以的。例如使用NC 直接把6000端口映射到:0就可以解决x window不监听tcp的问题了。或者使用inetd来做重定向。


-- Main.GangweiLi - 12 Dec 2013


renderscript   google又准备自己做一套graphic的system,了，这个功能类似于cuda,采用C99的语法。可以直接开发graphic.

-- Main.GangweiLi - 12 Dec 2013


standlone toolchain
======================
 

there is sample

#. Initalize standlone g++  

   .. code-block:: bash

      ##windows### bash ${installdir}/${android_ndk_dirname}/build/tools/make-standalone-toolchain.sh --platform=android-14 --system=linux-x86_64 --install-dir=${installdir}/${android_ndk_dirname}/toolchains/arm-linux-androideabi-4.6/gen_standalone/linux-x86_64
      ## linux ### $NDK/build/tools/make-standalone-toolchain.sh --toolchain=arm-linux-androideabi-4.9  --system=linux-x86_64 --platform=android-21 --install-dir=$PWD/android-toolchain  

#. build an test app 
   
   :command:`arm-linux-androideabi-g++ -O0 -ggdb2 hello-except.cpp -fPIE -pie -o hello-except`

   .. code-block:: c

      #include <stdlib.h> // exit()
      #include <string>
      #include <iostream>
      
      class GreetingException
      {
        private:
          std::string _text;
      
        public:
          GreetingException(char const* text) { _text = text; }
          std::string Text() const { return _text; }
      };
      
      
      std::string construct_greeting(const std::string& name)
      {
        if (name == "ryan")
          throw GreetingException("Get lost, kid!!!!!!");
      
        std::string salutation = "Hello, ";
      
        salutation = "Hello, " + name + "!";
      
        return salutation;
      
      }
      
      
      int main(int argc, char** argv)
      {
        std::string name;
        std::string salutation;
      
        if (argc == 2)
          name = argv[1];
        else
          name = "World";
      
        try
        {
          salutation = construct_greeting(name);
        }
        catch (const GreetingException& e)
        {
          std::cout << e.Text() << std::endl;
          exit(-1);
        }
      
        std::cout << salutation << std::endl;
        exit(0);
      }

#. push the app
   
   :command:`adb shell gdbserver:5039 /system/bin/hello-except lgw`

#. set up the session:

   .. code-block:: bash
      
      adb forword tcp:5039 tcp:5039
      arm-linux-androideabi-gdb ./hello-except
      (gdb) target remote :5039
