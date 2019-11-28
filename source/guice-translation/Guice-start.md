---
title: Guice Translation 02
date: 2019-09-14 10:29:55
categories:
- guice
- translation
---

translation from [google/guice](https://github.com/google/guice/wiki/GettingStarted) and [google/guice/bindings](https://github.com/google/guice/wiki/Bindings)

## 入门

怎样通过使用`Guice`来依赖注入。

### 入门

用依赖注入方法，在其构造器中接受依赖。为构造对象，你需要建造其依赖，但是为了生成这些依赖，你需要生成这些依赖所需依赖，依次类推。所以当你要建造一个对象，你需要建立对象图。

手动建立对象图属于劳动密集型，容易出错，且使得测试变得异常困难。然而，`Guice`可以替代你建立对象图。但首先，`Guice`需要你去配置你想要的图。

举例来说，我们从`BillingService`类说起，他构造器接受的依赖接口是`CreditCardProcessor` 和 `TransactionLog`。为明确地指明 `BillingService` 被`Guice` 所调用的构造器，我们给其构造器增加`@Inject` 注解。

```java
class BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  BillingService(CreditCardProcessor processor, 
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    ...
  }
}
```

我们想使用`PaypalCreditCardProcessor` 和 `DatabaseTransactionLog` 建造`BillingService`。`Guice` 使用绑定其类型和对应的实现类的Map。一个`Module`是一系列特定使用流畅的，英文式方法调用来组成Map。

```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {

     /*
      * This tells Guice that whenever it sees a dependency on a TransactionLog,
      * it should satisfy the dependency using a DatabaseTransactionLog.
      */
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);

     /*
      * Similarly, this binding tells Guice that when CreditCardProcessor is used in
      * a dependency, that should be satisfied with a PaypalCreditCardProcessor.
      */
    bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
  }
}
```

模块是一个Injector，那是一个Guice的对象图建造者。首先我们创建Injector，然后我们使用他来建造`BillingSerice`

```java
 public static void main(String[] args) {
    /*
     * Guice.createInjector() takes your Modules, and returns a new Injector
     * instance. Most applications will call this method exactly once, in their
     * main() method.
     */
    Injector injector = Guice.createInjector(new BillingModule());

    /*
     * Now that we've got the injector, we can build objects.
     */
    BillingService billingService = injector.getInstance(BillingService.class);
    ...
  }
```

通过建造`billingService`，我们已经使用Guice构造了一个小的对象图。这个图包含billingService 和 他的依赖 信用卡处理器 和 交易记录。


## 绑定

在`Guice`中的bindings总览。

### Bindings

注入器(Injector)的工作是装配对象图。你需要一个给定类型的实例，他搞清楚你想要建造的是什么，解决依赖注入，并且将所有编制在一起。为弄清楚依赖是如何解析的，需要你给注射器做配置。

### 创建Bindings

为了创建Bindings，继承`AbstractModule`并且覆写他的`configure`方法。在方法体中，调用`bind()`来指定每个绑定关系。这些方法都有类型检查如果你写了错误类型，那么编译器就会报错。一旦你创建了你自己的modules(模组)，通过`Guice.createInjector()`传递参数来创建一个injector(注入器)。

使用modules(模组)来创建关联绑定关系，实例绑定，`@Provides`方法，提供者绑定(provider bindings),构造器绑定和非指向性的绑定。

### 更多绑定
除了明确包含在你创建的injector中绑定关系。当依赖被需要但却不存在时企图去创建一个实时绑定。注入器也包含其他绑定关系的提供者。