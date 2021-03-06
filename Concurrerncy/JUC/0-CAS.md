## CAS ##

> 由于**synchronized是采用的是悲观锁**策略，并不是特别高效的一种解决方案。 实际上，在J.U.C下的atomic包提供了一系列的操作简单，性能高效，并能保证线程安全的类去 更新基本类型变量，数组元素，引用类型以及更新对象中的字段类型。 atomic包下的这些类都是采用的是**乐观锁策略**去原子更新数据，在Java中则是使用CAS操作具体实现

---

### 什么是CAS ###

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：

- 先进行操作，如果没有其它线程争用共享数据，那操作就成功了
- 否则采取补偿措施（不断地重试，直到成功为止）

这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步, 但是这样我们同时就失去了用互斥来保持同步的可能性

所以, 实现 操作 和 冲突检测 这两个步骤必须依赖于 硬件支持的原子性操作

一个典型的硬件支持的原子性操作就是: **比较并交换**(Compare and Swap, CAS)

**CAS** 指令需要有三个操作数

- 内存地址 V
- 旧的 预期值 A
- 新的 预期值 B

**CAS** 指令的执行策略是:

​	当执行操作时, 只有当V的值等于A, 即符合预期, 才将V更新为B

**CAS** 实现同步的基本思路是:

​	从地址V 读取一个值 为A, 执行多步计算得到一个值为B, 应用CAS将V的值更改为B,

---

### CAS 的问题 ###

| 问题名称                       | 问题描述                                                     | 解决办法                                                     |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ABA问题                        | 如果一个变量初次读取的时候是A, <br>它的值被改为了B, 而后又改回为A, <br>那么CAS就会误认为这个变量从未改变过 | J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题， 它可以通过控制变量值的版本来保证 CAS 的正确性<br> 大部分情况下 ABA 问题不会影响程序并发的正确性， 如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效 |
| 自旋时间长开销大               | 自旋CAS（也就是**不成功就一直循环执行直到成功**）<br>如果长时间不成功，会给CPU带来非常大的执行开销 | 如果JVM能支持处理器提供的pause指令那么效率会有一定的提升<br>pause指令有两个作用<br>第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源， 延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。 <br>第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation） 而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率 |
| 只能保证一个共享变量的原子操作 | 当操作涉及跨多个共享变量时CAS无效                            | 从 JDK 1.5开始，提供了AtomicReference类来保证引用对象之间的原子性<br>可以把多个变量封装成对象里来进行 CAS 操作<br>所以我们可以使用锁或者利用AtomicReference类把多个共享变量封装成一个共享变量来操作 |

---

### CAS 实现 Concurrent 包 ###

由于java的CAS同时具有 volatile 读和volatile写的内存语义，因此Java线程之间的通信现在有了下面四种方式：

1. A线程写volatile变量，随后B线程读这个volatile变量。
2. A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
3. A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
4. A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。

Java的CAS会使用现代处理器上提供的高效机器级别原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键（从本质上来说，能够支持原子性读-改-写指令的计算机器，是顺序计算图灵机的异步等价机器，因此任何现代的多处理器都会去支持某种能对内存执行原子性读-改-写操作的原子指令）。同时，volatile变量的读/写和CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现的基石。如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式：

1. 首先，声明共享变量为volatile；
2. 然后，使用CAS的原子条件更新来实现线程之间的同步；
3. 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

AQS，非阻塞数据结构和原子变量类，这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的, 所以CAS就是实现整个JUC的基础