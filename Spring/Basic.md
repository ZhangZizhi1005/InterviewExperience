# Spring 基础知识 #

---

[toc]

---

---

## IOC 和 DI ##

---

### 什么是IOC，引入IOC的好处有哪些，如何实现IOC？ ###

- **IOC** 是 **Inversion of Control** 的缩写，一般可以翻译为“控制反转“；大致观点是：借助“第三方”（通常被称为IOC容器）实现具有依赖关系的对象之间的解耦

    ​	![IOC解耦原理](ref/30131727-a8268fe6370049028078e6b8a1cbc88f.png)

    - 在不使用IOC容器的情况下，假定对象A依赖于对象B，那么对象A在初始化或者是运行到某一点的时候，必须由自身主动创建或者是引用已经创建的对象B，整个过程由对象A自身进行控制
    - 引入IOC容器后，对象A与对象B之间不存在直接的依赖，而是通过一个第三方容器进行联系，那么对象A运行到需要对象B的时候，IOC容器会创建一个对象B注入到对象A需要的地方
    - 通过这两种情形的对比，获取对象B的过程由主动行为变为了被动注入，这也就是IOC名称的由来
    - 同时，我们可以发现，DI和IOC是从不同的角度阶段描述的同一件事，即：利用IOC容器，通过依赖关系注入的方式，实现对象之间的解耦

- 引入IOC，有很多优势

    - 解开或者是降低类之间的耦合
    - 实现面向接口编程，实施依赖倒换原则
    - 提高系统的插入、修改和测试的能力

- 具体的，在Spring container中

    - 在配置过程中
        - 将bean中的依赖关系尽可能转换为关联关系
        - 将具体类的关联转换为对接口的关联
        - 将接口与实现类进行关联
    - 在需要的时候，通过Spring提供的IOC容器，根据配置信息，实例化具体的bean类，并向需要的地方注入实例

---

### 什么是DI，如何实现DI？ ###

**依赖注入(Dependenby Injection)的基本原则是 ：**  

- 应用组件不应该负责查找资源或者其他依赖的协作对象。
- 配置对象的工作应该由容器负责，查找资源的逻辑应该从应用组件中抽取处理，交给容器。

**DI是对IOC的细致诠释，即：**

- 组件之间的依赖关系由容器再运行期间决定，形象的来说，即由容器动态的将某种依赖关系注入到组件之中

**DI的实现：**

- 不采用DI时，类cA要调用接口iB中的方法，就必须在类cA中维护一个接口iB实现类cB的对象oB，这种依赖关系由开发人员维护，且除非重构项目否则不会发生变动
- 采用DI时
    - 类cA要调用接口iB中的方法，但是我们希望不对类cA和接口iB建立依赖关系
    - 而是建立关联关系
        - 在类classA中定义方法(构造器或setter方法)，用于关联接口iB
        - 将类cA、接口iB和接口iB的实现类cB放入容器中
    - 通过配置容器来实现关联关系，即在需要调用的时候，由容器动态的生成cB的对象oB，并将这个实例注入A中需要的地方

**DI的方式：**

1. 构造器注入：通常用来注入必须的依赖关系
2. setter方法注入（设值注入）：
    - 对于可选的依赖关系一般采用这种方法
    - setter注入需要类提供 无参构造器 或者 静态工厂方法 来创建实例
3. 接口注入：Spring不支持

---

---

## Bean 和 配置 ##

---

### bean的生命周期？ ###

- 在简单的Java应用中，bean的生命周期较为简单：

    - 使用```new```关键字进行bean的实例化，这个bean就可以使用了；
    - 一旦bean不再被使用，则由Java自动进行GC

