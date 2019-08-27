---
title: 日志异常的问题解析
date: 2019-08-06 17:50:57
tags:
---
## 事发问题

由于线上会有日志部分丢失情况，进而进行了此次分析。

## 分解问题
由于日志中记录了大多数的日志，进而定位到此日志方法得位置。
正常能够打印出日志的最终定位于Lombok的注解：
```java
    @Slf4j
```
查看其注解注释
> 

```html
 will generate:
 <pre>
  public class LogExample {
      private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
  }
  </pre>
```

而不能够打印日志的 定位于：
```java
import org.apache.log4j.*;

private Logger logger = Logger.getLogger(getClass());
```
初步定位于俩个Logger不同而导致最终结果不一致。

查看了日志组件(阿里云) ,并无特殊之处，于是开始查询slf4j是如何实现的，希望从中查询出原因和org.apache.log4j有什么不同。

默认使用了 `com.sense.cloud.aliyun.loghub.logback.LoghubAppender` 阿里云日志组件

其默认引用了 `<include resource="org/springframework/boot/logging/logback/base.xml"/>` springboot的默认日志配置
于是查看 `org.springframework.boot.logging.logback.base.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Base logback configuration provided for compatibility with Spring Boot 1.1
-->

<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>


```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Default logback configuration provided for import, equivalent to the programmatic
initialization performed by Boot
-->

<included>
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
	<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
	<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
	<property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{yyyy-MM-dd HH:mm:ss.SSS} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

	<appender name="DEBUG_LEVEL_REMAPPER" class="org.springframework.boot.logging.logback.LevelRemappingAppender">
		<destinationLogger>org.springframework.boot</destinationLogger>
	</appender>

	<logger name="org.apache.catalina.startup.DigesterFactory" level="ERROR"/>
	<logger name="org.apache.catalina.util.LifecycleBase" level="ERROR"/>
	<logger name="org.apache.coyote.http11.Http11NioProtocol" level="WARN"/>
	<logger name="org.apache.sshd.common.util.SecurityUtils" level="WARN"/>
	<logger name="org.apache.tomcat.util.net.NioSelectorPool" level="WARN"/>
	<logger name="org.crsh.plugin" level="WARN"/>
	<logger name="org.crsh.ssh" level="WARN"/>
	<logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="ERROR"/>
	<logger name="org.hibernate.validator.internal.util.Version" level="WARN"/>
	<logger name="org.springframework.boot.actuate.autoconfigure.CrshAutoConfiguration" level="WARN"/>
	<logger name="org.springframework.boot.actuate.endpoint.jmx" additivity="false">
		<appender-ref ref="DEBUG_LEVEL_REMAPPER"/>
	</logger>
	<logger name="org.thymeleaf" additivity="false">
		<appender-ref ref="DEBUG_LEVEL_REMAPPER"/>
	</logger>
</included>

```


```xml

<?xml version="1.0" encoding="UTF-8"?>

<!--
Console appender logback configuration provided for import, equivalent to the programmatic
initialization performed by Boot
-->

<included>
	<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>${CONSOLE_LOG_PATTERN}</pattern>
			<charset>utf8</charset>
		</encoder>
	</appender>
</included>

```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
File appender logback configuration provided for import, equivalent to the programmatic
initialization performed by Boot
-->

<included>
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<encoder>
			<pattern>${FILE_LOG_PATTERN}</pattern>
		</encoder>
		<file>${LOG_FILE}</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
			<fileNamePattern>${LOG_FILE}.%i</fileNamePattern>
		</rollingPolicy>
		<triggeringPolicy
			class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
			<MaxFileSize>10MB</MaxFileSize>
		</triggeringPolicy>
	</appender>
</included>

```
这些默认配置文件看完后发现并没有什么收获，仅了解到这个日志是基于ch.qos.logback去做实现的。

而后查看引入的日志包的pom.xml文件

```xml
 <dependency>
            <groupId>com.aliyun.openservices</groupId>
            <artifactId>log-loghub-producer</artifactId>
            <version>0.1.1</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.2</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.6</version>
        </dependency>
```

大概了解到上层 api接口定义是以slf4j-api为接口定义,其可以在工程中通过使用slf4j接入不同的日志系统（解耦合的作用），而底层用logback来实现所以使用org.apache.log4j.Logger并不会进行日志输出。
slf4j在编译时负责寻找合适的日志系统并进行绑定。
LoggerFactor类中 bind()方法会寻找具体的日志实现类绑定。
```java
private final static void bind() {
        try {
            Set<URL> staticLoggerBinderPathSet = null;
            // skip check under android, see also
            // http://jira.qos.ch/browse/SLF4J-328
            if (!isAndroid()) {
                staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
                reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
            }
            // the next line does the binding 进行绑定
            StaticLoggerBinder.getSingleton();
            INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
            reportActualBinding(staticLoggerBinderPathSet);
            fixSubstituteLoggers();
            replayEvents();
            // release all resources in SUBST_FACTORY
            SUBST_FACTORY.clear();
        } catch (NoClassDefFoundError ncde) {
            String msg = ncde.getMessage();
            if (messageContainsOrgSlf4jImplStaticLoggerBinder(msg)) {
                INITIALIZATION_STATE = NOP_FALLBACK_INITIALIZATION;
                Util.report("Failed to load class \"org.slf4j.impl.StaticLoggerBinder\".");
                Util.report("Defaulting to no-operation (NOP) logger implementation");
                Util.report("See " + NO_STATICLOGGERBINDER_URL + " for further details.");
            } else {
                failedBinding(ncde);
                throw ncde;
            }
        } catch (java.lang.NoSuchMethodError nsme) {
            String msg = nsme.getMessage();
            if (msg != null && msg.contains("org.slf4j.impl.StaticLoggerBinder.getSingleton()")) {
                INITIALIZATION_STATE = FAILED_INITIALIZATION;
                Util.report("slf4j-api 1.6.x (or later) is incompatible with this binding.");
                Util.report("Your binding is version 1.5.5 or earlier.");
                Util.report("Upgrade your binding to version 1.6.x.");
            }
            throw nsme;
        } catch (Exception e) {
            failedBinding(e);
            throw new IllegalStateException("Unexpected initialization failure", e);
        }
    }
```
## 处理问题

替换 org.apache.log4j.Logger 为 org.slf4j.Logger

## 为什么

由于基于的底层并非`org.apache.log4j.Logger`而是logback所以并不会打印日志。

## 总结

很低级的问题，但确实不了解日志这块，仅为了业务开发而开发，这样的状态很差。并没有深入了解其内部逻辑。