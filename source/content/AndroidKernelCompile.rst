如何在内核中编译你的module
====================================

   
.. ::
 
   make -C ~/work/android/15r7/out/debug/target/product/ventana/obj/KERNEL M=`pwd` ARCH=arm CROSS_COMPILE=~/work/android/15r7/prebuilt/linux-x86/toolchain/arm-eabi-4.4.3/bin/arm-eabi- modules         
   


.. csv-table:: 

   *-C* , change to kernel path to do build. ,
   M=`pwd`,  just tell you the module source code path ,
   CROSS_COMPILE, it was the prefix of build-tools  ,
   -O , specify the output of build ,
   modules  the target to build ,

normally, the module is relevant with the kernerl version. it would check the kernel version to make sure compatible.  how to check it . it use the uts_release.h to get the version.  the uts means `unix time sharing system <http://en.wikipedia.org/wiki/Time-sharing>`_ 
normally, you just need to change two things.

.. csv-table:: 

   ARCH ,  CPU arch type ,
   toolchain , you just change it path prefix, if you follow the convention ,

#. check your target kernel version 
#. change */out/target/product/ardbeg/obj/KERNEL/include/generated/utsrelease.h to your target version
#. use the above command to build your module, you need set Module path with *M=" and CROSS_COMPILE= with toolchain prefix.
#. push the model to the */system/modules*
   
Module install and 
===================


.. csv-table:: 

   /out/target/product/ardbeg/obj/KERNEL/modules.bultin ,  bultin modules list ,
   /out/target/product/ardbeg/obj/KERNEL/modules.order ,  most of the time, you module has dependencies, you should insmod following some sequence ,
   source code  Kbuild , there is document to build module. , module.txt ,

source code build
=================


.. csv-table:: 

   <source code>/build/core ,  just like the ndk build there is all build make file  ,
   <srouce code>/kernel/Makefile , kernel_release , you change it by yourself, this would change all kernel version 

.. ::
 echo "3.8.2-g15235dc" >$@

.. csv-table:: 

   
   Documentation/kbuild/kbuild.txt ,  记录了所有的kernel 编译的需要的make变量定义 ,
   Documentation/kbuild/modules.txt , 讲解如何编译modules ,

   
   -+nvfash
   ========
   
   其实这个就是分区装系统的过程，如果你在USB上装上，并且在USB做过多个分区，并安装系统，那么这对你来说就是小菜一碟了。其实主要就是制作分区表，然后在不同的位置进行读写，如果你能够强的话，是可以直接用dd进行读写来直接烧录。
      
.. ::
 
     #.  should usb recovermode driver
     #.  copy nvflash.exe and dll
     #. nvflash.exe --bct bct.cfg --setbct --odmdata 0x80098000 --configfile flash.cfg --create --bl bootloader.bin --go
      


*boot.img，system.img, bootloader.bin* 最终只需要这个三个文件，再加上flash工具就行了。
正常情况下要把/out/target/product/arddeg/ 下面东西都是需要的，但是同一产品不同的版本之间基本只有那boot.img,system.img,bootloader.bin 变化。当然不是完全都在变化。
而在/out/host/下面生成都是一些工具，例如那些nvflash 等等。
#. `tftp_flash <http://www.dd-wrt.com/wiki/index.php/Tftp_flash>`_  这个让我想起了精日的日子。
#. `to write a linux image in a CF card with DD <http://stackoverflow.com/questions/12519749/dd-opening-dev-sdb-permission-denied>`_  
#. `nvflash from XDA  <http://forum.xda-developers.com/wiki/Talk:Nvflash>`_ 

See also
========

#. `Can&#39;t build hello world kernel module on Android JellyBean <http://stackoverflow.com/questions/14389326/cant-build-hello-world-kernel-module-on-android-jellybean>`_  
#. `HowTo - Building Android Kernel Modules  <http://www.cs.duke.edu/~schfan/blog/blog/2012/04/09/howto-building-android-kernel-modules-bcm4329-dot-ko/>`_  
#. `compiling-kernel-modules-tunko-for.htm <http://stevechui.blogspot.com/2011/10/compiling-kernel-modules-tunko-for.html>`_  
#. `further-issues-kernel-configuration <http://crashcourse.ca/introduction-linux-kernel-programming/lesson-2-further-issues-kernel-configuration>`_  

Thinking
========




*Kbuild* 编译内核以及module时需要用到的build系统。

-- Main.GangweiLi - 04 Nov 2013


*开机起动*
   */etc/mkshrc*
   */init.rc* 等可以使用，如果想修改这些文件，先要adb remount 一下，就可以写文件了。
   

-- Main.GangweiLi - 28 Feb 2014