- 我们可以使用Spring IOC容器管理bean的生命周期：

    <img src="https://www.javazhiyin.com/wp-content/uploads/2019/05/java0-1558500658.jpg" alt="深究Spring中Bean的生命周期" style="zoom:50%;" />

    1. 启动Spring，容器查找并加载需要被Spring管理的bean，进行bean的实例化
        - 通过构造器
        - 通过工厂方法
    2. 为bean的属性设置值或者是其他bean的引用，依赖注入的过程
        - 如果bean实现了 ```BeanNameAware```接口，Spring将调用```setBeanName()```方法，将 *bean的 id* 传入
        - 如果bean实现了 ```BeanFactoryAware```接口，Spring将调用```setBeanFactory()```方法，将 *BeanFactory容器实例* 传入
        - 如果bean实现了 ```ApplicationContextAware```接口，Spring将调用```setApplicationContext()```方法，将 *bean所在的上下文引用* 传入
    3. 调用bean的初始化方法
        - 如果bean实现了```BeanPostProcesser```接口，Spring将调用其```postProcessBefoerInitializatio()```方法
        - 如果bean实现了```InitializingBean```接口，Spring将调用其``` afterPropertiesSet()```方法
        - 如果bean使用```init-method```声明了初始化方法，该方法也会被调用
        - 如果bean实现了```BeanPostProcesser```接口，Spring将调用其```postProcessAfterInitializatio()```方法
    4. 此时，bean实例已经准备就绪，可以被使用了，它将一直驻留在应用的上下文中，直到上下文被销毁
    5. 容器关闭，调用bean的销毁方法
        - 如果bean实现了```DisposableBean```接口，Spring将调用其```destroy()```方法
        - 如果bean使用```destroy-method```声明了销毁方法，该方法也会被调用

---

### Spring中Bean的作用域？ ###

- 在Spring的早期版本，仅提供了两种作用域
    1. **singleton**： bean以单例的形式存在，即，Spring容器中只会存在一个共享的bean实例。所有对bean的请求，都只会返回这个bean的同一个实例
        - 缺省作用域
        - 一般来说，适用于无状态或者状态不可变的类
    2. **prototype:** 原型模式，即，每次对该bean请求的时候，SpringIOC都会创建一个新的作用域并返回一个新的实例
        - ```<bean ... scope="prototype"></bean>```开启
        - 一般来说，适用于有状态的bean
- 在Spring 2.X中针对```WebApplicationContext``` 新增了三种作用域
    1. **request：** 针对当前的Http请求，Spring容器会根据相关的bean来创建一个全新的实例，且该bean实例仅在当前request中有效
    2. **session：** session中的所有Http请求共享同一个bean实例，session结束后销毁实例
    3. **global session：** 全局session共享同一个bean实例

---

### BeanFactory 和 ApplicationContext 的区别？ ###

- **BeanFactory** 是Spring中最底层的接口，它的各类实现负责bean的实例化，为Spring管理bean提供依赖注入和生命周期管理支持
- **ApplicationContext** 继承自BeanFactory接口，除了以更加面向框架的方式实现了bean的实例化，它还提供了以下高级功能
    1. 国际化的消息访问：MessageSource
    2. 资源访问：URL、文件等等
    3. 事件传递：消息发送、响应机制
    4. bean的自动装配
    5. 各种应用层的上下文实现
- 区别：
    - 使用A时，如果bean配置为singleton，在启动时就会被实例化。好处在于预加载提升了运行速度，坏处在于加载速度变慢、浪费内存
    - 使用F时，只要使用该bean(getBean)时，才被实例化。优缺点正好相反，多用于移动设备的开发
    - 在没有特殊要求的情况下，多采用A，通过配置```lazy-init=true```可以实现延迟实例化bean，此外也可以实现更多F没有的功能

---

### Spring中的自动装配方式有哪些？ ###

- **-no:** 不进行自动装配，手动设置bean的依赖关系

- **-byName：**根据bean的名字进行自动装配

- **-byType：**根据bean的类型进行自动装配

- **-constructor：**类似于byType，但是这里检索的依据是构造器参数。如果一个bean正好和构造器参数相同即可装配，否则会导致错误

- **-autodetect：** 如果有默认的构造器就通过constructor方式，否则byType

    > 自动装配不如自定义装配精确，且不能自动装配基本类型、字符串等

---

### 什么是Spring注解？ ###

​	IOC容器是Spring的一个重要功能，我们可以使用IOC容器管理大量的bean。通常，我们使用 *applicationContext.xml* 文件对这些bean的配置进行管理。

​	Spring注解是一系列以 @ 开头，并规定了特定用途的单词，例如：```@Component```。通过使用注解，我们可以直接在 *类文件* 中实现原本在 *配置文件*  中进行规定的功能。

​	开启注解需要在 *applicationContext.xml*  文件中进行说明如下，否则Spring不能正常读取注解。

```xml
<beans ...>
   <bean>...</bean>
    <!-- 开启Spring的注解功能 -->
	<context:annotation-config/>
    <!-->
		配置注解扫描
		例如：component-scan指的是将所有含有@Component注解的类作为bean
		base-package指的是需要进行扫描的包
		使用了配置注解扫描语句时，可以省略开启语句
	< -->
    <context:component-scan base-package="..."/>
</beans>
```

