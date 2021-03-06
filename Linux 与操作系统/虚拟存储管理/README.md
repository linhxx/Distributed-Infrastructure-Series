# 虚拟存储管理

在[存储管理](https://parg.co/Z47)一节中我们介绍了计算机系统中常见的存储类型与管理方式，本章则是对于虚拟存储管理相关的内容进行详细介绍。

虚拟存储器是基于程序局部性原理上的一种假想的而不是物理存在的存储器，允许用户程序以逻辑地址来寻址，而不必考虑物理上可获得的内存大小。这种将物理空间和逻辑空间分开编址但又统一管理和使用的技术为用户编程提供了极大方便，它为每个进程提供了一个假象，即每个进程都在独占地使用主存。此时，用户作业空间称虚拟地址空间，其中的地址称虚地址。

虚拟存储器的容量由计算机的地址结构和辅助存储器的容量决定，建立在离散分配的存储管理方式的基础上，它允许将一个作业分多次调入内存。Linux 内存管理的设计充分利用了计算机系统所提供的虚拟存储技术，真正实现了虚拟存储器管理，其关注于程序编译链接后形成的地址空间管理、如何将虚地址转化为物理地址等方面。

从进程的视角来看，其虚拟地址空间最上面的区域是为操作系统中的代码和数据保留的，这对所有进程来说都是一样的。地址空间的底部区域存放用户进程定义的代码和数据。

![image](https://user-images.githubusercontent.com/5803001/52272019-52032000-2980-11e9-953c-89de286e5174.png)

# 虚拟内存、内核空间和用户空间

Linux 简化了分段机制，使得虚地址与线性地址总是一致的；系统将虚拟地址分为两部分：一部分专门给系统内核使用，另一部分给用户进程使用。对于 32 位的系统，虚拟地址范围是 0x00000000 ~ 0xFFFFFFFF，即最大虚拟内存为 2^32 Bytes = 4GB，系统将最高的 1G 字节（从虚拟地址 0xC0000000 到 0xFFFFFFFF）分配内核使用，此区域称作内核空间；另外将较低的 3G 字节（从虚拟地址 0x00000000 到 0xBFFFFFFF），供各个进程使用，称为用户空间。

![](https://i.postimg.cc/FFDMk15p/image.png)

因为每个进程可以通过系统调用进入内核，因此, Linux 内核空间由系统内的所有进程共享。于是，从具体进程的角度来看，每个进程可以拥有 4GB 的虚拟地址空间（也叫虚拟内存）。

- 内核空间：系统内核使用的内存空间，当一个进程执行调用系统命令(例如 read, write)时，会进入内核代码的执行，进程此时的状态我们称之为内核态。
- 用户空间：用户进程使用的内存空间，当一个进程执行用户自己的代码时，该进程此时的状态为用户态。

任意一个时刻,在一个 CPU 上只有一个进程在运行。所以对于此 CPU 来讲,在这时刻,整个系统只存在一个 4GB 的虚拟地址空间,这个虚拟地址空间是面向此进程的。当进程发生切换的时候,虚拟地址空间也随着切换。Linux 为每一个进程都建立其页表,将每个进程的虚拟地址空间根据自己的需要映射到物理地址空间上。既然某一时刻在某一 CPU 上只能有个进程在运行,那么当进程发生切换的时候,将页表也更换为相应进程的页表,这就可以实现每个进程都有自己的虚拟地址空间而互不影响。所以,在任意时刻对于一个 CPU 来说,只需要有当前进程的页表,就可以实现其虚拟地址到物理地址的转化。

虽然内核空间占据了每个虚拟空间中的最高 1GB,但映射到物理内存却总是从最低的地址(0x0000000)开始的,以方便在内核空间与物理内存之间建立起简单的线性映射关系。其中,3GB(0xC000000)就是物理地址与虚拟地址之间的位移量,在 Linux 代码中就叫做 `PAGE_OFFSET`。
