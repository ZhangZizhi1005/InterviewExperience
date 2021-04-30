# 单例模式

> 单例模式，属于创建类型的一种常用的[软件设计模式](https://baike.baidu.com/item/软件设计模式/2117635)。通过单例模式的方法创建的类在当前进程中只有一个[实例](https://baike.baidu.com/item/实例/3794138)（根据需要，也有可能一个线程中属于单例，如：仅线程上下文内使用同一个实例）

---

## 单例模式的要点 ##

| 要求                             | 特点                       |
| -------------------------------- | -------------------------- |
| 某个类只能有一个实例             | 类构造器私有               |
| 它必须自行创建这个实例           | 持有自己类型的属性         |
| 它必须自行向整个系统提供这个实例 | 对外提供获取实例的静态方法 |



---

## 单例模式构建方式 ##

- 懒汉式—线程不安全：最基础的实现方式，线程上下文单例，不需要共享给所有线程，也不需要加synchronize之类的锁，以提高性能
- 懒汉式—线程安全：加上synchronize之类保证线程安全的基础上的懒汉模式，相对性能很低，大部分时间并不需要同步
- 饿汉方式。指全局的单例实例在类装载时构建
- 双检锁式。在懒汉式基础上利用synchronize关键字和volatile关键字确保第一次创建时没有线程间竞争而产生多个实例，仅第一次创建时同步，性能相对较高
- 登记式。作为创建类的全局属性存在，创建类被装载时创建
- 枚举。java中枚举类本身也是一种单例模式

---

### 懒汉式 ###

​	懒汉式，顾名思义就是实例在用到的时候才去创建，“比较懒”，用的时候才去检查有没有实例，如果有则返回，没有则新建

​	有线程安全和线程不安全两种写法

#### 线程不安全 ####

```java
public class Singleton{
    private static Singleton instance;
    private Singleton(){
        ...
    }
    
    public static Singleton getInstance(){
        if(instance == null)
            instance = new Singleton();
        return instance;
    }
}
```

#### 线程安全 ####

[见下方双检锁](###双检锁式###), 是对懒汉模式的一种优化

---

### 饿汉式 ###

饿汉式，从名字上也很好理解，就是“比较勤”，实例在初始化的时候就已经建好了，不管你有没有用到，都先建好了再说

好处是没有线程安全的问题，坏处是容易产生垃圾, 浪费内存空间

```java
public class Singleton{
    private static Singleton instance = new Singleton();
    private Singleton(){
        ...
    }
    
    public static Sintgleton getInstance(){
        return instance;
    }
}
```

---

### 双检锁式 ###

线程安全，延迟初始化

这种方式采用双锁机制，安全且在多线程情况下能保持高性能

```java
public class Singleton{
    private static volatile Singleton instance;
    private Singleton(){
        ...
    }
    
    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

双重检查模式，进行了两次的判断:

- 第一次是为了提高效率, 如果已经存在实例, 线程不会被阻塞
- 第二次是为了进行同步，避免多线程问题
    - 假定两个线程都通过了第一次检查, 可能会重复创建实例, 这时必须上锁
- 由于`singleton=new Singleton()`对象的创建在JVM中可能会进行重排序，在多线程访问下存在风险，使用`volatile`修饰`signleton`实例变量

---

### 静态内部类模式 ###

​	创建一个静态内部类来持有一个唯一的类实例

​	只有在第一次调用`getInstance()`方法的时候, 才会实例化类

```java
public class Singleton{
    private Singleton(){
        ...
    }
    private static class SingletonHolder{
        private static final instance = new Singleton();
    }
    
    public Singleton getInstance(){
        return SingletonHolder.instance;
    }
}
```

---

### 枚举单例模式 ###

默认枚举实例的创建是线程安全的，并且在任何情况下都是单例。实际上

- 枚举类隐藏了私有的构造器。
- 枚举类的域 是相应类型的一个实例对象

```java
public enum Singleton{
    INSTANCE;
   	// any method
}
```

