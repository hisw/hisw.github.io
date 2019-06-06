<html>
<head>
    <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
    <script>
        (adsbygoogle = window.adsbygoogle || []).push({
        google_ad_client: "ca-pub-1724482933749702",
        enable_page_level_ads: true
        });
    </script>
</head>
<body>
# Linux内核核心架构 #
![resource/architecture.JPG](resources/kernel/architecture.jpg)
## System call interface ##
对应用程序的API接口机制

SCI是一个薄层，提供了从用户空间到内核执行函数调用的方法。 如前所述，即使在同一处理器系列中，此接口也可能依赖于体系结构。 SCI实际上是一种有趣的功能调用"多路复用和多路分解"服务。 您可以在./linux/kernel中找到SCI实现，也可以在./linux/arch中找到与体系结构相关的部分。

![调用一个系统调用](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHAvaW1hZ2VzLmNuaXRibG9nLmNvbS9ibG9nMjAxNS83MzEyMDgvMjAxNTAzLzMwMDExOTI2MTU0NTc0Ny5wbmc=.jpg)


>应用程序调用API时在应用层执行“CPU提供的内核调用中断”从而切换运行环境到内核态。

## Process/Threads ##
内核的核心，提供并行工作能力。

![The Linux O(1) scheduler algorithm.](http://books.gigatux.nl/mirror/kerneldevelopment/0672327201/images/0672327201/graphics/04fig02.gif)

在内核中，这些被称为线程，代表处理器的单独虚拟化（线程代码，数据，堆栈和CPU寄存器）。 在用户空间中，通常使用术语“进程”，尽管Linux实现不会将这两个概念（进程和线程）分开。

内核实现了一种新颖的调度算法，该算法在恒定时间内运行，而不管争用CPU的线程数量。 这称为O（1）调度程序，表示调度一个线程所花费的时间与调度多个线程所花费的时间相同。

## Memory Management ##
对内存进行管理：

1. 分配/释放（对于大多数架构，内存被以4KB的页进行分割管理）

	内存管理不仅仅是管理4KB缓冲区。 Linux提供超过4KB缓冲区的抽象，例如slab分配器。 此内存管理方案使用4KB缓冲区作为其基础，但随后从内部分配结构，跟踪哪些页面已满，部分使用和空。 这允许该方案基于更大系统的需要动态地增长和缩小。

1. 通过MMU进行虚拟内存与物理内存的映射

	![MMU地址翻译](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1541665863810&di=e873afd52314308ca8abed2c818dc3e4&imgtype=0&src=http%3A%2F%2Fwww.eechina.com%2Fimage%2Ftech%2F2011032813570441.gif)

	现代操作系统使用虚拟地址进行寻址，MMU翻译虚拟地址并访问实际的物理地址。虚拟地址和物理地址的映射由内核管理,这样可以对物理地址进行保护。

1. 提供内存交换机制

	支持多个内存用户，有时可用内存耗尽。 因此，页面可以移出内存并移动到磁盘上，此过程称为交换，因为页面已从内存交换到硬盘上。 您可以在./linux/mm中找到内存管理源。
## Virtual file system ##
对应用程序提供树型接口机制，“一切皆文件”的核心。

>图：VFS在用户和“文件系统/内核功能"之间提供交换结构

![文件系统接口](https://www.ibm.com/developerworks/library/l-linux-kernel/figure4.jpg)

>在VFS的顶部是一组常见的API抽象函数，如open，close，read和write。 在VFS的底部是文件系统抽象，它定义了如何实现上层功能。 这些是给定文件系统的插件（其中存在超过50个），同时也为用户提供内核功能的抽象为"文件"的接口。 您可以在./linux/fs中找到文件系统源。

>文件系统层下面是缓冲区缓存，它为文件系统层提供了一组通用功能（独立于任何特定的文件系统）。 此缓存层通过将数据保持一小段时间（或推测性地预读，以便在需要时可用数据）来优化对物理设备的访问。 缓冲区高速缓存下面是设备驱动程序，它们实现特定物理设备的接口。
## Device Drivers ##
内核同硬件的接口，内核提供设备驱动模型对设备驱动的关系进行组织。

![设备驱动模型](https://i.imgur.com/e0jRwFL.jpg)

驱动依赖总线执行对设备的控制,设备依赖驱动来执行设备特有的控制。

>- 设备可通过中断控制器产生中断。
- 驱动通过对设备地址的访问来执行对设备的驱动；
- 设备地址分为物理地址和总线地址两种；
- 通过总线地址访问设备的驱动依赖总结驱动提供的接口。
- 类设备使用类子系统提供的类型功能。

Linux内核中的绝大多数源代码存在于使特定硬件设备可用的设备驱动程序中。 Linux源代码树提供了一个驱动程序子目录，该子目录被所支持的各种设备进一步划分，例如蓝牙，I2C，串行等。 您可以在./linux/drivers中找到设备驱动程序源。

## Network/Socket ##
Linux使用Socket提供网络/本地复杂IPC通信接口。

![tcp](http://i.imgur.com/cqr4O2P.png)

这个一个TCP通信的过程。Linux使用fd(文件系统描述符)来绑定通信的本地网络地址，进行Socket操作，执行网络通信。

![Domain sockets](http://www.cs.columbia.edu/~jae/4118-LAST/L09/fig17.1.jpg)

Local Domain Socket也叫本地IPC或Unix Socket, 使用一个文件系统路径名代替网络地址来标致进程地址。
</body>
</html>