---
title: "Spring基础"
aliases: 
tags: [Spring]
date: 2018-09-18
time: 17:05
---
## Spring bean 生命周期流程图

![spring bean 生命周期](/img/spring/spring-bean-life-cycle.png)


## SpringMVC 执行过程 or 原理

![springmvc 执行过程](/img/spring/spring-mvc-order.png)

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




