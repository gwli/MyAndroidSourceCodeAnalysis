

NDK 消息处理
============

是经过 ALooper 再加上一消息队列来实现。 Android中每一个线程都可以一个消息队列。

最初使的声明可以在sensor.h 看到。实现是在 :file:`/frameworks/native/include/utils/Looper.h`


vold进程用来发送消息
====================

ginerbreak进程用来发送消息的。
