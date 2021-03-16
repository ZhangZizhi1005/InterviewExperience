# 目录 #

---

[toc]

---

---

# 线程 #

---

## 多线程中 ```i++```安全吗？ 不安全应该如何解决 ##

- 线程不安全，```i++``` 执行了多步操作

    1. 读取i的值
    2. 将i的值+1
    3. 将+1后的值写回i中

    这其中的任何一步都有可能被其他线程抢占

    > 假设有两个线程执行```i++```, 那么最后最大为200, 最小为2

- 我们可以使用```java.util.concurrent.atomic``` 包来提供线程安全的基本类型包装类

---

## 如何线程安全的实现一个计数器 ##

- 使用```java.util.concurrent.atomic``` 包的 ```AtomicInteger``` 类, 这是一个线程安全的基本类型包装类

- 使用```synchronized```关键字或者加锁, 来自己实现一个包装类

    ```java
    public class AtomicCounter {
        private volatile int count = 0;
        
        public synchronized void increment(){
            count++;
        }
        
        public int getCount(){
            return count;
        }
    }
    ```

---

## 多线程同步的方法有哪些 ##

1. 使用```synchronized```关键字

    - Java为每个对象都提供了一个内置锁, 当用```synchronized```关键字修饰的时候, 内置锁会保护被修饰的部分, 只有获得这把锁才能调用这部分代码

    - ```synchronized``` 关键字既可以用来修饰方法, 也可以用来修饰语句块

        - ```synchronized```关键字修饰方法示例

            ```java
            public class mySyn{
                public synchronized void mySynMethod(){
                    ...
                }
            }
            // 特别的,如果使用synchronized关键字锁住一个静态方法,那么将会锁住整个类
            ```

        - ```synchronized```关键字修饰代码块: 被该关键字修饰的语句块会自动加上内置锁

            ```java
            public class mySyn{
                public  void mySynCodeBlock(){
                    synchronized(this){
                        ...
                    }
                }
            }
            // 同步是一个开销很高的工作,通常不必要同步整个方法,只需要同步关键代码即可
            ```

2. 使用```volatile```关键字来标识特殊域变量

    - ```volatile```关键字为域变量提供了一种免锁机制, 使用```volatile```修饰等于告诉虚拟机,该域会被其他线程更新
    - 因此, 每次使用这个域都需要重新计算而不能从寄存器中读取
    - ```volatile```不提供任何原子操作, 也不能修饰```final```变量(因为其本身就不可变了)

3. 使用```java.util.concurrent```包中, 实现了```Lock```接口的类来实现锁, 例如```ReentrantLock```类

    - 与```synchronized```具有基本相同的行为和语义
    - 实现了一定的扩展, 在需要某些高级功能的时候使用

4. 使用```ThreadLocal```管理变量

    - ```ThreadLocal```管理变量指: 每一个使用该变量的线程都获得该变量的副本,副本之间相互独立

    - ```ThreadLocal```代码示例

        ```java
        public class myThreadLocalClass{
            private static ThreadLocal<Integer> myThreadLocalVariable = new ThreadLocal<Integer>();
        }
        ```

---

## 简述生产者/消费者模式 ##

- 在实际的软件开发过程中, 我们常常抽象出一个模块(生产者)用来产生数据, 另外一个模块(消费者)负责处理数据

    但是, 依赖于一个中间的缓冲区, 才能称为生产者/消费者模式

- 作用

    - 解耦: 假定不设置缓冲区, 则生产者必须依赖于消费者的某些特定接口或方法, 有了缓冲区，这两者就无需直接依赖了
    - 并发: 生产者直接调用消费者的另一个弊端是, 函数调用是同步的(或者称为阻塞式的), 在消费者方法未返回前 生产者只能等待. 但是使用了这种模式之后, 生产者和消费者将会是两个独立的并发主体, 基本上不依赖消费者的处理速度
    - 支持忙闲不均: 假定生产数据的速度不恒定, 缓冲区可以保存尚未处理的数据, 留待闲时处理