​	Spring中的注解可以分为两类：

 1. 类级别的注解：如@Component、@Repository、@Controller、@Service以及JavaEE6的@ManagedBean和@Named注解，都是添加在类上面的类级别注解。

    Spring容器根据注解的过滤规则扫描读取注解Bean定义类，并将其注册到Spring IoC容器中。

 2. 类内部的注解：如@Autowire、@Value、@Resource以及EJB和WebService相关的注解等，都是添加在类内部的字段或者方法上的类内部注解。
    SpringIoC容器通过Bean后置注解处理器(```AnnotationBeanPostProcessor```)解析Bean内部的注解。

---

### 什么是@Autowired 和 @Resource， 它们有什么区别？ ##

**@Resource:** 

- 由J2EE提供，用来为 *修饰字段、构造函数或者```setter()```方法* ，并注入
- 有两个重要的属性：name和type
- 装配顺序
    - 如果同时指定了name和type：Spring从上下文找到唯一匹配的bean进行注入，找不到则抛出异常
    - 如果指定了name：Spring从上下文中查找名称(id)匹配的bean进行装配，找不到则抛出异常
    - 如果指定了type：Spring从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个则抛出异常
    - 如果均未指定：Spring自动按照byName方式进行匹配，如果没有匹配，则回退为一个原始类型进行匹配

**@Autowired:**

- 由Spring提供，用来为 *修饰字段、构造函数或者```setter()```方法* ，并注入
- 只按照byType方式注入

**区别：**

- A默认按照类型装配，默认情况下必须要求依赖对象存在
    - 如果要允许Null值，可以设置```required```属性为```false```
    - 如果要按照名称装配，可以结合```@Qualifier```注解使用
- R默认按照名称装配
- 推荐使用R注解在字段上
    - 可以不用再单独写```setter()```方法
    - 注解属于J2EE，减少与Spring耦合

---

---

## AOP 和 事务 ##

---

### 什么是AOP ###

**AOP** 是 *Aspect Oriented Programming* 的缩写，可以翻译为 面向切面编程

- 通过 *预编译* 方式 和 *运行期动态代理* 实现程序功能统一维护的一种技术
- AOP
    - 是OOP的延续
    - 是Spring框架的一个重要内容
    - 是函数式编程的一种衍生范型
- 利用AOP可以对业务逻辑的各个部分进行隔离，降低耦合度，提升程序可用性，同时提高了开发效率

---

### AOP有哪些基本概念 ###

AOP中有以下**基本概念**

1. **Aspect（切面）：** *Aspect* 声明类似于Java中的类声明，在一个 *Aspect* 中会包含着一些 *Pointcut* 以及相应的 *Advice*

2. **Joint Point（连接点）：** 标识程序中明确定义的点，也可以被称为所有可能被 *织入* *Advice* 的候选点

    典型的包括：

    1. 方法调用
    2. 对类成员的访问
    3. 异常处理程序块的执行
    4. etc

3. **Pointcut（切点）：** 表示一组 *Joint Point*，通过特定的规则来筛选出一组 *Joint Point*  *织入* *Advice*

    - 通过逻辑关系组合起来
    - 或者通过通配、正则等方式集中起来

4. **Advice（增强）：** *Advice* 定义了在 *Pointcut* 中定义的 *Joint Point* 具体要做的增强处理

    - 前置通知:在我们执行目标方法之前运行(**@Before**)
    - 后置通知:在我们目标方法运行结束之后 ,不管有没有异常***\*(@After)\****
    - 返回通知:在我们的目标方法正常返回值后运行***\*(@AfterReturning)\****
    - 异常通知:在我们的目标方法出现异常后运行***\*(@AfterThrowing)\****
    - 环绕通知:动态代理, 需要手动执行joinPoint.procced()(其实就是执行我们的目标方法执行之前相当于前置通知, 执行之后就相当于我们后置通知**(@Around)**

5. **Target（目标对象）：** *织入* *Advice* 的对象

6. **Weaving（织入）：** 将 *Advice* 添加到目标类具体的 *JointPoint* 上的过程，AOP有三种织入方式

    1. 编译期织入：需要特殊的Java编译期（例如AspectJ的ajc）
    2. 装载器织入：需要特殊的类加载器，在装载类的时候对类进行增强
    3. 运行时织入：在运行时为目标类生成代理实现增强（Spring采用）