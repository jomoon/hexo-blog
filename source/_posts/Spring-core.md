---
title: Spring core Translation 
categories:
- spring core
- translation
toc: true
---
translation from [spring/core](https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/core.html#spring-core)

## 核心技术

这部分参考文档包含了SpringFramework所有必须的技术。

其中最重要的是SpringFramework的IOC控制反转。Spring Framework的控制反转容器被全覆盖的Spring AOP技术所覆盖。SpringFramework有他自己的切面框架，这个切面框架易于理解且在java企业开发中有80%语法糖。

spring通过AspectJ来全面覆盖AOP(最为丰富的 - 在特征方面 - 且是在java企业开发中最成熟的AOP实现）

### 控制反转容器

这一章包含Spring的控制反转(IoC)容器

#### 介绍Spring控制反转容器和实例
  介绍Spring控制反转容器和实例。这一章包含了控制反转(IoC)的实现：Spring框架。控制反转也被人所知为依赖注入(DI)。这是一个凭借对象和其依赖（依赖意味着其他与之一起工作的对象）仅通过构造参数，工厂方法参数，或是构造之后，从工厂返回的实例将属性塞入。容器而后要创建实例时注入了这些所需的依赖。这个过程基本上就是反转(因此其名字叫做，控制反转)实例本身，控制安装或通过使用直接构造器或其他机制例如（Service Locator pattern）定位其依赖。

`org.springframework.beans` 和 `org.springframework.context` 包是Spring框架控制反转容器的基础包。`BeanFactory`接口提供了 一个增强的配置文件机制且有能力管理任何类型的对象。`ApplicationContext` 是`BeanFactory`其子接口。 他增加了如下：

与Spring的AOP特征更容易的整合。

消息资源处理(在于使用国际化时)

事件发布

应用层具体上下文如`WebApplicationContext`在web应用中。

简短来说`BeanFactory`提供了配置框架和基本的功能，而`ApplicationContext`增加了更多企业具体的功能。`ApplicationContext`就是一个完整的集合`BeanFactory`，且作为这一章控制反转的重点描述。想要查看更多信息关于使用`BeanFactory` ，去看`BeanFactory`。

在Spring中，那些作为你应用的核心组成的且被SpringIoC所管理的对象叫做组件。一个组件是之那些被实例化的，装配的或是被Spring IoC容器所管理的。然而，一个组件可能仅仅是你应用中多个对象中的一个。组件们和他们所需要的依赖，都将解析到配置元数据中，给容器使用。


#### 容器总览

`org.springframework.context.ApplicationContext`接口代表着Spring IoC容器并且对组件的安装，配置，组装负责。容器拥有如何初始化，配置和装配此实例的说明（通过读取配置元数据）。配置元数据表现在XML，Java注解，或是java代码中。这些配置是用作于编排你的应用中实例的相互依赖关系。

`ApplicationContext` 几个被Spring所提供的实现。在独立运行的应用中，经常使用的是
`ClassPathXmlApplicationContext`或是 `FileSystemXmlApplicationContext`。当XML已经为被定义的配置元数据被格式化过，你可以通过Java注解或是代码容器下指令来明确声明支持一些额外的自定义配置通过少量XML配置。

在大多数应用场景，明确用户代码不需要安装Spring IoC容器 一个或多个实例。例如：在一个web应用场景下，一个简单的8行左右的模板在Web.xml中描述，就足以满足大多数需求。如果你使用Spring Tool Suite(an Eclipse-powered development environment)，你可以简单的创建这个模板配置通过点点鼠标旧可以完成。

如下图表，展示了Spring在高层是如何运作的。你的应用类结合配置元数据，在`ApplicationContext`被创建，初始化后，你就有了一个完全配置可执行系统（应用）。

![运作方式](https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/images/container-magic.png)

#### 配置元数据

如之前图表所展示的，Spring IoC容器消费配置元数据。配置元数据表示你,作为一个应用开发者,告诉Spring 容器来如何在你的应用中安装配置组装对象.

配置元数据通常被简单和直接以XML格式,这将是本章节最多传输的概念和Spring控制反转容器的特性.

基于XML元数据不仅仅是配置元数据的组成.Spring控制反转容器他本身完全脱离了配置元数据的所写配置设计.现今,许多开发者选择使用基于java的配置.

查看更多信息关于Spring容器所组成的元数据.查看:

· 基于注解的配置:Spring2.5开始支持基于注解的配置元数据.
· 基于Java配置: 从Spring 3.0开始,许多基于java配置的项目成为Spring框架的核心.因而,你可以自行定义实例通过java类而非XML文件.为使用这些新特性,请查看 `@Configuration`,`@Bean`, `@Import`, `@DependsOn` 这些注解.

Spring配置认为至少一个或常常不仅仅一个组件定义都被Spring容器所管理.基于XML文件的配置通过使用`<beans/>`(处于顶层元素)的子元素:`<bean/>`来配置.java配置通常使用在存有`@Configuration`的注解的类的`@Bean`的注解方法来进行配置.

这些组件定义与你所需装配的应用的对象对应.通常来说,你定义的业务层对象,data access objects (DAOs),显示层如 Structs `Action`实例,基础对象例如 Hibernate的事务工厂,JMS队列 等等.通常来说,在容器中,并不会配置有细粒度的域对象,因为这些责任均属于DAOs层或业务逻辑来创建域对象.然而,你可以通过使用Spring整合的AspectJ来配置对象(那些在IoC容器之外创建的对象) 查看 Using [AspectJ to dependency-inject domain objects with Spring](https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/core.html#aop-atconfigurable)


如下例子展示了XML配置元数据的基本架构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">   
        <!-- id 标签是字符串用作于区分每个实例定义 -->
        <!-- class 标签定义了此组件的类型 使用全名 -->
        <!-- 此组件需要的依赖或是配置在这填写 -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- 更多组件定义在此 -->

</beans>
```
id 标签的值指向协作对象,XML中引用协作对象在此例子中并不会展示,在`Dependecies`中查看更多信息.

#### 实例化容器

位置路径或是多个位置路径作为`ApplicationContext`构造函数让容器加载配置元数据从各种个样的额外的资源,例如本地文件系统,或是Java CLASSPATH 等等.
```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

> 在你学习了关于Spring 控制反转容器后,你可能想知道更多关于Spring的资源抽象(Resources 如中所述),他将会提供一个方便的机制从以URI语法位置定义来读流.特别得说,资源路径用作于构造应用上下文的参数,如描述所说Application Contexts and Resource Paths.

如下例子展示了业务层对象配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 业务们 -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- 额外的协作对象和配置在此增加 -->
    </bean>

    <!-- more bean definitions for services go here -->
    <!-- 更多的组件定义在这里写 -->
</beans>
```

如下例子展示了数据接入对象 daos.xml 文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- 额外的协作对象和配置在此增加 -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
       <!-- 额外的协作对象和配置在此增加 -->
    </bean>

    <!-- 更多的组件定义在这里写 -->

</beans>
```

在之前的例子中,业务层由`PetStoreServiceImpl`类和两个DAO类型分别为`JpaAccountDao` 和 `JpaItemDao` (基于JPA Object-Relational Mapping 标准). 属性名称元素引用了java组件的属性名称,`ref`元素指向另一个组件定义的名称.这个id 和 ref 之间元素的关系表现了协作对象的依赖关系.想查看更多相关配置对象依赖的信息,请查看Dependencies.

编排基于XML的配置元数据
多个XML文件拥有组件定义十分有用.通常来说,每个独立的XML配置文件在你的架构中表示一个逻辑层面或模块.

你可以使用Application context构造器加载从XML片段中加载组件定义.此构造器有多种资源路径,就如同上一节所展示的.另外,使用一个或多个`<import/>`的元素来从其他文件中加载组件定义.如下例子告诉你如何去做:

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

之前的例子中,外部的组件定义从三个文件中被加载 `services.xml`, `messageSource.xml` 和 `themeSource.xml`.所有文件都是以当前正在导入的文件所相关的,所以 `services.xml`必须与当前导入文件的处于相同的文件夹中或是在classPath中,而messageSource.xml 和 themeSource.xml 必须在导入文件的资源文件位置下面.就如你所能看到的,将会忽略最前面的"/".然而给的这些路径都是相对路径,最好不要使用“/”.文件所要导入的内容,包括最高层的`<bean/>`元素,必须是有效的XML实例定义,依据Spring模式.

>   虽然可以但是不建议,引用文件通过使用相对路径“../”,这样做将会使你的应用依赖于一个外部文件.尤其不建议你使用Classpath: URLS (例如:classpath:../services.xml),当运行时解析器选择一个“最为相近的”类路径根而后找到了父类的文件夹中.类配置的改变可能会导致你选择了错的文件夹.

你可以总使用全路径来做而非使用相对路径,例如(file:C:/config/services.xml 或是 classpath:/config/services.xml),要注意你的应用和一些绝对路径下的文件产生了耦合,通常使用"${…​}" 占位符来间接性的使用绝对路径,通过jvm解析系统变量.

命名空间本身提供导入指令特征。更多超出普通实例定义的配置特性，使用Spring所提供的XML命名空间，例如，`context`和`util`命名空间。

##### Groovy实例定义 DSL
作为一个外部配置元数据的进一步例子，实例定义也可以通过` Spring’s Groovy Bean Definition DSL`来表达。众所周知的Grails 框架，通常配置在'.groovy'文件结构如下：
```java
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

这个配置风格很大程度与XML实例定义相当，甚至都支持Spring XML配置命名空间。此方式也允许通过直接使用`importBeans`来导入XML实例定义。


#### 1.2.3 Using the Container 使用容器