---

## Java中创建线程的方式有哪几种 ##

1. 继承```Thread```类实现
    1. 定义一个```Thread```类的继承类
    2. 重写```Thread```类的```run()```方法: ```run()方法```的方法体就是线程要完成的任务, 因此也被称为线程的执行体
    3. 创建该类的实例对象
    4. 调用线程对象的```start()```方法
    
2. 实现```Runnable```接口
    1. 定义一个```Runnable```接口的实现类
    2. 创建这个类的实例对象
    3. 将这个对象, 作为构造器参数传入```Thread```类的实例对象(真正的线程对象)
    4. 调用线程对象的start方法
    
    - 特别的:
        - 实现```Runnable```接口的对象只是作为```Thread```类对象的 *target* ; ```Runnable```实现类中包含的```run()```方法只是线程的执行体 , 实际的线程对象仍然是```Thread```实例
        - 通过这种方式实现多线程的时候, 获取当前线程不能用this, 而必须用```Thread.currentThread()```方法
        - 可以用lambada表达式简化代码
    
3. 通过```Callable```和```Future```接口创建

    1. 定义一个```Callable```接口的实现类, 并实现```call()```方法
    2. 创建这个类的实例对象, 并用```FutureTask```类来包装该对象
    3. 使用```FutureTask```对象, 作为构造器参数传入```Thread```类的实例对象
    4. 调用线程对象的```start()```方法启动线程
    5. 调用```FutureTask```对象的```get()```方法来获取子线程执行结束后的返回值
    
    - ```Callable```接口:
        - ```Callable```接口提供了一个```call()```方法作为线程执行体
            - ```call()```方法可以有返回值
            - ```call()```方法可以声明抛出异常
        - ```Callable```接口的实现类的对象,不能作为```Thread```类对象的 *Target*
            - ```Callable```接口是新增的接口, 并非```Runnable```接口的子接口, 所以不能直接作为 *Target*
            - ```Call()```方法有返回值, 但是```call()```方法不能直接调用, 只能作为线程执行体调用, 那么返回值需要特殊获取
    - 为了解决以上问题, 需要引入```Future```接口
        - ```Future```接口可以对```Callable```中执行的任务进行操作, 最基本的就是获取任务执行结果
            - ```boolean isDone()```: 判断任务是否完成, 完成返回true
            - ```boolean cancel(boolean mayInterruptIfRunning)```: 终止执行过程中的任务, 如果任务已经完成或者设置为不允许被 *Interrupt* 则返回false; 如果任务尚未开始或者设置为允许被 *Interrupt* 返回true;
            - ```V get() thrwos InterruptedException, ExecutionException``` & ```V get(long timeOut, TimeUnit unit) throws InterruptedException, ExecutionException,TimeoutException``` : 获取执行结果, 但是前一个方法会产生阻塞, 后一个方法再超时后会自动返回null
        - ```Future```接口的实现类```FutureTask```同时也实现了```Runnable```接口
            - 既可以作为```Future```对象得到
            - 也可以作为```Runnable```对象作为线程的 *Target*

---

---

# 线程池 #

---

## 什么是线程池 ##

