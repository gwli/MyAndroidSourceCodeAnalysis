NSight Tegra
============

对于NSight Tegra来说，其实很简单，一个是编译，一个调试。并且把远程调试都给自动化了。但是做了哪些事情呢。这与我们手工做的效果是一样的。无非是通过代码实现而己。
例如对于设备的管理就是adb 命令，对于设备的属性的得到例如设备名的设置就是直接读取了adb shell getprop的结果，然后这些output 来得到的。


*debug*

如何起动apk,如何得到apk的信息，以前都是通过androidmanifest 来得到的，现在是直接通过*aapt* `aapt dump badging` 来得到，这个是 host端直接分析来得到的。

而对于gdbserver的使用机制是这样的。不在依赖于编译的时候把它给弄进去。当然也可以去读一下编译的pipeline来准备加载的自己的东东。例如unreal.
而是在debug时加截的。
并且现在的不在强制需要gdb.setup,因为debugInfo中本身就有代码的路径，当代码中路径找不到的时候，就会提示你让你输出入sourcecode.并且还会强制把project.outputDirectory设备为 *solib-search-path*
如果在这个位置还有就加载，没有继续执行。默认的情况是与gdbserver放在一起的。


#. Push gdbserver to /data/local/tmp/ and chmod 777 it.
#. If it runs/attaches - use it.
#. Cat-copy gdbserver from /data/local/tmp/ to /data/data/<package>/ and chmod 777 it. 4. If it runs/attaches - use it.
#. Use /data/data/<package>/lib/gdbserver. If it doesn't run/attach - display error message.

.. tip:: "Cat-copy" command is :command:`cat <from> | run-as <package> sh -c 'cat > <to>'`

General Rrule
#. deploy apk use install -r 以及pm clear packagename 
#. 是不是完全一样是采用签名来验证的。
#. push 文件是根据连接类型来的，如果wifi的话，就不pull一些系统的库。
#. 然后打开gdbserver.

debug Manually
--------------
#. launch apk
#. pulling files
#. start gdbserver
#. start jdb debugger
#. start gdb debugger

#. for target
   #. launch app :command:`am start -D -W -n com.nvidia.devtech.hdrdemo/com.nvidia.devtech.NativeHDR.NativeHDR`
   #. get pid by :command:`adb shell ps`
   #. gdbserver attach :command:`adb shell run-as com.nvidia.devtech.hdrdemo lib/gdberver +debug-pipe  --attach <PID>` 
   #. forword gdb port :command:`adb forward tcp:2020 localfilesystem:/data/data/com.nvidia.devtech.hdrdemo/debug-pipe`
   #. forword jdb port :command:`adb forward tcp:4040 jdwp:3923`

#.  host
   #. start gdb 
      #. init gdb
         - source the gdb.setup :command:`-interpreter-exec console \" source ...gdb.setup\"`

         .. code-block:: bash
         
            -gdb-show solib-search-path
            -gdb-set solib-search-path
            -gdb-set can-use-hw-watchpoints 1

   #. attach the jdb :command:`java -classpath {0} -Duser.home=. com.sun.tools.example.debug.tty.TTY -connect com.sun.jdi.SocketAttach:hostname=localhost,port={1} -sourcepath \"{2}\" -mi",classPath, port, sourceFilesDirectory` 



Android NDK 编译流程
====================

.. graphviz::
   
   digraph NDK_compiling {
       CUDA -> NDK_Build -> ANT;
   }


gdb setup process
-----------------


现在的android的也有64与32位之分，所在同步lib时，也像操作系统一直接分为32位与64位。

.. csv-table:: sync file or path
   :header: "os type,name", remark

   32bit, deviceFiles, system/bin/app_process;/system/bin/app_process32;/system/bin/linker;/system/lib/lic.so
   ^,deviceFolders, /system/lib/;/system/vendor/libs
   64bit, deviceFiles, /system/bin/app_process64;/system/bin/linker64;/system/lib/lic.so
   ^,deviceFolders, /system/lib64/;/system/vendor/libs64

.. csv-table:: local lib
   :header: "路径", "说明"
   
   %APPDATA%\Local\Temp\Android\<deviceID>\system32, 32 bit lib
   %APPDATA%\Local\Temp\Android\<deviceID>\system64, 64 bit lib
   
.. note::

:file:`NVIDIA_Nsight_Tegra_Release_2.0.0.14342.exe` 之前是把32位与64位放在一起的，而在之后系统就直接把根据操作系统的类型直接分开了。
并且还会保存build.prop与build.prop.version属性。来判断是不要更新。之后的版本则是fingerprint是记录，同时记录自己安装installed_apks.xml

.. code-block:: bash

   .. include:: libloadorder.log.txt


New gdbserver selection logic:
------------------------------

 1. Push gdbserver to /data/local/tmp/ and chmod 777 it.
 2. If it runs/attaches - use it.
 3. Cat-copy gdbserver from /data/local/tmp/ to /data/data/<package>/ and chmod 777 it.
 4. If it runs/attaches - use it.
 5. Use /data/data/<package>/lib/gdbserver. If it doesn't run/attach - display error message.

"Cat-copy" command is "cat <from> | run-as <package> sh -c 'cat > <to>'".

开起不了gdb就可以这样的方式的查找哪里问题。



