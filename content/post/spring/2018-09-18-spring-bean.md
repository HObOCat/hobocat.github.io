---
title: "Spring基础"
aliases: 
tags: [Spring]
date: 2018-09-18
time: 17:05
---

## Spring 控制反转/依赖注入

### IOC容器

IoC容器负责实例化，配置和组装对象。 IoC容器从XML文件获取信息并相应地工作。 IoC容器执行的主要任务是:
1. 实例化
2. 应用程序类配置对象组装
3. 对象之间的依赖关系

**容器**

    BeanFactory
    ｜ 低级容器，负责加载bean，getBean
        XmlBeanFactory

    ApplicationContext
    ｜ 高级容器。依赖低级容器，同时具备别的功能
    ｜ 如fresh Bean BeanFactory等
        ClassPathXmlApplicationContext


**什么是IOC**

控制反转，即把传统上有程序代码直接操作的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。“控制反转”概念就是对组件对象控制权的转移，从程序代码转移到容器。 

**IOC有什么作用**

- 管理对象的创建和依赖关系的维护
- 解耦，有容器去维护具体的对象
- 托管了类的产生过程

**IOC分类**

- 依赖注入：详见DI
- 依赖查找

**配置方式**

1. 基于XML

`<bean id="userService" class="xx.xxx.service.impl.xxxImpl"></bean>`

bean标签详解

    1. bean标签作用：用于配置对象让spring创建；默认情况下它会调用类中的无参构造行数，如果没有无参构造行数则不能成功创建bean。 
    2. bean标签属性   
        ① id： 给对象在容器中提供唯一标识，用于获取对象。   
        ② class： 指定类的全限定类名。用于反射创建对象。   
        ③ scope： 指定对象的作用范围。

            a. singleton： 默认值 单例的（在整个容器中只有一个对象）        
            b. prototype： 多例         
            c. request：WEB项目中


2. 基于注解

当一个类添加IOC注解，这个bean的id为这个类名的的首字母小写

- 前提：包路径扫描

    <context:component-scan base-package="xx.xxx.service,xx.xxx.dao"/>

    `@ComponentScan` 作用：定义扫描的路径，从中找出标识了需要装配的类自动装配到spring的bean容器中

        1. 自定扫描路径下带有@Controller @Service @Repostitory @Component注解加入到spring容器
        2. 通过includeFilter 加入扫描路径下没有以上几个注解的类加入spring容器
        3. 通过excludeFilter过滤出不用加入到spring容器的类


    `@Component` 相当于这个类当成bean，放到IOC容器中

    `@Controller` 衍生  添加到web层 

    `@Service` 衍生  添加到service层

    `@Repository` 衍生 添加到持久化层


### DI依赖注入
#### 概念

依赖注入更加具体的实现了IOC的设计理念，是IOC的技术实现。

#### 实现原理

**配置方式**

1. 基于XML

    - 构造函数注入
    - `setter`注入
        
        集合类型 和引用类型 ref  
            引用ref
            集合



2. 基于注解
    - `@Autowired`

        默认按类型装配，可通过@Qualifiler结合去指明bean中的id（按名称）

    - `@Resource`

        默认按名称装配，通过name属性进行指明bean中的id，若没有指名name，则默认按字段名找bean，若找不到匹配的bean，则按类型装配

    - 其他
        - `@Value` 给基本类型和String 注入值  一般情况下读取配置文件属性

        - `@Scope` 指定bean的作用范围，默认singleton

        - `@PostConstruct` 写在方法上，指定此方法为初始化回调方法

        - `@PreDestroy` 写在方法上，指定此方法为销毁的回调方法


**注入方式**

- 构造方法
- setter方法
- 接口注入

**装配模式**

- `byName`

    byName模式根据bean的名称注入对象依赖项。在这种情况下，属性名称和bean名称必须相同。它在内部调用setter方法。
- `byType`

    byType模式根据类型注入对象依赖项。因此属性名称和bean名称可以不同。它在内部调用setter方法。
- `constructor`

    构造函数模式通过调用类的构造函数来注入依赖项。它会调用具有大量参数的构造函数。


**DI和IOC的区别**

