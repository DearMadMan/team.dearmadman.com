title: 从百草园到三味书屋 —— 接口约定
date: 2016-07-10 08:59:51
tags: [php, laravel]
---

# Interface As Contract 接口约定

## Strong Typing & Water Fowl 强类型和小鸭子

在之前的章节里，涵盖了依赖注入的基础知识：什么是依赖注入；如何实现依赖注入；依赖注入有什么好处。之前章节里面的例子也模拟了将 interface 注入到 classes 里面的过程。在我们继续学习之前，有必要深入讲解一下接口，而这正是很多 PHP 开发者所不熟悉的。

在我成为 PHP 程序员之前，我是写 .NET 的。你觉得我是 M 吗？在 .NET 里可到处都是接口。事实上很多接口是定义在 .NET 框架核心中了，一个好的理由是：很多 .NET 语言比如 C# 和 VB.NET 都是强类型的。也就是说，你在给一个函数传值，要么传原生类型对象，要么就必须给这个对象一个明确的类型定义。比如考虑以下 C# 方法：

```C#
public int BillUser(User user) {
  this.biller.bill(user.GetId(), this.amount)
}
```

注意在这里，我们不仅要定义传进去的参数是什么类型的，还要定义这个方法返回值是什么类型的。C# 鼓励类型安全。除了指定的 `User` 对象，它不允许我们传递其他类型的对象到 `BillUser` 方法中。

然而 PHP 是一种鸭子类型的语言。所谓鸭子类型的语言，一个对象可用的方法取决于使用方式，而非这个方法从哪儿继承或实现。来看个例子：

```php
public function billUser($user) {
  $this->biller->bill($user->getId(), $this->amount);
}
```

在 PHP 里面，我们不必告诉一个方法需要什么类型的参数。实际上我们传递任何类型的对象都可以，只要这个对象能响应 `getId` 的调用。这里有个关于鸭子类型（下文译作：弱类型）的解释：如果一个东西看起来像个鸭子，叫声也像鸭子叫，那他就是个鸭子。换言之在程序里，一个对象看上去是个 User，方法响应也像个 User，那他就是个 User。

不过 PHP 到底有没有任何强类型功能呢？当然有！PHP 混合了强类型和弱类型的结构。为了说明这点，咱们来重写一下 `billUser` 方法：

```php
public function billUser(User $user)
{
  $this->biller->bill($user->getId(), $amount);
}
```

给方法加上了 `User` 类型提示后，我们可以确信的说所有传入 `billUser` 方法的参数，都是 `User` 类或是继承自 `User` 类的一个实例。

强类型和弱类型各有优劣。在强类型语言中，编译器通常能提供编译时错误检查的功能，这功能可是非常有用的。方法的输入和输出也更加明确。

与此同时，强类型的特性也使得程序僵化。比如 Eloquent ORM 中，类似 `whereEmailOrName` 的动态方法就不可能在 C# 这样的强类型语言里实现。我们不讨论强类型弱类型哪种更好，而是要记住他们分别的优劣之处。在 PHP 里面使用强类型标记不是错误，使用弱类型特性也不是错误。但是不加思索，不管实际情况去使用一种模式，这么固执的使用就是错的。

## A Contract Example 约定的范例

接口就是约定。接口不包含任何代码实现，只是定义了一个对象应该实现的一系列方法。如果一个对象实现了一个接口，那么我们就能确信这个接口所定义的一系列方法都能在这个对象上使用。因为有约定保证了特定方法的实现标准，通过多态也能使类型安全的语言变得更灵活。

> Polywhat？多什么态？
> 
> 多态含义很广，其本质上是说一个实体拥有多种形式。在本书中，我们讲多态是一个接口有着多种实现。比如 `UserRepositoryInterface` 可以有 MySQL 和 Redis 两种实现，每一种实现都是 `UserRepositoryInterface` 的一个实例。

为了说明在强类型语言中接口的灵活性，咱们来写一个酒店客房预订的代码。考虑以下接口：

```php
interface ProviderInterface {
  public function getLowestPrice($location);
  public function book($location);
}
```

当用户订房间时，我们需要将此事记录在系统里。所以在 `User` 类里面写点方法：

```php
class User {
  public function bookLocation(ProviderInterface $provider, $location) 
  {
    $amountCharged = $provider->book($location);
    $this->logBookedLocation($location, $amountCharged);
  }
}
```

因为我们写出了 `ProviderInterface` 的类型提示，该 `User` 类的就可以放心大胆的认为 `book` 方法是可以调用的。这使得 `bookLocation` 方法有了重用性。当用户想要换一家酒店提供商时也就更灵活。最后咱们来写点代码来强化他的灵活性。

```php
$location = 'Hilton, Dallas';

$cheapestProvider = $this->findCheapest($location, array(
  new PricelineProvider,
  new OrbitzProvider,
));

$user->bookLocation($cheapestProvider, $location);
```

太棒了！不管哪家是最便宜的，我们都能够将他传入 `User` 对象来预订房间了。由于 `User` 对象只需要有一个符合 `ProviderInterface` 约定的实例就可以预订房间，所以未来有更多的酒店供应商我们的代码也可以很好的工作。

> Forget The Details 忘掉细节
> 
> 记住，接口实际上不真正做任何事情。它只是简单的定义了类们必须实现的一系列方法。

## Interfaces & Team Development 接口与团队开发

当你的团队在开发大型应用时，不同的部分有着不同的开发速度。比如一个开发人员在制作数据层，另一个开发人员在做前端和网站控制器层。前端开发者想测试他的控制器，不过后端开发较慢没法同步测试。那如果两个开发者能以接口的方式达成协议，后台开发的各种类都遵循这种协议，就像这样：

```php
interface OrderRepositoryInterface {
  public function getMostRecent(User $user);
}
```

一旦建立了约定，就算约定还没实现，前端开发者也可以测试他的控制器了！这样应用中的不同组件就可以按不同的速度开发，并且单元测试也可以做。而且这种处理方法还可以使组件内部的改动不会影响到其他不相关组件。要记着无知是福。我们写的那些类们不用知道别的类如何实现，只要知道它们能实现什么。这下咱们有了定义好的约定，再来写控制器：

```php
class OrderController {
  public function __construct(OrderRepositoryInterface $orders)
  {
    $this->orders = $orders;
  }
  public function getRecent()
  {
    $recent = $this->orders->getMostRecent(Auth::user());
    return View::make('orders.recent', compact('recent'));
  }
}
```

前端开发者甚至可以为这接口写个“假”实现，然后这个应用的视图就可以用假数据填充了：

```php
class DummyOrderRepository implements OrderRepositoryInterface {
  public function getMostRecent(User $user)
  {
    return array('Order 1', 'Order 2', 'Order 3');
  }
}
```

一旦假实现写好了，就可以被绑定到 IoC 容器里，然后整个程序都可以调用他了：

```php
App::bind('OrderRepositoryInterface', 'DummyOrderRepository');
```

接下来一旦后台开发者写完了真正的实现代码，比如叫 `RedisOrderRepository`。那么 IoC 容器就可以轻易的切换到真正的实现上。整个应用就会使用 Redis 读出来的数据。

> Interface As Schematic 接口就是大纲
> 
> 接口在开发程序的“骨架”时非常有用。在设计组件时，使用接口进行设计和讨论都是对你的团队有益处的。比如定义一个 `BillingNotifierInterface` 然后讨论他有什么方法。在写任何实现代码前先用接口讨论好一套好的 API！

译文地址：[http://my.oschina.net/zgldh/blog/389246](http://my.oschina.net/zgldh/blog/389246)