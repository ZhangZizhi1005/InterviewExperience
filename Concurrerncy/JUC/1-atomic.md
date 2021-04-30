# Java.util.concurrent.atomic #

---

[toc]

---

## 参考资料 ##

[1](https://blog.csdn.net/weixin_34162695/article/details/91446373)

[2](https://blog.csdn.net/panweiwei1994/article/details/78646390)

---

## 概述 ##

---

### 分类 ###

atomic包中类可以分为以下五大类

> - 基本类型：AtomicBoolean、AtomicInteger、AtomicLong
> - 引用类型：AtomicReference、AtomicStampedRerence、AtomicMarkableReference
> - 数组：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
> - 对象的属性：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater
> - JDK1.8新增：DoubleAccumulator、LongAccumulator、DoubleAdder、LongAdder

1. [基本类型](## 基本类型 ##):
    - 基本类型实例各自提供对相应类型单个变量的原子方式访问和更新功能
        - 例如AtomicBoolean提供对int类型单个变量的原子方式访问和更新功能
    - 每个类也为该类型提供适当的实用工具方法
        - 例如，类AtomicLong和AtomicInteger提供了原子增量方法，可以用于生成序列号
2. 引用类型:
    - AtomicStampedRerence维护带有整数“标志”的对象引用，可以用原子方式对其进行更新
    - AtomicMarkableReference维护带有标记位的对象引用，可以原子方式对其进行更新
3. 数组:
    - 进一步扩展了原子操作，对这些类型的数组提供了支持
        - 例如AtomicIntegerArray是可以用原子方式更新其元素的int数组
4. 对象属性:
    - 基于反射的实用工具，可以提供对关联字段类型的访问
        - 例如AtomicIntegerFieldUpdater可以对指定类的指定volatile int字段进行原子更新
5. 新增:
    - DoubleAccumulator、LongAccumulator、DoubleAdder、LongAdder是JDK1.8新增的部分，是对AtomicLong等类的改进
        - 比如LongAccumulator与LongAdder在高并发环境下比AtomicLong更高效

---

### 原子类是否可以替代锁 ###

不可以, 当且仅当对象的重要更新限定于单个变量时才使用

---

### 原子类和Integer类的区别 ###

原子类不提供hashCode和CompareTo等方法,因为原子变量是可变的

---

---

