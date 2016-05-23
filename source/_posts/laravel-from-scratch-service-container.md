title: laravel 基础教程 —— 服务容器
date: 2016-05-17 21:11:56
tags: [php, laravel]
---

# 简介

laravel 提供了强大的服务容器工具来管理类之间的依赖关系并以此来提供依赖注入。依赖注入是一个奇特的短语，其实它的意思是说所依赖的类的实例通过构造函数或者其它的方式被注入到类中。如果你还是不太明白，你可以参考我写的这篇 [《理解依赖注入与控制反转》](http://team.dearmadman.com/2016/04/13/dependency-injection-and-inversion-of-control/)。

让我们来看一个简单的例子：

```php
<?php

namespace App\Jobs;

use App\User;
use Illuminate\Contracts\Mail\Mailer;
use Illuminate\Contracts\Bus\SelfHandling;

class PurchasePodcast implements SelfHandling
{
    /**
     * The mailer implementation.
     */
     protected $mailer;

     /**
      * Create a new instance.
      *
      * @param Mailer $mailer
      * @return void
      */
      public function __construct(Mailer $mailer)
      {
        $this->mailer = $mailer;
      }

      /**
       * Purchase a podcast.
       *
       * @return void
       */
       public function handle()
       {
          //
       }
}
```

上面的例子中 `PurchasePodcast` 任务需要在播客服务被购买时发送一个 email，所以我们注入了一个可以发送邮件的服务。因为服务是注入进去的而不是类自身硬编码进去的，所以我们可以很容易的切换到其他发送邮件的实现，我们也可以很容易的去 `mock`，这样我们在做单元测试的时候就很容易去做一些假的数据。

能够更深层次的理解 laravel 的服务容器是非常必要的，它是能够构造强大的应用的关键，服务容器构成了整个 laravel 应用的核心。

## 绑定

几乎所有的服务容器的绑定都是在服务提供者中被注册的，所有下面的例子都是在这个环境下进行演示的。如果类并没有依赖什么接口，它是没有必要在容器中进行绑定的。容器并不需要有什么具体的指示去如何构造这些实例，因为他们会根据 PHP 的反射进行自动的实例化。

在服务提供者内部，你可以通过 `$this->app` 来访问容器的实例。我们可以使用 `bind` 方法来注册绑定，这需要传递类或接口名，然后跟上一个 `Closure` 闭包函数，闭包用来返回一个所绑定类的实例:

```php
$this->app->bind('HelpSpot\API', function ($app) {
  return new HelpSpot\API($app['HttpClient']);
});
```
注意上面的例子，我们将容器的实例作为参数传递到闭包中，这样我们就可以使用容器来解析我们所构建对象的子依赖。

### 单例式绑定

`singleton` 方法绑定的类或者接口只会被解析一次，也就是说随后的访问都会返回这个相同的实例：

```php
$this->app->singleton('FooBar', function ($app) {
  return new FooBar($app['SomethingElse']);
});
```

### 直接绑定实例

你也可以使用 `instance` 方法来绑定一个已经存在的实例对象到容器中。随后的访问中，容器都会返回这个给定的实例：

```php
  $fooBar = new FooBar(new SomethingElse);

  $this->app->instance('FooBar', $fooBar);
```

### 绑定接口到实现

服务容器具有一个非常强大的特性就是能够绑定接口到给定的实现。比如，让我们假设我们有 `EventPusher` 接口和 `RedisEventPusher` 实现。一旦我们绑定 `RedisEventPusher` 实现到这个接口，我们可以在服务容器中这么来进行注册：

```php
$this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');
```

这就告诉了容器如果类需要一个 `EventPusher` 的实现时，那就注入一个 `RedisEventPusher` 的实例。现在我们可以在构造函数或者其他地方来写入 `EventPusher` 接口的类型提示来进行依赖注入：

```php
use App\Contracts\EventPusher;

/**
 * Create a new class instance.
 *
 * @param EventPusher $pusher
 * @return void
 */
 public function __construct(EventPusher $pusher)
 {
    $this->pusher = $pusher;
 }
```

### 上下文绑定

有时候你可能会有实现了同一个接口的两个类，而且你想要在不同的场景下注入不同的实现，比如说，当我们的系统接收到了一个订单，我们可能想要通过 [PubNub](http://www.pubnub.com/) 来发送一个事件，而不是 Pusher。laravel 提供了一种简单流利的接口来定义这种行为：

```php
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('App\Contracts\EventPusher')
          ->give('App\Services\PubNubEventPusher');
```

你也可以在 `give` 方法中传递一个 Closure :

```php
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('App\Contracts\EventPusher')
          ->give(function () {
            // Resolve dependency...
          });
```

### 绑定原始类型

有时候你可能需要在类中注入许多的依赖类，但是你可能也需要注入一些 PHP 的原始类型数据，比如说 integer, boolean。你可以在上下文绑定中非常轻松的绑定类所需要的：

```php
$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('$maxOrderCount')
          ->give(10);
```

## 标记

有时候你可能需要解析一些符合某种类别的所有绑定。比如说，你可能需要来建造一个报告聚合器来接收一个包含了不同 `Report` 接口的实现集。在实现了 `Report` 的注册之后，你可以使用 `tag` 方法来给它们打上标记：

```php
$this->app->bind('SpeedReport', function () {
  //  
});

$this->app->bind('MemoryReport', function () {
  // 
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```

一旦这些服务被打上标记，你可以使用 `tagged` 方法来解析获得它们:

```php
$this->app->bind('ReportAggregator', function ($app) {
  return new ReportAggregator($app->tagged('reports'));
});
```

## 解析

laravel 提供了多种方式从服务容器中解析出实例。首先，你可以使用 `make` 方法，它接收一个你想要解析的类或者接口：

```php
$fooBar = $this->app->make('FooBar');
```

其次，你也可以使用数组的方式从容器中解析对象，因为它实现了 PHP 的 `ArrayAccess` 接口:

```php
$fooBar = $this->app['FooBar'];
```

最后，也是最重要的，你也可以在类的构造函数中简单的使用‘类型提示’来从容器中解析依赖。控制器，事件监听器，队列任务，中间件等都支持这种方式。在实践中，这是大部分对象从容器中解析的方式。

容器会自动的为类进行依赖的注入。比如，你可以在控制器的构造函数中对应用中定义的仓库类进行类型提示。仓库实例会自动的从容器中进行解析并注入到控制器中:

```php
<?php

namespace App\Http\Controllers;

use App\Users\Repository as UserRepository;

class UserController extends Controller
{
  /**
   * The user repository instance.
   *
   */
   protected $users;

   /**
    * Create a new controller instance.
    *
    * @param UserRepository $users
    * @return void
    */
    public function __construct(UserRepository $users)
    {
      $this->users = $users;
    }

    /**
     * Show the user with the given ID.
     *
     * @param int $id
     * @return Response
     */
     public function show($id)
     {
       //
     }
}
```

## 容器事件

容器在每次解析一个对象时都会触发一个事件，你可以通过 `resolving` 方法来监听这个事件:

```php
$this->app->resolving(function ($object, $app) {
  // Called when container resolves object of any type... 
});

$this->app->resolving(FooBar::class, function (FooBar $fooBar, $app) {
  // Called when container resolves objects of type "FooBar"... 
});
```

如你所看到的，`resolving` 方法中你可以传递一个回调函数，回调函数会接收两个参数，一个是解析得到的对象，一个是容器本身，这样你就可以在对象传递到消费者之前添加一些额外的属性。