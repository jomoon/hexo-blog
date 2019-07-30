---
title: Guice Translation Bindings
---
translation from [google/guice](https://github.com/google/guice/wiki/LinkedBindings)

##多种绑定方式

### LinkedBindings 链式绑定

链式绑定映射了类型和其实现类。如下例子映射了接口`TransactionLog`和他的实现类`DatabaseTransactionLog`

```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);
  }
}
```

现在，当你调用injector.getInstance(TransactionLog.class)时，或当`injector`遇到`TransactionLog`的依赖时，他将会使用`DatabaseTransactionLog`。联接了一个类型和他任一子类，例如实现了类或者是继承一个类。你也可以联接具体的类`DatabaseTransactionLog`和他的子类：

```java
 bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
```
联接绑定亦可链式的：

```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);
    bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
  }
}
```

在这种情况下，当一个TransactionLog被需要时，注入器会返回一个`MySqlDatabaseTransactionLog`


### 注解绑定
有时候，你会为同一类型想要多绑定。例如你可能同时要一个PayPal信用卡处理器和一个Google 账单处理器。为了使用这种，绑定支持可选择的绑定注解。注解和他的类一起来确定一个绑定关系。这种关系叫做key。

定义一个绑定注解，需要俩行代码和一些导入。把这些放到他自己的java文件或他所要用的类文件中。

```java
package example.pizza;

import com.google.inject.BindingAnnotation;
import java.lang.annotation.Target;
import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;

@BindingAnnotation @Target({ FIELD, PARAMETER, METHOD }) @Retention(RUNTIME)
public @interface PayPal {}
```

你都不需要去理解所有的元注解，但如果你好奇的话：
`@BindingAnnotation` 告诉`Guice`这个注解是绑定注解。如果你对同一个对象使用多重绑定`Guice`会出错。
`@Target({FIELD, PARAMETER, METHOD})`是对用户好意的。他会防止你无意在在其他地方应用了`@PayPal`