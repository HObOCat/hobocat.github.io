---
title: "Java Log"
aliases: 
tags: [Log]
date: 2024-07-03
time: 17:46
---
## 常用日志框架

- **Log4j**
    Apache Log4j 是一个基于 Java 的日志记录工具。它是由 Ceki Gülcü首创的，现在则是 Apache 软件基金会的一个项目。 Log4j 是几种 Java 日志框架之一。
- **Log4j 2**
    Apache Log4j 2 是 apache 开发的一款 Log4j 的升级产品。
- **Commons Logging**
    Apache 基金会所属的项目，是一套 Java 日志接口，之前叫 Jakarta Commons Logging，后更名为 Commons Logging。
- **Slf4j** 
    类似于 Commons Logging，是一套简易 Java 日志门面，本身并无日志的实现。（Simple Logging Facade for Java，缩写 Slf4j）。
- **Logback**
    一套日志组件的实现（Slf4j 阵营）。
- **Jul**
    Java Util Logging，自 Java1.4 以来的官方日志实现。

## 日志结构框架

### 日志门面

门面设计模式，它只提供一套接口规范，自身不负责日志功能的实现，目的是让使用者不需要关注底层具体是哪个日志库来负责日志打印及具体的使用细节。目前使用广泛的有两种：`Slf4J`  和 `Common Logging`

### 日志实现

他具体实现了日志的相关功能，主流的三个 `Log4J`、`logback` 、`log-jdk`。

### 日志适配器

    日志适配器分两种场景：桥接模式-即中间桥梁，把两方连接起来

1. 日志门面适配器：

    因为 slf4j 是后提出来的，在此之前的日志库是没有实现 slf4j 接口的，所以额外需要一个适配器解决接口不兼容问题。

2. 日志库适配器：

    老的工程里，已经使用了别的日志库的 api 打印日志，要将日志库模式改为 slf4j+日志库模式，但是项目中使用老的 api 打印日志的地方太多，无法全改，此时需要一个适配器完成从就日志 api 到 slf4j 的路由，使得在不改动原有代码的情况下，升级为 slf4j 规范统一管理日志。

### 为什么要用日志门面？

日志门面提供了统一接口规范，使用者只需使用门面，而不需要关心具体的实现。另一方面，项目中引入别的依赖项目或 jar，而此依赖使用别的日志框架，则会使项目变得难以维护

![日志框架结构](/img/log/log-struct.png)

## 常用日志框架的使用及配置

    新项目推荐使用  slf4j + logback 组合

### Slf4J

**Slf4j 用法**

Slf4j 与其它日志组件的关系说明

- Slf4j 的设计思想比较简洁，使用了 Facade 设计模式，Slf4j 本身只提供了一个 slf4j-api-version.jar 包，这个 jar 中主要是日志的抽象接口，jar 中本身并没有对抽象出来的接口做实现。
- 对于不同的日志实现方案（例如 Logback，Log4j...），封装出不同的桥接组件（例如 logback-classic-version.jar，slf4j-log4j12-version.jar），这样使用过程中可以灵活的选取自己项目里的日志实现。

Slf4j 与其它日志组件调用关系图

![Slf4j-api](/img/log/Slf4j-api.png)

![Slf4j-api](/img/log/Slf4j-api2.png)

**项目集成**

```xml
<!--只有slf4j-api依赖-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>${slf4j-api.version}</version>
</dependency>
```

```java
private static final Logger log = LoggerFactory.getLogger(XXX.class);
// log 使用static修饰，代表跟当前类绑定，避免每次都new 一个新对象
```

    另外，使用 slf4j+日志库模式时，一定得注意 jar 或日志库冲突问题，可能导致日志打印失效。
    
### Log4j

先于 slf4j 出现的日志实现，若想使用 slf4j 的接口规范，需要中间桥接 slf4j-log4j12

**项目集成**
```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>${slf4j-log4j12.version}</version>
</dependency>
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>${log4j.version}</version>
</dependency>
```

**配置文件 log4j.xml | log4j.properties**

```properties
# [level], [appenderNames]  eg : DEBUG, stdout,D,E
log4j.rootLogger = DEBUG, stdout,D,E
# 配置日志信息输出目的地
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
# Target是输出目的地的目标
log4j.appender.stdout.Target = System.out
# 指定日志消息的输出最低层次
log4j.appender.stdout.Threshold = INFO
# 定义名为stdout的输出端的layout类型
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
# 如果使用pattern布局就要指定的打印信息的具体格式ConversionPattern
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss} %l%m%n

#log4j.logger.com.camore.copy = DEBUG,stdout

# 名字为D的对应日志处理
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
# File是输出目的地的文件名
log4j.appender.D.File = LOG//app_debug.log
#false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG
log4j.appender.D.layout = org.apache.log4j.TTCCLayout

# 名字为E的对应日志处理
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File = LOG//app_error.log
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

### Logback

与 log4j 同一作者，是 log4J 的升级，具备比 log4j 更多的优点。后于 slf4j 接口规范开发，所以直接实现了 slf4j 的接口。

logback 当前分为 3 个模块 `logback-core`，`logback-classic`， `logback-access`

- **logback-core** 是其他模块的基础
- **logback-classic** 是 log4j 的改良，本省实现了 slf4j 的接口
- **logback-access** 访问模块与 servlet 容器集成提供通过 http 来访日志的功能

> `logback 组件`

- **Logger**：日志的记录器，主要用于存放日志对象，也可以定义日志类型、级别
- **Appender**：用于指定日志输出的目的地，可以是 控制台、文件、数据库等
- **Layout**：负责把事件转成字符串，格式化的日志信息的输出。在 logback 中 layout 对象被封装成 encoder 中

**项目集成**

```xml
<!--logback-classic依赖logback-core，会自动级联引入-->
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>${logback-classic.version}</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>${logback-core.version}</version>
</dependency>
```

**配置文件  logback.groovy | logback-test.xml | logback.xml**

加载顺序 `logback-test.xml` > `logback.groovy` > `logback.xml` > `BasicConfigurator`（默认配置）

![logback](/img/log/logback-struct.png)

配置详情

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- 配置文件修改时重新加载，默认true -->
<configuration scan="true">
    
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="CATALINA_BASE" value="**/logs"></property>
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder charset="UTF-8">
            <!-- 输出日志记录格式 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
 
    <!-- 第一个文件输出,每天产生一个文件 -->
    <appender name="FILE1" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 输出文件路径+文件名 -->
            <fileNamePattern>${CATALINA_BASE}/aa.%d{yyyyMMdd}.log</fileNamePattern>
            <!-- 保存30天的日志 -->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder charset="UTF-8">
            <!-- 输出日志记录格式 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
 
    <!-- 第二个文件输出,每天产生一个文件 -->
    <appender name="FILE2" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${CATALINA_BASE}/bb.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${CATALINA_BASE}/bb.%d{yyyyMMdd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder charset="UTF-8">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="CUSTOM" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${CATALINA_BASE}/custom.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- daily rollover -->
            <fileNamePattern>${CATALINA_BASE}/custom.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- keep 30 days' worth of history -->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder charset="UTF-8">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- 设置日志输出级别 -->
    <root level="ERROR">
        <appender-ref ref="CONSOLE" />
    </root>
    <logger name="file1" level="DEBUG">
        <appender-ref ref="FILE1" />
    </logger>
    <logger name="file1" level="INFO">
        <appender-ref ref="FILE2" />
    </logger>
    <!-- 自定义logger -->
    <logger name="custom" level="INFO">
        <appender-ref ref="CUSTOM" />
    </logger>
</configuration>

```