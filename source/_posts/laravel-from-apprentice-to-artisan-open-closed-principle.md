title: 从百草堂到三味书屋 —— 开放封闭原则
date: 2016-07-10 20:36:21
tags: [php, laravel]
---

# Open Closed Principle 开放封闭原则

## Introduction 介绍

在一个应用的生命周期里，大部分时间都花在了向现有代码库增加功能，而非一直从零开始写新功能。正像你所想的那样，这会是一个繁琐且令人痛苦的过程。当你修改代码的时候，你可能引入新的程序错误，或者将原来管用的功能搞坏掉。理想情况下，我们应该可以像写全新的代码一样，来快速且简单的修改现有的代码。只要采用开放封闭原则来正确的设计我们的应用程序，那么这是可以做到的！

> **Open Closed Principle 开放封闭原则**
>
> 开放封闭原则规定代码对扩展是开放的，对修改是封闭的。

## In Action 实践

为了演示开放封闭原则，我们来继续编写上一章节的 `OrderProcecssor`。考虑下面的 `process` 方法：

```php
$recent = $this->orders->getRecentOrderCount($order->account);

if ($recent > 0) {
  throw new Exception('Duplicate order likely.');
}
```

这段代码可读性很高，且因为我们使用了依赖注入，变得很容易测试。然而，如果我们判断订单的规则改变了呢？如果我们又有新的规则了呢？更进一步，如果随着我们的业务发展，要增加一大堆新规则？那我们的 `process` 方法会很快变成一坨难以维护的浆糊。因为这段代码必须随着每次业务逻辑的改变而跟着改变，它对修改是开放的，这违反了开放封闭原则。记住，我们希望代码对扩展开放，而不是修改。

不必再把订单验证直接写在 `process` 方法里面，我们来定义一个新的接口 `OrderValidator`:

```php
interface OrderValidatorInterface {
  public function validate(Order $order);
}
```

下一步我们来定义一个实现接口的类，来预防重复订单：

```php
class RecentOrderValidator implements OrderValidatorInterface {
  public function __construct(OrderRepository $orders)
  {
    $this->orders = $orders;
  }

  public function validate(order $order)
  {
    $recent = $this->orders->getRecentOrderCount($order->account);
    if ($recent > 0) {
      throw new Exception('Duplicate order likely.');
    }
  }
}
```

很好！我们封装了一个小巧的、可测试的单一业务逻辑。咱们来再创建一个来验证账号是否停用吧：

```php
class SuspendedAccountValidator implements OrderValidatorInterface {
  public function validate(Order $order)
  {
    if ($order->account->isSuspended()) {
      throw new Exception('Suspended accounts may not order.');
    }
  }
}
```

现在我们有两个不同的类实现了 `OrderValidatorInterface` 接口。咱们将在 `OrderProcessor` 里面使用它们。我们只需简单的将一个验证器数组注入进订单处理器实例中。这将使我们以后修改代码时能轻松的添加和删除验证器规则。

```php
class OrderProcessor {
  public function __construct(BillerInterface $biller, OrderRepository $orders, array $validators = array())
  {
    $this->biller = $biller;
    $this->orders = $orders;
    $this->validators = $validators;
  }
}
```

然后我们只要在 `process` 方法里面循环这个验证器数组即可：

```php
public function process(Order $order)
{
  foreach ($this->validators as $validator) {
    $validator->validate($order);
  }

  // Process valid order...
}
```

最后我们在 IoC 容器里面注册 `OrderProcessor` 类：

```php
App::bind('OrderProcessor', function () {
  return new OrderProcessor(
    App::make('BillerInterface'),
    App::make('OrderRepository'),
    array(
      App::make('RecentOrderValidator'),
      App::make('SuspendedAccountValidator')
    )
  );
});
```

在现有代码里付出些小努力，做一些小改动之后，我们现在可以添加删除新的验证规则而不必修改任何一行现有代码了。每一个新的验证规则就是对 `OrderValidatorInterface` 的一个实现类，然后注册进 IoC 容器里。不必再为那个又大又笨的 `process` 方法做单元测试了，我们现在可以单独测试每一个验证规则。现在，我们的代码对扩展是开放的，对修改是封闭的。

> **Leaky Abstractions 抽象的漏洞**
>
> 小心那些缺少实现细节的依赖（译者注：比如上面的 RecentOrderValidator）。当一个依赖的实现需要改变时，不应该要求它的调用者做任何修改。当需要调用者进行修改时，这就意味着该依赖遗漏了一些实现的细节。当你的抽象有漏洞的话，开放封闭原则就不管用了。

在我们继续学习前，要记住这些原则不是法律。这不是说你应用中每一块代码都应该是“热插拔”式的。例如，一个仅仅从 MySQL 检索几条记录的小应用程序，不值得去严格遵守每一条你想到的设计原则。不要盲目的应用设计原则，那样你会造出一个“过度设计”的繁琐的系统。记住这些设计原则是用来解决通用的架构问题，制造大型容错能力强的应用。我就这么一说，你可别把它当做懒惰的借口！

译文地址：[http://my.oschina.net/zgldh/blog/389246](http://my.oschina.net/zgldh/blog/389246)