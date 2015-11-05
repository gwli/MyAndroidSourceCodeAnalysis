tools
======

adb 各种命令的使用说明  http://developer.android.com/tools/help/adb.html#logcatoptions

logcat
------

可以清理也可以，把log写到文件中然后退出。

:command:`adb logcat -c` clear the entire log and exist

:command:`adb logcat -f log.txt`  写把log写到文件中。
:command:`adb logcat -f -d log.txt`  写把log写到文件中。


bugreport
---------

:command:`adb bugpreport` 对于收集板子信息非常有用，但是信息具大。


AM and PM
---------

一个是Activity Management, 另一个PackageManagement. 同时我们还可以用aapt直接apk 读到各种信息。

AM 里边也有bugreport功能。收集各种信息。


打开一个app

:command:`adb shell am -n com.nvidia.Bloom/.Bloom` 就打开了。
:command:`adb shell am stop/force-stop com.nvidia.Bloom` 关闭一个apk.

AM可以控制还可以传递参数

:command:`am start -a android.intent.action.MAIN -n  net.yurushao.demo/net.yurushao.demo.ExampleActivity   --ei pid 10 --es str "hello, world"`

或者编程实现 

.. code-block:: java
   
   //sender
   Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
   Bundle b = new Bundle();
   b.putInt("key", 1); //Your id
   intent.putExtras(b); //Put your id to your next Intent
   startActivity(intent);
   finish();

   //receiver
   Bundle b = getIntent().getExtras();
   int value = b.getInt("key");


screenshot
----------

android有三条命令可以用

:command:`screencap screenrecord screenshot` 。

dumpsys
-------

可以用来查看系统的各种状态。
http://stackoverflow.com/questions/11201659/whats-the-android-adb-shell-dumpsys-tool-and-what-are-its-benefits

backup
------

想要弄得root权限，其本质就是换掉boot.img，并且修改selinux,得到root权限之后，remount 成可读可写就行了。

短信，联系人都是放在sqlite 中， 
https://chombium.wordpress.com/2012/09/30/android-how-to-backup-contacts-and-sms-messages/

./adb pull /data/data/com.android.providers.telephony/databases/mmssms.db
./adb pull /data/data/com.android.providers.contacts/databases/contacts2.db

用来恢复的。
./adb shell mount -o remount,rw -t yaffs2 /dev/block/mtdblock3 /system
./adb shell rm /data/data/com.android.providers.telephony/databases/mmssms.db
./adb shell rm /data/data/com.android.providers.contacts/databases/contacts2.db
./adb push mmssms.db /data/data/com.android.providers.telephony/databases/
./adb push ~/Desktop/phone/contacts2.db /data/data/com.android.providers.contacts/databases/


或者直接使用 backup 指令，http://mobilenetworkcomparison.org.uk/tutorials/how-to-backup-or-restore-any-android-phone-with-adb-shell/
Issues
======

#. when connect to the adb, you kill the process of adb and restart again.
#. login as root  *adb root then adb shell*  所有可用命令在/system/bin  以及/system/xbin/ 这个是需要管理员权限的。  `ADB&#95;Shell&#95;Command&#95;Reference <http://en.androidwiki.com/wiki/ADB&#95;Shell&#95;Command&#95;Reference>`_  
      
.. ::
 
      ### these two cmd is mapping to pm install/uninstall
        adb uninstall [-k] <package> - remove this app package from the device
                                   ('-k' means keep the data and cache directories)
        adb install [-l] [-r] [-s] [--algo <algorithm name> --key <hex-encoded key> --iv <hex-encoded iv>] <file>
                                - push this package file to the device and install it
                                  ('-l' means forward-lock the app)
                                  ('-r' means reinstall the app, keeping its data)
                                  ('-s' means install on SD card instead of internal storage)
                                  ('--algo', '--key', and '--iv' mean the file is encrypted already)
      


#. `adb on developer.android.com <http://developer.android.com/tools/help/adb.html>`_    the adb framework is just like the medII. and console is a little like the DPH. 
   #. support port forwording.
   #. support the port 5555,5585. if so only 15 device can be connected. is there any solution.