- `IoC` 是 Inversion of Control 的缩写，翻译成中文是“控制反转”的意思，它不是一个具体的技术，而是一个实现对象解耦的思想。
- `DI` 是 Dependency Injection 的缩写，翻译成中文是“依赖注入”的意思。依赖注入不是一种设计实现，而是一种具体的技术，它是在 IoC 容器运行期间，动态地将某个依赖对象注入到当前对象的技术就叫做 DI（依赖注入）。

**优势**

- 使代码易于测试
- 使代码松散耦合，因此易于维护



## Spring AOP
#### 概念

Aspect Orient Programming 面向切面编程，AOP是一种编程思想。是面向对象编程（OOP）的一种补充。实现在不修改源代码的情况下，给程序动态统一添加额外功能的一种技术。
拦截指定方法并进行增强，无需侵入业务代码

### 作用

分离功能性需求和非功能性需求，可以集中处理某一个关注点，减少对业务代码的侵入，增强代码的可读性和可维护性。

### 常用场景

- 事务
- 日志
- 权限
- 监测

### 核心

**术语**

- `JoinPoint` 连接点

    指哪些被拦截到的点，在spring中只可以被动态代理拦截目标类的方法

- `PointCut` 切入点

    指要对哪些JoinPoint进行拦截，即被拦截的连接点

- `Advice` 通知

    指拦截到JoinPoint之后要做的事情，即对切入点增强的内容

- `Target` 目标

    指代理的目标对象

- `Weaving` 植入

    指把增强代码应用到目标上，生成代理对象的过程

- `Proxy` 代理

    指生成的代理对象

- `Aspect` 切面

    切入点和通知的结合


**通知分类**

- `before` 前置通知

    通知方法在目标方法调用之前执行

- `after` 后置通知

    通知方法在目标方法返回或异常后调用

- `after-returning` 返回后通知

    通知方法在目标方法返回后调用

- `after-throwing` 抛出异常通知

    通知方法在目标方法抛出异常后调用

- `around` 环绕通知

    通知方法会将目标方法封装起来


**织入时期**

- 编译期
- 类加载期
- 运行期

        spring AOP使用方式

### 日常使用

**基于Aspectj**
- XML
- 注解

    1. `@Aspect` 类上，声明当前类为切面
    2. `@Component` 并交给spring容器管理
    3. 自定义通知方法

        - `@Pointcut` 切入点

            - `execution` 表达式

                    （[方法修饰符] 返回类型  包名.类名.方法名(参与类型) [异常类型]）
                        方法修饰符  可省略
                        返回值类型、包名、类名、方法名 可以用通配符* 代表任意
                        包名与类名之间一个点 . 代表当前包下的类，两个点 .. 代表当前包下级子包下的类
                        参数列表可以使用两个点 .. 代表任意类型参数，任意个数

                    （* com.xx.xxx..*.*(..) ） xxx包及任意子包下所有方法
                    （* com.xx.xxx.*.*(..)）xxx包下所有方法
                    （* com.xx.xxx.*.yyy.*.*(..)） xxx包下任意yyy包下的任意方法

            - `within`
            - `this`
            - `target`
            - `args`

        - `@Before` 前置通知
        - `@After` 后置通知
        - `@AfterReturning` 后置通知，有返回或有异常
        - `@Arround` 环绕通知
        - `@AfterThrowing` 异常通知


    4. 配置自动代理
        - `<aop:aspectj-autoproxy/>`
        - `@Configuration + @EnableAspectAutoProxy`


spring 自带方式

### 实现原理

**动态代理实现**

- **JDK动态代理**  默认 基于接口的动态代理技术

- **CGLib代理** 基于父类的动态代理技术





## Spring 事务
### 概念
事务是逻辑上的一组操作，要么都执行，要么都不执行。
### 注意
事务能否生效，主要看数据库引擎是否支持事务

    MySQL 的innoDB引擎支持事务，myisam引擎不支持事务


### 特性  ACID

- **原子性 Atomicity**

一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在某个中间环节。若执行中发生错误，会被回滚到事务开始前的状态。

- **一致性 Consistency**

