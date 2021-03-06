# 并发控制

涉及多线程程序涉及的时候经常会出现一些令人难以思议的事情，用堆和栈分配一个变量可能在以后的执行中产生意想不到的结果，而这个结果的表现就是内存的非法被访问，导致内存的内容被更改。在一个进程的线程共享堆区，而进程中的线程各自维持自己堆栈。 在 Windows 等平台上，不同线程缺省使用同一个堆，所以用 C 的 malloc (或者 windows 的 GlobalAlloc)分配内存的时候是使用了同步保护的。如果没有同步保护，在两个线程同时执行内存操作的时候会产生竞争条件，可能导致堆内内存管理混乱。比如两个线程分配了统一块内存地址，空闲链表指针错误等。

最常见的进程/线程的同步方法有互斥锁(或称互斥量 Mutex)，读写锁(rdlock)，条件变量(cond)，信号量(Semophore)等；在 Windows 系统中，临界区(Critical Section)和事件对象(Event)也是常用的同步方法。总结而言，同步问题基本的就是解决原子性与可见性/一致性这两个问题，其基本手段就是基于锁，因此又可以分为三个方面：指令串行化/临界资源管理/锁、数据一致性/数据可见性、事务/原子操作。在并发控制中我们会考虑线程协作、互斥与锁、并发容器等方面。

# 线程通信

并发控制中主要考虑线程之间的通信(线程之间以何种机制来交换信息)与同步(读写等待，竞态条件等)模型，在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递。Java 就是典型的共享内存模式的通信机制；而 Go 则是提倡以消息传递方式实现内存共享，而非通过共享来实现通信。

在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信。在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。同步是指程序用于控制不同线程之间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。

常见的线程通信方式有以下几种：

- 管道（Pipe）：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用，其中进程的亲缘关系通常是指父子进程关系。

- 消息队列（Message Queue）：消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。

- 信号量（Semophore）：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。

- 共享内存（Shared Memory）：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量配合使用，来实现进程间的同步和通信。

- 套接字（Socket）：套接字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同主机间的进程通信。
