# 垃圾收集器和内存动态分配策略 #

---

 [toc]

---

---

## 1 对象死亡？ ##

> **对象死亡** ：不可能再次被任何途径使用的对象
>
> 判断一个对象是否死亡，是所有对 *堆* 进行垃圾回收的基础

---

## 1.1 引用计数法 ##

> 可行，但存在缺陷，主流的JVM未采用

- 算法：在对象中添加一个引用计数器，任何时刻计数器为零的情况下，这个对象未被使用
    - 每次被引用时，计数器值加一
    - 每次引用失效时，计数器值减一
- 缺陷：有大量的例外情况需要考虑，必须配合大量的额外处理
    - 例如，如果对两个对象之间循环引用，这两个计数器都不会为0，但如果其他地方不再引用这两个对象，这时采用引用计数就无法正确回收这两个应该被回收的对象