事务开始之前和事务结束之后，数据路的完成性没有被破坏。即数据是预期和期望的样子

- **隔离性 Isolation**

数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致的数据不一致。事务隔离分为不同级别

- **持久性 Durability**

事务处理结束后，对数据的修改是永久的，即系统故障也不会丢失

### 实现方式

**编程式事务管理**

        通过TransactionTemplate或TransactionManager手动管理事务

**声明式事务管理**

1. 使用@Transactional注解进行事务管理

    - 作用范围

            - 方法：推荐将注解用在方法上。另外要注意的是只有在public方法上才生效
            - 类：放在类上，表示对类中所有的public方法都生效
            - 接口：不推荐

    - 常用参数配置

        - `propagation`
        
            事务的传播行为，默认值为 REQUIRED

        - `isolcation`

            事务的隔离级别，默认值采用 DEFAULT

        - `timeout`

            事务的超时时间，默认值为-1

        - `readOnly`

            指定事务是否为只读事务，默认值为 false

        - `rollbackFor`

            用于指定能够触发事务回滚的异常类型，并且可以指定多个异常类型

        - `noRollbackFor`

            用于指定不触发事务回滚的异常类型，并且可以指定多个异常类型


    - 原理解释

        如果类或者类中`public方法上`被标注`@Transactional`注解，Spring容器会在启动时为其创建一个代理类，在调用被注解的public方法时，`实际调用的是TransactionInterceptor类中的invoke()方法。这个方法的作用就是在目标方法之前开启事务，方法执行过程中若遇到异常时回滚事务。`方法调用完成之后提交事务。
            TransactionInterceptor 类中的 invoke()方法内部实际调用的是 TransactionAspectSupport 类的 invokeWithinTransaction()方法



2. 推荐使用:代码侵入性小，基于AOP实现
3. AOP自调用问题

    - 同一类中其他没有`@Transactional`注解的方法内部调用有@Transactional注解的方法，有注解的方法事务会失效
    - 只有当 `@Transactional` 注解的方法在`类以外`被调用的时候，Spring 事务管理才生效。



### 核心接口

#### PlatformTransactionManager`

    事务管理器，Spring事务策略的核心。提供了不同平台的事务管理器如JDBC、Hibernate、JPA等

**主要方法**

- `TransactionStatus getTransaction(TransactionDefinition var)` 获取事务
- `void commit(TransactionStatus var)` 提交事务
- `void rollback(TransactionStatus var)` 回滚事务


#### TransactionDefinition

**事务定义信息**

- 隔离级别
    - `TransactionDefinition.ISOLATION_DEFAULT`
        使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ 隔离级别 Oracle 默认采用的 READ_COMMITTED 隔离级别

    - `TransactionDefinition.ISOLATION_READ_UNCOMMITTED`
        最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读

    - `TransactionDefinition.ISOLATION_READ_COMMITTED`
        允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

    - `TransactionDefinition.ISOLATION_REPEATABLE_READ`
        对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生

    - `TransactionDefinition.ISOLATION_SERIALIZABLE`
        最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。


- 传播行为
    1. `TransactionDefinition.PROPAGATION_REQUIRED`
    2. `TransactionDefinition.PROPAGATION_REQUIRES_NEW`
    3. `TransactionDefinition.PROPAGATION_NESTED`
    4. `TransactionDefinition.PROPAGATION_MANDATORY`

- 超时

    指一个事务所允许执行的最长时间，若超过该时间限制事务还没有完成，则自动回滚事务。 单位秒，默认值-1

- 只读

    只读数据查询的事务，不涉及数据的修改。适用于复杂业务，多条多次查询

- 回滚规则

    定义遇到哪些异常才会事务回滚。默认RuntimeException及其子类回滚


**主要方法**

- `int getPropagationBehavior();`
    返回事务的传播行为，默认值为 REQUIRED。

- `int getIsolationLevel();`
    返回事务的隔离级别，默认值是 DEFAULT

- `int getTimeout();`
    返回事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

- `boolean isReadOnly();`
    返回是否为只读事务，默认值为 false



#### TransactionStatus
**事务运行状态**

**主要方法**