[参考资料1](https://www.cnblogs.com/dolphin0520/p/3932921.html)

[参考资料2](https://blog.csdn.net/weixin_40271838/article/details/79998327)

> 线程池是一种线程的使用模式
>
> - 创建若干个可执行的线程放入一个容器中(池)
> - 有任务需要处理时, 将任务提交到线程池的任务队列
> - 调用可用的线程执行任务
> - 处理完的线程不销毁, 而是在池中等待下一个任务

- 为什么使用线程池

    - 线程是稀缺资源, 频繁的创建会导致大量的开销
    - 解耦: 将线程的 创建 和 执行 分离, 便于维护
    - 提高响应速度, 任务到达时, 无需等待线程创建即可直接执行

- 线程池的创建

    - 利用```ThreadPoolExecutor```类的构造器  或 提供的接口实现(详见下 四种线程池)
        - ```ThreadPoolExecutor(int corePoolsize, int maximunPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)```
        - 参数作用:
            - corePoolSize : 核心线程池的大小, 也称为线程池的基本大小
            - maximumPoolSize : 线程池的最大线程大小
            - keepAliveTime 和 unit : 线程空闲后的存活时间
            - workQueue : 用于存放任务的阻塞队列
            - handler : 当最大线程池和队列都饱和之后的策略

- 线程池的使用

    - 线程池的状态:

        ​		![img](https://segmentfault.com/img/remote/1460000015808902?w=707&h=126)

        - RUNNING : 运行状态, 可以接受任务执行队列里的任务
        - SHUTDOWN : 调用了```shutdown()```方法 , 不再接受新的任务, 但是需要等待队列中的任务执行完毕
        - STOP : 调用了```shutdownNow()```方法, 不再接受新的任务, 同时抛弃阻塞队列里的所有任务并且中断所有正在执行的任务
        - TYDYING : 所有任务都执行完毕(调用```shutdown()```或者```shutdownNow()```方法都会试图更新为这个状态)
        - TERMINATED : 终止状态, 调用```terminated()```方法后会更新为这个状态

        <img src="ref/1460000015808903" alt="img" style="zoom: 67%;" />

    - 线程池的主要处理流程

        <img src="ref/1460000015808905" alt="img"  /><img src="ref/1460000015808904" alt="img" style="zoom: 67%;" />

        - 调用 ```threadPool.execute(new Job())```方法, 提交任务
        - 处理```execute()```方法 :
            - 提交任务后, 会获取线程池的当前状态
            - 判断 当前线程数量 是否小于 corePoolSize规定的数量; 
                - 如果小于, 创建新的线程执行任务
                - 否则 将任务存储到阻塞的任务队列中
            - 如果队列已满, 存储失败, 判断当前线程数量 是否小于 maximumPoolSize规定的线程数量, 
                - 如果小于,  创建新的线程执行任务
                - 如果已满, 执行无法处理时的拒绝策略

---

## Java中有哪几种线程池 ##

> 通过```Executor```类的工厂方法,可以创建几类常用的 遵循特定规则的线程池

| 方法                                     | 作用                                                         | 优点                                                         | 缺陷                                                         |
| ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ```Executor.newFixedThreadPool()```      | 创建一个指定工作线程数量的线程池<br>每当提交一个任务就创建一个工作线程<br>如果工作线程数量达到线程池初始的最大值, 则将提交的任务存入线程池队列中 | 典型且优秀的线程池<br>具有线程池提高程序效率和节省创建线程开销的优点 | 在线程池空闲时(即没有可运行任务时), 线程池不会释放工作线程, 并且会占用一定的系统资源 |
| ```Executor.newCachedThreadPool()```     | 创建一个可缓存的线程池<br>如果线程池的长度超出了处理需要, 可以灵活的回收空闲线程<br>在没有可以回收的线程时, 才会新建线程 | 工作线程的创建数量几乎没有限制(除非超过Integer.MAX_VALUe)    | 如果长时间未向线程池中提交任务(即工作线程空闲时间超出指定的时间), 该线程会被回收, 即需要重用时, 必须重新创建<br>必须时刻注意任务数量,否则可能会引起由于大量线程同时运行造成的系统瘫痪 |
| ```Executor.newSingleThreadExecutor()``` | 创建一个单线程化的线程池                                     | 线程池中任意给定时间内只会有唯一的工作线程来执行任务<br>保证任务会按照指定的顺序(LIFO or FIFO)来执行<br>如果这线程异常结束, 会创建一个新的工作线程取代, 保障任务顺序执行 |                                                              |
| ```Executor.newScheduleThreadPool()```   | 创建一个支持定时或者周期性执行的线程池                       |                                                              |                                                              |

---

