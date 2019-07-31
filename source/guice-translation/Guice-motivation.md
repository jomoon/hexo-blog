---
title: Guice Translation
---
translation from [google/guice](https://github.com/google/guice/wiki/Motivation)

## 动机

### 动机
在应用开发的时候，将所有的实例写在一起是最无聊的一部分。其实有多种方式将数据，服务，类和其他接在一起。为了对比这些方式，我们写了披萨订单网站的账单代码。
```java
   public interface BillingService {

  /**
   * Attempts to charge the order to the credit card. Both successful and
   * failed transactions will be recorded.
   *
   * @return a receipt of the transaction. If the charge was successful, the
   *      receipt will be successful. Otherwise, the receipt will contain a
   *      decline note describing why the charge failed.
   */
  Receipt chargeOrder(PizzaOrder order, CreditCard creditCard);
  } 
```
随着接口的实现，我们要写为我们的代码写测试类，在这些测试中，我们需要`FakeCreditCardProcessor`(假的信用卡过程)来避免消费了真实的信用卡！

### 直接构造器调用
这里就是我们直接`new`信用卡过程实例和交易记录的代码，如下：

```java
public class RealBillingService implements BillingService {
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = new PaypalCreditCardProcessor();
    TransactionLog transactionLog = new DatabaseTransactionLog();

    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

这个代码暴露了模块化和可测试能力的问题。直接来说，编译依赖真实信用卡流程意味着测试代码会扣真实的信用卡！若收费被拒绝或是服务不可用，测试也显得十分笨拙。

### 构造器模式调用
工厂类解藕了客户端和实现类，简单工厂使用静态方法来取和设置接口的模拟实现。一个工厂实现了一些模板代码：

```java
public class CreditCardProcessorFactory {
  
  private static CreditCardProcessor instance;
  
  public static void setInstance(CreditCardProcessor processor) {
    instance = processor;
  }

  public static CreditCardProcessor getInstance() {
    if (instance == null) {
      return new SquareCreditCardProcessor();
    }
    
    return instance;
  }
}
```

在客户端代码种，我们仅需要替换`new`的工厂调用：

```java
public class RealBillingService implements BillingService {
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = CreditCardProcessorFactory.getInstance();
    TransactionLog transactionLog = TransactionLogFactory.getInstance();

    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

工厂方法使得做单元测试时成为可能。

```java
public class RealBillingServiceTest extends TestCase {

  private final PizzaOrder order = new PizzaOrder(100);
  private final CreditCard creditCard = new CreditCard("1234", 11, 2010);

  private final InMemoryTransactionLog transactionLog = new InMemoryTransactionLog();
  private final FakeCreditCardProcessor processor = new FakeCreditCardProcessor();

  @Override public void setUp() {
    TransactionLogFactory.setInstance(transactionLog);
    CreditCardProcessorFactory.setInstance(processor);
  }

  @Override public void tearDown() {
    TransactionLogFactory.setInstance(null);
    CreditCardProcessorFactory.setInstance(null);
  }

  public void testSuccessfulCharge() {
    RealBillingService billingService = new RealBillingService();
    Receipt receipt = billingService.chargeOrder(order, creditCard);

    assertTrue(receipt.hasSuccessfulCharge());
    assertEquals(100, receipt.getAmountOfCharge());
    assertEquals(creditCard, processor.getCardOfOnlyCharge());
    assertEquals(100, processor.getAmountOfOnlyCharge());
    assertTrue(transactionLog.wasSuccessLogged());
  }
}
```

这段代码也很笨拙。一个全局变量包含有模拟实现，因此我们需要对于其创建和销毁要十分小心。销毁失败的话，全局变量继续指向我们的测试实例。这样可能会导致其他问题。这也阻止了我们同时进行多重测试。

但最大的问题是依赖是隐藏在代码中的。如果我们在信用卡欺诈追踪器`CreditCardFraudTracker`上增加了依赖，我们必须重新跑测试来找到哪个导致其失败。
我们在正式业务种应该忘了初始化工厂吗，我们直到收费开始才能找到问题。随着应用生长，婴儿喂养式的工厂成为一种持续上升的拖累。

质量问题足以被QA或是验收测试发现，但我们明显可以做得更好。

### 依赖注入

就像工厂，依赖注入就是一个设计模式。他的核心原则就是从依赖解析中分离行为。在我们的例子中，真实的收费业务`RealBillingService`并无责任来找交易记录`TransactionLog`和信用卡处理器`CreditCardProcessor`。然而，他们是作为构造参数传入的。

```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  public RealBillingService(CreditCardProcessor processor, 
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

我们不需要任何工厂，我们可以通过移除构造和卸载模板来简化测试用例。

```java
public class RealBillingServiceTest extends TestCase {

  private final PizzaOrder order = new PizzaOrder(100);
  private final CreditCard creditCard = new CreditCard("1234", 11, 2010);

  private final InMemoryTransactionLog transactionLog = new InMemoryTransactionLog();
  private final FakeCreditCardProcessor processor = new FakeCreditCardProcessor();

  public void testSuccessfulCharge() {
    RealBillingService billingService
        = new RealBillingService(processor, transactionLog);
    Receipt receipt = billingService.chargeOrder(order, creditCard);

    assertTrue(receipt.hasSuccessfulCharge());
    assertEquals(100, receipt.getAmountOfCharge());
    assertEquals(creditCard, processor.getCardOfOnlyCharge());
    assertEquals(100, processor.getAmountOfOnlyCharge());
    assertTrue(transactionLog.wasSuccessLogged());
  }
}
```

现在，无论何时我们要增加或者移除依赖，编译器都会提示我们哪些需要被修复。这些依赖都会背暴露在API签名上。

不幸的是，现在BillingService的客户端需要去查看他的依赖，我们再通过运用设计模式可以修复一些。BillingService类中依赖可以通过构造函数去接受。对于高层的类，有一个框架是非常有用的，否则的化，你就需要递归构造依赖来创建你想要的业务类：

```java
 public static void main(String[] args) {
    CreditCardProcessor processor = new PaypalCreditCardProcessor();
    TransactionLog transactionLog = new DatabaseTransactionLog();
    BillingService billingService
        = new RealBillingService(processor, transactionLog);
    ...
  }
```

### 通过Guice来依赖注入
依赖注入模式将代码引向模块化和可测试化，并且Guice使得依赖注入十分容易去写。在我们收费例子中来使用Guice，我们先的高素Guice如何**映射**我们的接口到其实现类。那些实现了`Module`接口的java类，配置应该在Guice模型中。

```java
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);
    bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
    bind(BillingService.class).to(RealBillingService.class);
  }
}
```

我们通过给`RealBillingService`的构造器增加`@Inject`注解，这个注解引导Guice来使用这个构造器。Guice将会探测到构造器的注解，和构造器所要的值。

```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processor;
  private final TransactionLog transactionLog;

  @Inject
  public RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog) {
    this.processor = processor;
    this.transactionLog = transactionLog;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    try {
      ChargeResult result = processor.charge(creditCard, order.getAmount());
      transactionLog.logChargeResult(result);

      return result.wasSuccessful()
          ? Receipt.forSuccessfulCharge(order.getAmount())
          : Receipt.forDeclinedCharge(result.getDeclineMessage());
     } catch (UnreachableException e) {
      transactionLog.logConnectException(e);
      return Receipt.forSystemFailure(e.getMessage());
    }
  }
}
```

最终，我们可以将它们放在一起。`Injector`可以用作于获取任何相关实例

```java
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(new BillingModule());
    BillingService billingService = injector.getInstance(BillingService.class);
    ...
  }
```

[开始](https://github.com/google/guice/wiki/GettingStarted)解释其如何工作。