- `boolean isNewTransaction()`
    是否是新的事务

- `boolean hasSavepoint()`
    是否有恢复点

- `void setRollbackOnly()`
    设置为只回滚

- `boolean isRollbackOnly()`
    是否为只回滚

- `boolean isCompleted`
    是否已完成


## springframework 优点
预定义模板、松耦合、易于测试、轻巧、快速开发、声明式支持

![springframework 优点](/img/spring/springframework.png)






## Spring bean 生命周期流程图

![spring bean 生命周期](/img/spring/spring-bean-life-cycle.png)

## SpringMVC
    
### 概念

    基于spring的MVC框架

### 作用

    解决页面代码与后台代码分离，解耦合

### 执行过程 or 原理

![springmvc 执行过程](/img/spring/spring-mvc-order.png)

对servlet的封装，更易于使用

**流程**

1. 用户发出请求被前端控制器DispatcherServlet拦截处理
2. DispatcherServlet收到请求调用HandlerMapping（处理器映射器）
3. HandlerMapping找到具体的处理器(查找xml配置或注解配置)，生成处理器对象及处理器拦截器(如果有)，再一起返回给DispatcherServlet
4. DispatcherServlet调用HandlerAdapter（处理器适配器）
5. HandlerAdapter经过适配调用具体的处理器（Handler/Controller）
6. Controller执行完成返回ModelAndView对象
7. HandlerAdapter将Controller执行结果ModelAndView返回给DispatcherServlet
8. DispatcherServlet将ModelAndView传给ViewReslover（视图解析器）
9. ViewReslover解析ModelAndView后返回具体View（视图）给DispatcherServlet
10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）
11. DispatcherServlet响应View给用户

**组件**

    1.DispatcherServlet 前端控制器
        作用：接收请求，响应结果，相当于转发器，中央处理器。
        由框架提供，在web.xml中配置

    2.HandlerMapping 处理器映射器
        作用：根据请求的url查找Handler（处理器/Controller）
        通过XML或注解方式映射

    3. HandlerAdapter 处理器适配器
        作用：按照特定规则（HandlerAdapter要求的规则）执行handler中的方法

    4.Handler 处理器
        作用：接收用户的请求信息，调用业务方法处理请求，也称为后端控制器
        即Controller

    5.ViewResolver 视图解析器
        作用：视图解析，把逻辑视图解析成物理视图
        springMVC提供多种技术 如  jstlView freemarkerView等

    6. view  视图
        作用：把数据展现给用户页面
        View是个接口，实现类支持多种技术  如 jsp  freemarker PDF等




**使用**

1. web.xml 中配置DispatcherServlet
2. 编写Controller
3. 编写页面


## springBoot Application创建过程

SpringApplication.run() 源码解析

1. 创建SpringApplication实例

        1. 新建实例
            创建SpringApplication对象

        2. 实例初始化
            1. 设置resourceLoader
            2. 确定webApplicationType类型
            3. 设置ApplicationContextInitializer。遍历执行所有通过SpringFactoriesLoader 可以查找到的、并加载的ApplicationContextInitializer，
            4. 设置ApplicationListener。 遍历执行所有通过SpringFactoriesLoader 可以查找到的、并加载的ApplicationListener
            5. 设置mainApplicationClass