*adb* most of the time, it set up a connection via USB. so adb just the 3.5 layer which below the TCP above the IP. #. ` ADB remote Debug Android App on G1 （Through WiFi） <http://blog.csdn.net/stevenliyong/article/details/4799774>`_   so adb is using the TCP/IP. so you can connect to the adb by wifi. normally the emulator is using tcp/ip. and but you devices is using USB. there are some options control how to contect to adb. you can use netstat to check if there is 127.0.0.1:5037 this is using USB, if there is 0.0.0.0:5037 you can using IP.
`Android Adb Analyse <http://blog.csdn.net/wbw1985/article/details/5443910>`_  ,既然每个设备连接adb,都需要两个端口，一个为adb所用，而另一个console所用，那么我能否直接到console上，直接用telnet或者nc,如何直接连接到console上，然后直接烧uboot以及image呢。现在的方式是封闭的。硬件的设备管理都是/dev下，并且都是tty相关的东西，如何把自己以前那一些东西给整理出来。

并且adb.exe 与adbwubi.dll就可以直接使用了。 并且adb 通过jdwp来与dalvikvm进行交互。并且有DDMS可以与APK应用程序直接进行交互。例如发个信息，或者发送GPS信息等。
#. *adbd源代码在:/bootable/recovery/minadbd*

   #. the adb deamon works at the android shell just like telnet. 
   #. `Dalvik Debugger Support <http://www.netmite.com/android/mydroid/2.0/dalvik/docs/debugger.html>`_ 
   #. `ddmlib <http://sourceforge.net/apps/trac/android4maven/wiki/ddmlib>`_ 
   #. `DDMS simple introduction <http://my.oschina.net/zhijie/blog/6760>`_ 
`对于VM的调试是通过JDWP来进行的 <http://www.ibm.com/developerworks/cn/java/j-lo-jpda3/>`_ ， debugger 和 target vm。Target vm 中运行着我们希望要调试的程序，它与一般运行的 Java 虚拟机没有什么区别，只是在启动时加载了 Agent JDWP 从而具备了调试功能。而 debugger 就是我们熟知的调试器，它向运行中的 target vm 发送命令来获取 target vm 运行时的状态和控制 Java 程序的执行。Debugger 和 target vm 分别在各自的进程中运行，他们之间的通信协议就是 JDWP。



如何解决adb 看到device没有permission 的问题
-------------------------------------------

改变一个 udev 的rule就可以了。


.. code-block:: bash
   #filename 51-android.rules
   #adb protocol on passion (Tangle)
   SUBSYSTEM=="usb" ATTR{idVendor}=="0955" MODE="0666" GROUP="plugdev"
   SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666"
   SUBSYSTEM=="usb", ATTR{idVendor}=="2717", ATTR{idProduct}=="9039", MODE="0666", OWNER="<username>"


getprop and setprop
===================
https://guardianproject.info/wiki/Android_getprop_collection,每一家都可能不一样。 
#. `Android系统中setprop,getprop,watchprops命令的使用 <http://daimajishu.iteye.com/blog/1086627>`_  初始化的配置文件
getprop 取值，setprop设值，watchprops 监测。 

See also
========

#. `screencast <http://zh.soft-db.com/info/148174/screencast-pro/>`_  
#. `google&#95;android&#95;platform&#95;model <http://www.databaseanswers.org/data&#95;models/google&#95;android/images/google&#95;android&#95;platform&#95;model.gif>`_  
#. `android 不支tab补全以及ctl-c的方法 <http://www.360doc.com/content/10/0506/07/496343&#95;26284405.shtml>`_  


#. `ADB/Fastboot Setup <https://sites.google.com/site/teamroyalsginger/guides-under-development/adb-fastboot-setup>`_  
#. `adb over wifi <http://mehrvarz.github.io/android-debug-sans-usb/>`_  simple just need tcpport restart the daemo again.
   
.. ::
 
   the default port is 5555.
   if you want connected with another port. change it by
   adb tcpip 5555
   


thinking
========


-- Main.GangweiLi - 22 Oct 2012


*IDevice* 定义了一个逻辑设备的接口，这样就把与物理设备隔离开发了，这种实现就会很容易了。既然是物理设备就是要设备信息，以及设备的状态。就是把linux的整个设备都进行了封装。

-- Main.GangweiLi - 28 Oct 2012


*AndroidDebugBridge* 定义了一协议交互的接口，建立了一种连接。自己定义本身的服务器端口，从5037开始。

-- Main.GangweiLi - 28 Oct 2012

