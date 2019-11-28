---
title: howto-logging
date: 2019-11-27 10:11:17
tags:
categories:
- logging
- spring
- translation
---

translation from [springboot/reference](https://docs.spring.io/spring-boot/docs/1.5.12.RELEASE/reference/html/howto-logging.html)

# 76.Logging 打日志

SpringBoot没有强制的日志依赖，除了通用日志接口（许多实现都选择的）。想使用Logback你需要将`logback`和`jcl-over-slf4j`(其实现了通用日志接口)放入类路径下。最简单的方式通过使用staters其全依赖于`spring-boot-starter-logging`。对于网页应用来说你仅需要`spring-boot-starter-web`，因为他依赖了日志starter。例如，使用maven：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Springboot 有日志系统抽象，其试图通过基于类路径上的内容来配置日志。如果Logback是可用的，那么他就作为了首选。
如果你仅想对日志的级别进行改变，你仅需在`application.properties`中使用`logging.level`例如：

```xml
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```
你也可以通过使用`logging.file`设置文件位置。

为配置一个更细粒度的日志系统配置，你需要使用原生的配置格式来配置`LoggingSystem`。默认Springboot通过默认的配置如（Logback：classpath：logback.xml）来提升日志系统的原生配置，你也可以`logging.config`的属性来配置其路径。

## 76.1 Configure Logback for logging 为日志配置Logback

如果你在你的类路径下放入`logback.xml`他会从这个配置中获取（或是通过利用Boot提供的模板特性 使用`logback-spring.xml`）。SpringBoot提供了一个默认的配置，如果你只是想改level等。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <logger name="org.springframework.web" level="DEBUG"/>
</configuration>
```

如果你查看在Spring-boot.jar中的`base.xml`，你可以看到他使用一些有用的系统属性为你创建的。

- ${PID} 现在的进程ID
- ${LOG_FILE} 如果`logging.file` 已经设置如Boot的外部配置中
- ${LOG_PATH} 如果`logging.path` 已经被设置了
- ${LOG_EXCEPTION_CONVERSION_WORD} 如果`logging.exception-conversion-word` 已经设置入Boot的外部配置中

Spring Boot也提供了一些在控制台ANSI有颜色的命令输出（但不在日志文件中）使用定制的Logback转换器，查看细节通过默认的`base.xml`。

如果Groovy在类路径下，你也可以配置Logback也通过logback.groovy（如果他出现，他会是优选的）

### 76.1.1 Configure logback for file only output 配置Logback仅文件输出

如果你想要配置禁用控制台日志和写如文件，你需要一个定制的`logback-spring.xml`导入`file-appender.xml`但不引入`console-appender.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}/}spring.log}"/>
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <root level="INFO">
        <appender-ref ref="FILE" />
    </root>
</configuration>
```
你也需要将`logging.file`加入你的`application.properties`:
`logging.file=myapplication.log`

## 76.2 Configure Log4j for logging 
springboot 支持log4j2为日志配置如果他在类路径下。如果你使用starters来装配依赖那意味着你必须排除Logback且使用log4j2来做代替。如果你不用starters你至少需要提供`jcl-over-slf4j`。

最简单的可能是通过使用starters，虽然他需要一些排除的方式：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

> 使用Log4jstarter聚合所有通用日志需求（例如包含Tomcat使用`java.util.logging`通过使用Log4j2配置输出），看 Log4j 2的样品来看起是怎么运作的。

> 为保证debug的日志通过使用`java.util.logging`路由至Log4j 2,配置JDK logging adapter来设置java.util.logging.manager 系统属性 to org.apache.logging.log4j.jul.LogManager。


### 76.2.1 Use YAML or JSON to configure Log4j 2 使用yaml或json来配置Log4j 2

除了默认的XML配置格式之外，Log4j 2也支持YAML和json配置文件。通过使用可替代的配置文件格式来配置 Log4j 2,增加合适的依赖来命名配置文件来匹配你的选择的文件格式

yaml `com.fasterxml.jackson.core:jackson-databind`,`com.fasterxml.jackson.dataformat:jackson-dataformat-yaml` `log4j2.yaml`,`log4j2.yml`


json `com.fasterxml.jackson.core:jackson-databind` `log4j2.json` `log4j2.jsn`