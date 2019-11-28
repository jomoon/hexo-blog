---
title: howto-database-initialization
date: 2019-09-28 09:18:12
categories:
- translation
- spring
- database
---
translation from [springboot/reference](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/howto-database-initialization.html)


# 85. Database Initialization 数据库初始化
SQL数据库初始化可以根据你的不同需求来初始化。自然，你可以手工操作，提供数据库以分离的手段。建议你使用简单的机制来生成。

## 85.1 Initialize a Database Using JPA 通过JPA来初始化数据库
JPA有DDL语句生成的特征，这些可以在数据库启动时建立起来。这个通过额外的两个属性来控制：
- `spring.jpa.generate-ddl` (boolean) 生成DDL的特征的开关，此与厂家无关。
-  `spring.jpa.hibernate.ddl-auto` (enum) 是Hibernate的特性，以更细粒度的方式控制。这个特性会在后续的指引中描述的更加清楚。

## 85.2 Initialize a Database Using Hibernate 通过使用Hibernate来初始化数据库
你可以明确地设置`spring.jpa.hibernate.ddl-auto`标准的Hibernate属性值用以`none`, `validate`, `update`, `create`和`create-drop`.
springboot根据你选择嵌入的数据库来选择默认值。如果是no schema 的数据库那么默认值是`create-drop`或是其他情况下用`none`.嵌入式数据库通过`Connection`类型来判断。`hsqldb`, `h2`, 和 `derby` 是嵌入式的，其他并不是。当你从内存数据库切换到真实数据库时，您需假象这些数据表应该存在与新的平台（真实数据库）。你即可以明确设置`ddl-auto`或是使用其他机制来初始化数据库。

> 你可以通过激活`org.hibernate.SQL`来激活输出。如果你是在debug模式下，这个是自动打开的。

另外，在根路径下的命名为import.sql的文件会在启动时被执行如果Hibernate回从头创建结构(如果`ddl-auto`设置成`create` 或 `create-drop`)。这个对于demos或是测试来说十分有用但如果你放在生产类路径上你需警惕。这是Hibernate的特性（与Spring无关）。

## 85.3 Initialize a Database 初始化数据库
Springboot 可以自动创建结构（DDL脚本）来创建表并通过（DML脚本）。他从标准类路径的位置:`schema.sql` 和 `data.sql` 依次读取。另外Springboot也处理`schema-${platform}.sql` 和 `data-${platform}.sql`文件，`platform`根据`spring.datasource.platform`来判断。他允许你去切换数据库如果必要的话，例如，你可能回选择设置入厂商名称入(`hsqldb`, `h2`, `oracle`, `mysql`, `postgresql`等等)。

> Springboot为内嵌数据库自动创建了结构。这一特性可以通过`spring.datasource.initialization-mode`来定制。如果你想要总是初始化`DataSource`那么你需要使用`spring.datasource.initialization-mode=always`

默认，Springboot对于Spring JDBC 初始化工具开启快速出错的特性，这意味着，如果你的脚本导致了异常，应用也会启动失败。你可以通过调整这一行为通过
`spring.datasource.continue-on-error`配置。

> 在基于JPA的应用中，你可以选择让Hibernate来创建结构或使用`schema.sql`,但如果你不能俩者一起使用，确保禁用`spring.jpa.hibernate.ddl-auto`如果你使用了`schema.sql`。

## 85.4 Initialize a Spring Batch Database 初始化Spring批次数据库

如果你使用Spring Batch，他给主流的数据库平台做了预包装。Spring boot 可以检测你的数据库类型并在启动时执行那些脚本，如果你使用了嵌入式数据库，这也会默认开启。你可以对任意数据库类型来进行激活，如下配置：
`spring.batch.initialize-schema=always`
你可以关掉初始化通过设置`spring.batch.initialize-schema=never`。

## 85.5 Use a Higher-level Database Migration Tool 使用高层的数据迁移工具
Springboot 支持俩个高层的迁移工具 Flyway 和 Liquibase.

### 85.5.1 Execute Flyway Database Migrations on Startup 在应用启动时执行Flyway数据库迁移

