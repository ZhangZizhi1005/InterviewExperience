# 进程 #

---

[toc]

---

### 什么是进程, 什么是线程, 什么是协程 ###

- **进程** 是: 
    - 资源分配的基本单位 (资源包括CPU, 内存等)
    - 具有一定功能的程序 关于某个数据集合 的一次运行活动
    - 进程是系统进行 资源分配和调度 的一个独立单位
- **线程** 是: 
    - 进程的一个实体
    - 独立运行和独立调度 的基本单位 (CPU事实上运行的是线程)
    - 线程自己不拥有系统资源, 但是可以和同属一个进程的线程 共享进程拥有的全部资源
- **协程** 是:
    - 一种程序组件, 由程序控制而不是CPU调度
    - 协程的本地数据在后续调用中始终保持
    - 协程在控制离开时暂停执行，当控制再次进入时只能从离开的位置继续执行

---

### 进程和线程的区别是什么 ###

| 进程                                                         | 线程                                               |
| ------------------------------------------------------------ | -------------------------------------------------- |
| 资源分配的基本单位                                           | 程序执行的基本单位                                 |
| 拥有自己的资源空间<br>每启动一个进程, 系统就会为它分配地址空间 | 多个线程共享同一进程内的资源<br>使用相同的地址空间 |

基于这个根本的区别, 带来了这两者各自的优劣

- 通信
    - 线程之间的通信更为方便, 同一个进程下的线程共享全局变量, 静态变量等数据
    - 进程之间的通信必须依赖IPC进行, 带来的复杂性和开销更大
- 线程的调度和切换 远远快于进程
- 创建一个线程的开销 也远远小于一个进程
- 多进程程序更为健壮
    - 多线程程序只要有一个线程死掉,就可能锁住资源, 导致整个程序挂起
    - 一个进程死掉, 理论上不会对另一个进程造成影响, 因为进程有自己独立的地址空间

---

### 多进程和多线程对比 ###

|           | 多进程                           | 多线程             |
| --------- | -------------------------------- | ------------------ |
|           | 每个进程拥有自己的数据, 互不干涉 | 共享所属进程的数据 |
| 共享      | 需要IPC, 比较复杂                | 简单               |
| 同步      | 简单                             | 难                 |
| 占用内存  | 少                               | 多                 |
| 切换      | 复杂                             | 简单               |
| CPU利用率 | 低                               | 高                 |
| 创建销毁  | 复杂                             | 简单               |
| 健壮性    | 强                               | 弱                 |

---

### 进程间通信 ###

> - 竞态条件: 两个或多个线程同时对一共享数据进行修改，从而影响程序运行的正确性的条件
> - 临界区: 每个进程中访问临界资源的那段代码称为临界区（Critical Section）（临界资源是一次仅允许一个进程使用的共享资源

​	因此, 一个通信解决方案, 必须包含以下四种条件

> 1. 任何时候两个进程不能同时处于临界区
> 2. 不应对 CPU 的速度和数量做任何假设
> 3. 位于临界区外的进程不得阻塞其他进程
> 4. 不能使任何进程无限等待进入临界区

​	IPC 的解决方案一般包括以下七种:

- **管道PIPE**：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系
- **命名管道FIFO**：有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信
- **消息队列MessageQueue**：消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点
- **共享存储SharedMemory**：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量，配合使用，来实现进程间的同步和通信
- **信号量Semaphore**：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段
- **套接字Socket**：套解口也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同机器间的进程通信
- **信号Signal** ： 信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生

---

### 什么是进程空间 ###

---

### 怎么创建一个进程 ###

- 系统初始化：启动操作系统时，通常回创建若干个进程， 包括前台进程和后台进程
    - 前台进程：同用户进行交互并完成用户工作的进程
    - 后台进程：运行在后台，并不与特定用户进行交互的进程，也称为守护进程
- 正在运的程序执行了创建进程的系统调用（例如fork）
- 用户请求创建一个新的进程
- 初始化一个批处理工作

---

### 怎么杀死一个进程 ###

1. 定位进程：

    ps命令：process status的简称，用于报告当前系统的进程状态。此命令常配合grep过滤输出结果

    ```bash
    ps -aux | grep ***
    # a-显示所有用户的进程
    # u-显示进程的用户和拥有者
    # x-显示不依附于终端的进程
    ```

2. 杀死进程 :

    kill：通过进程ID来结束进程

    killall：通过进程名字结束进程

| Signal Name | Single Value | Effect         |
| :---------- | :----------- | :------------- |
| SIGHUP      | 1            | 挂起           |
| SIGINT      | 2            | 键盘的中断信号 |
| SIGKILL     | 9            | 发出杀死信号   |
| SIGTERM     | 15           | 发出终止信号   |
| SIGSTOP     | 17, 19, 23   | 停止进程       |

---

### 进程的切换机制 

​	操作系统 为了执行进程间的切换，会维护着一张表格，这张表就是 进程表(process table)

​	每个进程占用一个进程表项

​	该表项包含了进程状态的重要信息，包括程序计数器、堆栈指针、内存分配状况、所打开文件的状态、账号和调度信息，以及其他在进程由运行态转换到就绪态或阻塞态时所必须保存的信息

​	从而保证该进程随后能再次启动，就像从未被中断过一样

<img src="https://pic.leetcode-cn.com/1612664928-MVzWLA-os2-8.png" alt="os2-8.png" style="zoom: 33%;" />

---

