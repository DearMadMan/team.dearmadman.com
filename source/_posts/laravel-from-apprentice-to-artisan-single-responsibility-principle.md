title: 从百草堂到三味书屋 —— 单一职责
date: 2016-07-10 19:52:21
tags: [php, laravel]
---

# Single Responsibility Principle 单一职责原则

## Introduction 介绍

罗伯特“鲍勃叔叔”马丁阐述了名为“坚实”的一些设计原则（译者注：看下面五个原则的首个字母正是 SOLID）。这些都是制作完善的程序设计的优秀的基础，一共有五个原则：
- The Single Responsibility Principle 单一职责原则
- The Open Closed Principle 开放封闭原则
- The Liskov Substitution Principle 里氏替换原则
- The Interface Segregation Principle 接口隔离原则
- The Dependency Inversion Principle 依赖反转原则

让我们深入探索一下，再看点代码样例来说明各个原则。我们将看到，每个原则都对其他原则有增益作用。如果其中一个原则没有被遵循，那么其他大部分（可能不会是全部）的原则依然可以工作的很好。

## In Action 实践

单一职责原则规定一个类有且仅有一个理由使其改变。换句话说，一个类的功能边界和职责应当是十分狭窄且集中的。我们之前就提到过，在类的职责问题上，无知是福。一个类应当做它该做的事儿，并且不应当被它的依赖的任何变化所影响到。

考虑下列类：

```php
class OrderProcessor {
  public function __construct(BillerInterface $biller)
  {
    $this->biller = $biller;
  }
  public function process(Order $order)
  {
    $recent = $this->getRecentOrderCount($order);
    if ($recent > 0)
    {
      throw new Exception('Duplicate order likely.');
    }

    $this->biller->bill($order->account->id, $order->amount);

    DB::table('orders')->insert(array(
      'account' => $order->account->id,
      'amount' => $order->amount,
      'created_at' => Carbon::now()
    ));
  }

  protected function getRecentOrderCount(Order $order)
  {
    $timestamp = Carbon::now()->subMinutes(5);
    return DB::table('orders')->where('account', $order->account->id)
             ->where('created_at', '>=', $timestamps)
             ->count();
  }
}
```

上面这个类的职责是什么？很显然顾名思义，它是用来处理订单的。不过由于 `getRecentOrderCount` 这个方法的存在，这个类就有了在数据库中审查某账号订单历史来看有没有重复订单的职责。这个额外的验证职责意味着当我们的存储方式改变或当订单验证规则改变时，我们的这个订单处理器也要跟着改变。

我们必须将这个职责抽离出来放到另外的类里面，比如放到 `OrderRepository`:

```php
class OrderRepository {
  public function getRecentOrderCount(Account $account)
  {
    $timestamp = Carbon::now()->subMinutes(5);
    return DB::table('orders')->where('account', $account->id)
             ->where('created_at', '>=', $timestamp)
             ->count();
  }

  public function logOrder(Order $order)
  {
    DB::table('orders')->insert(array(
      'account' => $order->account->id,
      'amount' => $order->amount,
      'created_at' => Carbon::now()
    ));
  }
}
```

然后我们可以将我们的资料库（译者注：OrderRepository）注入到 `orderProcessor` 里，帮后者承担起对账户订单历史的处理责任：

```php
class OrderProcessor {
  public function __construct(BillerInterface $biller, OrderRepository $orders)
  {
    $this->biller = $biller;
    $this->orders = $orders;
  }

  public function process(Order $order)
  {
    $recent = $this->orders->getRecentOrderCount($order->account);

    if ($recent > 0) {
      throw new Exception('Duplicate order likely.');
    }

    $this->biller->bill($order->account->id, $order->amount);

    $this->orders->logOrder($order);
  }
}
```

现在我们提取出了收集订单数据的责任，当读取和写入订单的方式改变时，我们不再需要修改 `OrderProcessor` 这个类了。我们的类的职责更加的专注和精确，这提供了一个更干净、更有表现力的代码，同时也是更容易维护的代码。

请记住，单一职责原则的关键不仅仅是让函数变短，而是写出职责更精确更高内聚的类，所以要确保类里面所有的方法都属于该类的职责之下的。在建立一个小巧、清晰且职责明确的类库以后，我们的代码会更加解耦，更容易测试，并且更易于更改。

译文地址：[http://my.oschina.net/zgldh/blog/389246](http://my.oschina.net/zgldh/blog/389246)