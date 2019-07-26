---
title: Guice Translation
---
translation from [google/guice]{https://github.com/google/guice/wiki/Motivation}

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