2. 执行run方法

    1. 设置系统环境变量 java.awt.headless
        默认为true

    2. 加载并执行SpringApplicationRunListener
        遍历执行所有通过SpringFactoriesLoader 可以查找到的、并加载的SpringApplicationRunListener

    3. 遍历所有listeners，调用listener的starting方法
    4.获取默认的ApplicationArguments
        new DefaultApplicationArguments(args);

    5. 创建并配置当前SpringBoot应用将要使用的Environment，包括配置要使用的ProjectySource及Profile。ConfigurableEnvironment environment = prepareEnvironment(listeners,      applicationArguments);
        2.根据webApplicationType创建Environment
        3.配置Environment的参数argumrnts（ApplicationArguments）
        4. listeners遍历执行environmentPrepared方法
            表示SpringBoot应用的Environment准备好了
            每个监听器将给application添加事件

        5. 将Environment绑到SpringApplication    
            bindToSpringApplication(environment);


    6. 获取是否配置了忽略的bean，设置环境变量spring.beaninfo.ignore。
        configureIgnoreBeanInfo(environment);

    7. 若SpringApplication的showBanner属性被设置为true，则打印banner
        printBanner(environment)

    8. 根据用户是否设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为SpringBoot应用创建什么类型的ApplicationContext并创建完成。
        createApplicationContext();
            contextClass为空，则依据webApplicationType创建不同类型的contextClass的类型。


    9. 加载SpringBootExceptionReporter
        遍历执行所有通过SpringFactoriesLoader 可以查找到的、并加载的SpringBootExceptionReporter

    10. SpringApplicationContext进一步处理
        prepareContext()
            1. ApplicationContext创建好之后，SpringApplication再次借助springFactoriesLoader，查找并加载classpath中所有可用的Application-Initializer
            2. 遍历调用这些ApplicationContextInitializer的initalize方法来已经创建好的ApplicationContext进行进一步处理
            3. 遍历调用所有SpringApplicationRunListeners的contextPrepared方法
            4.  加载特殊的单例bean
            5.  将之前通过@EnableAutoConfiguration获取的所有配置的以及其他形式的IOC容器配置 （bean）加载到Context上下文
            6.  遍历调用SpringApplicationRunListeners的contextLoaded方法


    11. 调用ApplicationContext的refresh()方法
        refresh方法
            1. 准备对Context进行刷新prepareRefresh(); 
            2.  告诉子类刷新内部工厂obtainFreshBeanFactory();
            3. 预备工厂给Context使用prepareBeanFactory(beanFactory);
            4. 允许在context子类中对BeanFactory进行后续处理 postProcessBeanFactory(beanFactory);
            5.  调用在context中注册bean的工厂处理器  invokeBeanFactoryPostProcessors(beanFactory);
            6. 注册拦截bean创建的拦截处理器bean   registerBeanPostProcessors(beanFactory); 
            7.  初始化context的消息源 initMessageSource();
            8.  初始化context的multicaster事件   initApplicationEventMulticaster();
            9. 初始化特殊的bean给特定的context    onRefresh();
            10.  查找监听器bean并注册他们 registerListeners();
            11. 实例化所有剩余的非懒加载的单例bean。 finishBeanFactoryInitialization(beanFactory);
            12.  发布相应的事件finishRefresh();

        核心。工厂类
        根据registerShutdownHook判断是否需要添加shutdownHook
        完成IOC容器可用的最后一道工序

    12. 遍历调用所有SpringApplicationRunListeners的started()方法
    13. 查找当前ApplicationContext中是否注册有CommandLineRunner，若有，则遍历执行他们
    14.  正常情况下，遍历调用执行SpringApplicationRunListener的running方法
    15. 若出现异常，则处理异常，遍历并调用SpringApplicationRunListener的failed方法


## Autoconfiguration 原理解析

@EnableAutoConfiguartion

    @AutoConfigurationPackage
        注册向指明了包路径的包。若未指明，则注册带注解的包

    @Import
        AutoConfigurationImportSelector
            查找 META-INF/spring.factories
                获取到配置信息
                    获取到具体的XXXAutoConfigurtaion类
                        @Configuration
                            指明一个配置类

                        @ConditionalOnClass
                            指定类，只有当注解指定的类存在时，才加载

                        @AutoConfigureAfter 
                            非必须 在该注解指定的自动配置类之后注册

                        @AutoConfigureBefore
                            非必须在该注解指定的自动配置类之前注册

                        @AutoConfigureOrder
                            非必须指定自动配置类的注册顺序int。值越大级别越高 

                        @EnableConfigurationProperties
                            指定具体的配置类  XXXProperties类。如TransactionProperties
                                @ConfigurationProperties
                                    获取项目配置文件中对应的配置值  如 spring.redis

                                其实是一个pojo类，有默认值


                        @Bean method注入对应的bean


                autoconfigure包已经集成常用的自动配置类。所以我们才能简单的使用组件
                若自己集成的话，也是参照这个逻辑




