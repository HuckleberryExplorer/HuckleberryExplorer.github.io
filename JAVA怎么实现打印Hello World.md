---
layout: default
title:  JAVA怎么实现打印Hello World?
categories: 
tag:    
---

任何一个学过JAVA的人应该都对这段代码非常熟悉。空闲时间翻了下代码，看看它的底层是怎么实现的

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.print("Hello, World!");
    }
}
```

首先点开`out`，发现它是`System`类中的一个`public static final`变量，类型为`PrintStream`。为了找到它是怎么初始化的，一直往前翻到`System`类的构造函数

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527001719.png" alt="" data-align="center" width="500">
</div>

从`System`类的注释中发现，VM会调用`initPhase1`这个方法来初始化这个类。先不管VM，先看下`initPhase1`方法做了什么

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527002208.png" alt="" data-align="center" width="666">
</div>

发现它用`FileDescriptor.out`创建了`FileOutputStream`对象，再用这个对象创建了`PrintStream`对象，最后调用native的`setOut0`

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527002734.png" alt="" data-align="center" width="500">
</div>

在创建`PrintStream`对象时，先将`FileOutputStream`封装成`BufferedOutputStream`，然后把`BufferedOutputStream`封装成`OutputStreamWriter`。这一步中会根据传入的字符集创建`OutputStreamWriter`中的编码器`StreamEncoder`

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527003340.png" alt="" data-align="center" width="500">
</div>

这样`OutputStreamWriter`就创建好了，在print的时候会调用这个类的方法，最后根据调用栈发现调用了`FileOutputStream`中的`writeBytes`方法

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527003553.png" alt="" data-align="center" width="333">
</div>

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527003856.png" alt="" data-align="center" width="540">
</div>

发现这个方法是native的，也就是说不在JAVA中实现，打开[openjdk](https://github.com/openjdk/jdk)，checkout到tag jdk18

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527112055.png" alt="" data-align="center" width="540">
</div>

找到在jdk中的实现，发现调用了`IO_Append`和`IO_Write`，而这两个是宏定义，指向了`handleWrite`方法

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527112307.png" alt="" data-align="center" width="234">
</div>

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527112200.png" alt="" width="433" data-align="center">
</div>

在不同的平台下，这个方法有不同的实现

在windows下调用了`WriteFile`这个Win32 API

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527112524.png" alt="" data-align="center" width="565">
</div>

在linux下调用了`unistd.h`中定义的`write`方法

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527113149.png" alt="" width="450" data-align="center">
</div>

打开[glibc](https://sourceware.org/git/glibc.git)，在`write_nocancel.c`下看到提供的write方法实现，通过一堆的宏定义最终是一个系统调用，调用了linux的`write`方法

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527125423.png" alt="" width="400" data-align="center">
</div>

在linux内核源代码中，找到`write`的SYSCALL，其中调用了`ksys_write`方法

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527130945.png" alt="" data-align="center" width="448">
</div>

这个方法中会获取fd，然后再通过`vfs_write`写入，顺着调用链一路找到了下面这个`write`方法

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527131847.png" alt="" data-align="center" width="444">
</div>

后面的两个参数很好理解，第一个`tty_struct`是什么？

> In many computing contexts, "TTY" has become the name for any text terminal, such as an external console device, a user dialing into the system on a modem on a serial port device, a printing or graphical computer terminal on a computer's serial port or the RS-232 port on a USB-to-RS-232 converter attached to a computer's USB port, or even a terminal emulator application in the window system using a pseudoterminal device.

简单来说，就是一个文本终端。它也是一个虚拟文件系统，模拟了终端设备

回头看`ksys_write`方法的第一行，打开了一个文件描述符，其中调用了`__fget_light`方法

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527133741.png" alt="" data-align="center" width="424">
</div>

从第一行就能看到，从`current`中找到了`files_struct`。`current`是一个宏定义，获取当前正在运行的任务`current_task`，而`current_task->files`是打开的文件信息

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527134424.png" alt="" data-align="center" width="441">
</div>

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527134520.png" alt="" data-align="center" width="311">
</div>

也就是说，它打开了这个虚拟文件系统中的虚拟文件，也是一个伪终端PTY，然后向里面写入

在linux中输入tty，就可以看到对应的伪终端设备文件路径，还可以向另一个终端中打印东西

<div align=center>
<img title="" src="https://raw.githubusercontent.com/HuckleberryExplorer/PicBed/main/20230527141115.png" alt="" data-align="center" width="699">
</div>

参考：

1. https://github.com/openjdk/jdk
2. https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-writefile
3. https://www.linusakesson.net/programming/tty/index.php
4. https://en.wikipedia.org/wiki/Devpts
5. https://man7.org/linux/man-pages/man7/pty.7.html
6. https://github.com/GNOME/vte