为了在应用启动时自动跑Flyway数据库迁移，你需要将`org.flywaydb:flyway-core`到类路径上。
这个迁移脚本有如下格式`V<VERSION>__<NAME>.sql`(通过`<VERSION>`是以下划线分开的版本，例如'1','2_1')。默认他们会放在`classpath:db/migration`文件下，但你可以修改`spring.flyway.locations`的配置来修改默认路径。这个是一个逗号分割的列表`classpath: `or `filesystem:`位置。例如，如下配置即可以搜索默认的类路径位置可以用文件系统的`/opt/migration`文件夹。
`spring.flyway.locations=classpath:db/migration,filesystem:/opt/migration`
你可以增加一些特殊的如`{vendor}`占位符来使用特定厂商的脚本。假设如下:
`spring.flyway.locations=classpath:db/migration/\{vendor}`
而非使用`db/migration`,前者配置文件夹来使用数据库类型（如 `db/migration/mysql`) 。支持的库在`DatabaseDriver`枚举类中。

`FlywayProperties`提供了大多数Flyway的配置和一系列小的额外属性可以禁用迁移或关闭掉位置校验。如果你需要对配置进行更深的控制，考虑注册一个`FlywayConfigurationCustomizer`实例bean。

Springboot 调用Flyway.migrate()来执行数据库的迁移，如果你想要更多控制，写一个类作为Bean`@Bean`实现`FlywayMigrationStrategy`.

Flyway提供SQL和Java的回调方法。想使用基于SQL的回调，将所有回调脚本放置在`classpath:db/migration`文件夹下。想使用基于java的回调，创建一个或多个实例实现Callback接口.任何这样的实例都会注册到Flyway上。他们可以通过`@Order`或实现`Ordered`接口来排序。实例实现废弃的接口`FlywayCallback`也可以被检测到，然而他们不能和`Callback`实例一起使用。

默认，Flyway自动装配数据源在你的上下文中且使用他们来进行迁移。如果你想要使用不同的数据源，你可以创建一个bean标记为`@FlywayDataSource`。如果你这样做的话，记住你需要将一个标记为`@Primary`。可替代的，你可以通过设定`spring.flyway.[url,user,password]`使用Flyway的本地数据库。随意设置`spring.flyway.url`或`spring.flyway.user`使得Flyway使用其自身的`DataSource`。如果这三个属性都没有被设置，其等值的`spring.datasource`属性将被使用。

这里有一个[Flyway](https://github.com/spring-projects/spring-boot/tree/v2.1.7.RELEASE/spring-boot-samples/spring-boot-sample-flyway)的样例，你可以查看其如何做到的。

你可以使用Flyway来为特殊场景提供数据。例如，你可以方式特定测试的迁移在`src/test/resources`他们仅会在你的应用开始的时候。你也可以使用特定属性`spring.flyway.locations`配置来定制位置，明确迁移当特定的配置被激活。例如：在`application-dev.properties`,你可以指定如下配置：
`spring.flyway.locations=classpath:/db/migration,classpath:/dev/db/migration`
在装配时，`dev/db/migration`迁移仅会在dev属性被激活的情况下执行.

### 85.5.2 Execute Liquibase Database Migrations on Startup 启动时使用Liquibase数据库迁移
在应用启动时为了使应用启动时启动Liquibase数据库迁移,需要将`org.liquibase:liquibase-core`引入你的类路径下。

默认，主要变更日志时从`db/changelog/db.changelog-master.yaml`这里读取，但是你可以通过使用`spring.liquibase.change-log`来改变位置。除了YAML，Liquibase也支持了JSON,XML和SQL改变日志格式。

默认情况下，Liquibase在你的上下文装配(@Primary)数据源，并且使用其来做迁移。如果你需要使用不同的`DataSource`,你可以创建一个并把其标记为`@LiquibaseDataSource`。如果你这样做了，你需要两个不同的数据源，记住将另一个标记为`@Primary`。你也可以通过设置`spring.liquibase.[url,user,password]`的额外配置使用Liquibase本地数据源。如果这三个属性都没有被设置，其等值的`spring.datasource`属性将被使用。

查看[`LiquibaseProperties`](https://github.com/spring-projects/spring-boot/tree/v2.1.7.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/liquibase/LiquibaseProperties.java)查看上下文可用的细节和默认的结构。

如下有 [Liquibase sample](https://github.com/spring-projects/spring-boot/tree/v2.1.7.RELEASE/spring-boot-samples/spring-boot-sample-liquibase)来查看其是如何将所有都组装好的。