title: 从百草堂到三味书屋 —— 控制反转容器
date: 2016-07-08 17:28:17
tags: [php, laravel]
---

# The IoC Container 控制反转容器

## Basic Binding 基础绑定

我们已经学习了依赖注入，接下来咱们一起来探索“控制反转容器”（IoC）。IoC 容器可以使你更容易管理依赖注入，Laravel 框架拥有一个很强大的 IoC 容器。Laravel 的核心就是这个 IoC 容器，这个 IoC 容器使得框架各个组件能很好的在一起工作。事实上 Laravel 的 Application 类就是继承自 Container 类！

> **IoC Container 控制反转容器**
> 
> 控制反转容器使得依赖注入更方便。当一个类或接口在容器里定义以后，如何处理它们——如何在应用中管理、注入这些对象？

在 Laravel 应用里，你可以通过 App 来访问控制反转容器。容器有很多方法，不过我们从最基础的开始。让我们继续使用上一章写的 `BillerInterface` 和 `BillingNotifierInterface`，且假设我们使用了 Stripe 来进行支付操作。我们可以将 Stripe 的支付实现绑定到容器里，就像这样：

```php
App::bind('BillerInterface', function () {
  return new StripeBiller(App::make('BillingNotifierInterface'));
});
```

注意在我们处理 `BillingInterface` 时，我们额外需要一个 `BillingNotifierInterface` 的实现，也就是再来一个 bind:

```php
App::bind('BillingNotifierInterface', function () {
  return new EmailBillingNotifier; 
});
```

如你所见，这个容器就是个用来存储各种绑定的地方（译者注：这么理解简单点。再扯匿名函数、闭包就扯远了。）。一旦一个类在容器里绑定了以后，我们可以很容易的在应用的任何位置调用它。我们甚至可以在 bind 函数内写另外的 bind。

> **Have Acne?**
> 
> Laravel 框架的 Illuminate 容器和另一个名为 [Pimple](https://github.com/fabpot/pimple) 的 IoC 容器是可替换的。所以如果你之前用的是 Pimple，你尽可以大胆的升级为 [Illuminate Container](https://github.com/jilluminate/container)，后者还有更多新功能!

一旦我们使用了容器，切换接口的实现就是一行代码的事儿。比如考虑以下代码：

```php
class UserController extends BaseController {
  public function __construct(BillerInterface $biller)
  {
    $this->biller = $biller;
  }
}
```

当这个控制器通过被容器实例化后，包含着 `EmailBillingNotifier` 的 `StripeBiller` 会被注入到这个控制器中（译者注：见上文的两个 bind）。如果我们现在想要换一种提示方式，我们可以简单的将代码改为这样：

```php
App::bind('BillingNotifierInterface', function (){
  return new SmsBillingNotifier; 
});
```

现在不管在应用的哪里需要一个提示器，我们总会得到 `SmsBillingNotifier` 的对象。利用这种结构，我们的应用可以在不同的实现方式之间快速切换。

只改一行就能切换代码实现，这可是很厉害的能力。比如我们想把短信服务从原来的提供商替换为 Twilio。我们可以开发一个新的 Twilio 的提示器类（译者注：当然要继承自 `BillingNotifierInterface`）然后修改绑定语句。如果 Twilio 有任何闪失，我们只需修改一行代码就可以快速的切换回原来的短信提供商。看到了吧，依赖注入的好处多得很呢。你能再想出几个使用依赖注入和控制反转容器的好处么？

想在应用中只实例化某类一次？没问题，使用 `singleton` 方法吧：

```php
App::singleton('BillingNotifierInterface', function () {
  return new SmsBillingNotifier; 
});
```

这样只要这个容器生成了这个提示器对象一次，在接下来的生成请求中容器都只会提供这同样的一个对象。

容器的 `instance` 方法和 `singleton` 方法很类似，区别是 `instance` 可以绑定一个已经存在的对象。然后容器每次返回的都是这个对象了。

```php
$notifier = new SmsBillingNotifier;
App::instance('BillingNotifierInterface', $notifier);
```

现在我们熟悉了容器的基础用法，让我们深入发掘它更强大的功能：依靠反射来处理类和接口。

> **Stand Alone Container 容器独立运行**
> 
> 你的项目中没有使用 Laravel？但你依然可以使用 Laravel 的 IoC 容器！只要用 Composer 安装 `illuminate/container` 包就可以了。

## Reflect Resolution 反射解决方案

用反射来自动处理依赖是 Laravel 容器的一个最强大的特性。反射是一种运行时探测类和方法的能力。比如，PHP 的 `ReflectionClass` 可以探测一个类的方法。`method_exists` 某种意义上说也是一种反射。我们来把玩一下 PHP 的反射类，试试下面的代码吧（StripeBiller 换成你自己定义好的类）：

```php
$reflection = new ReflectionClass('StripeBiller');
var_dump($reflection->getMethods());
var_dump($reflection->getConstans());
```

依靠这个强大的 PHP 特性，Laravel 的 IoC 容器可以实现很有趣的功能！考虑接下来这个类：

```php
class UserController extends BaseController
{
  public function __construct(StripBiller $biller)
  {
    $this->biller = $biller;
  }
}
```

注意这个控制器的构造函数暗示着有一个 `StripBiller` 类型的参数。使用反射就可以检测到这种类型暗示。当 Laravel 的容器无法解决一个类型的明显绑定时，容器会试着使用反射来解决。程序流程类似于这样的：
- 已经有一个 `StripBiller` 的绑定了么？
- 没绑定？那用反射来探测一下 `StripBiller` 吧。看看他都需要什么依赖。
- 解决 `StripBiller` 需要的所有依赖（递归处理）
- 使用 `ReflectionClass->newInstanceArgs()` 来实例化 `StripBiller`

如你所见，容器替我们做了好多重活，这能帮你省去写大量绑定的麻烦。这就是 Laravel 容器最强大也最独特的特性。熟练掌握这种能力对构建大型 Laravel 应用是十分有益的。

下面我们修改一下控制器，改成这样会发生什么事儿呢？

```php
class UserController extends BaseController
{
  public function __construct(BillerInterface $biller)
  {
    $this->biller = $biller;
  }
}
```

假设我们没有为 `BillerInterface` 做任何绑定，容器该怎么知道要注入什么类呢？要知道，interface 不能被实例化，因为它只是个约定。如果我们不提供更多信息的话，容器是无法实例化这个依赖的。我们需要明确指出哪个类需要实现这个接口，这就需要用到 `bind` 方法：

```php
App::bind('BillerInterface', 'StripBiller');
```

这里我们只传了一个字符串进去，而不是一个匿名函数。这个字符串告诉容器总是使用 `StripBiller` 来作为 `BillerInterface` 的实现类。此外我们也获得了只改一行代码即可轻松改变实现的能力。比如，假设我们需要切换到 Balanced Payments 作为我们的支付提供商，我们只需要新写一个 `BalancedBiller` 来实现 `BillerInterface` 接口，然后这样修改容器代码：

```php
App::bind('BillerInterface', 'BalancedBiller');
```

我们的应用程序就装上了新的支付实现代码了！

你也可以使用 `singleton` 方法来实现单例模式。

```php
App::singleton('BillerInterface', 'StripBiller');
```

> Master The Container 掌握容器
> 
> 想了解更多关于容器的知识？去读源码！容器只有一个类 `Illuminate\Container\Container`。读完了你就对容器有更深入的认识了。

译文地址：[http://my.oschina.net/zgldh/blog/389246](http://my.oschina.net/zgldh/blog/389246)