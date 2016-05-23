title: laravel 基础教程 —— 契约
date: 2016-05-18 16:28:06
tags: [php, laravel]
---

# 契约

## 简介

laravel 的契约是对应用框架的核心服务所要求的一种强有力的约束。它本身定义一些接口，要求服务必须要遵守。比如，`Illuminate\Contracts\Queue\Queue` 契约定义了队列任务所必须的方法，而 `Illuminate\Contracts\Mail\Mailer` 契约定义了一些发送邮件所必须的方法。

每种契约在框架中都有相应的提供者去进行实现。比如，laravel 提供了多种驱动的队列任务的实现，还有其中一个的邮件服务的实现是由 [SwiftMailer](http://swiftmailer.org/) 集成的。

所有的 laravel 契约你都可以在这里找到：[GitHub](https://github.com/illuminate/contracts)。这里提供了一个对 laravel 契约参考的快速入口，你可以很好的对这些单一解耦的包进行独立实现的开发。

### 契约 Vs. 假面（Facades）

laravel 的假面模式提供了一种简单的方法去从服务容器中取出服务而不需要使用类型提示。使用契约可以使你明确的定义类间的依赖。而对于大多数应用来说，使用假面模式就可以了。但是，如果你想要松耦合易扩展的服务，那么契约可以实现。

## 为什么使用契约？

对于契约，你可能会产生很多的疑问。为什么要总是使用接口？使用接口不是会变的更复杂吗？让我们来提炼一下使用接口的原因：松耦合和简洁性。

### 松耦合

首先，让我们来看一下一个紧耦合的关于缓存的实现，请阅读下面的代码好好思考一下：

```php
<?php

namespace App\Orders;

class Repository
{
  /**
   * The cache instance.
   */
   protected $cache;

   /**
    * Create a new repository instance.
    *
    * @param \SomePackage\Cache\Memcached $cache
    * @return void
    */
    public function __construct(\SomePackage\Cache\Memcatched $cache)
    {
      $this->cache = $cache;
    }

    /**
     * Retrieve an Order by ID.
     *
     * @param int $id
     * @return Order
     */
     public function find($id)
     {
       if ($this->cache->has($id)) {
          //
       }
     }
}
```

在上面的类中，代码给予了类一个紧耦合的缓存的实现。之所以说是紧耦合的，是因为它依赖于一个具体的实现类，也就是说它和这个包的供应商是紧密耦合的。如果我们需要换一个包的供应商，那么我们就要修改我们的代码。

同样的，如果我们想要切换我们的底层缓存驱动，从 memcached 切换到 redis，那么我们也必须要修改大量的业务代码。所以，鉴于此，我们的资料库不应该对所提供的依赖需要知道的太多，而仅仅需要知道他们提供了我们所需的就可以了（这和鸭子类型的故事有点相似）。

鉴于此，我们可以采取一种与上述相反的方法来进行解耦。我们提供一个无关具体实现的接口：

```php
<?php

namespace App\Orders;

use Illuminate\Contracts\Cache\Repository as Cache;

class Respository
{
  /**
   * The cache instance.
   */
   protected $cache;

   /**
    * Create a new repository instance.
    *
    * @param Cache $cache
    * @return void
    */
    public function __construct(Cache $cache)
    {
      $this->cache = $cache;
    }
}
```

现在上面的代码并没有连接到任何的具体实现库，甚至是 laravel。因为契约只定义了接口，它并不包含任何的依赖和实现，所以你可以非常简单的去实现任何给定的契约，你只需要根据契约编写一个不同的实现就可以进行轻松的替换。而并不用修改之前写过的代码。

### 简单

当所有的 laravel 的服务都通过简单整洁的接口进行定义，那么它就可以被非常轻松的确定所给定服务具有哪些功能。这样也可以说 laravel 的契约服务其实是已经提供了简洁的功能文档了。

## 契约参考

下面是 laravel 契约及其对应的假面的参考:


Contract                                                                                                                          | References Facade
---                                                                                                                               | ---
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/master/Auth/Factory.php)                         | Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)           | Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)                     | Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/master/Broadcasting/Broadcaster.php) |
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php)                 | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php)                       | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php)               | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php)           | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php)                     | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php)     | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php)         | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php)               | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php)                 |
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php)             | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php)       | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php)     | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php)                     | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php)                           | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php)                     | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php)                           | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php)                       | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php)                           | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php)                     | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php)               | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php)   | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php)         | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php)               |
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php)                 |
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php)             |
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php)             | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php)         |
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php)                         | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php)                               |


## 怎么使用契约

那么，如何得到一个契约的实现呢？其实这相当的简单。


在 laravel 中很多类型的类都是通过服务容器来获取的，包括控制器，事件监听器，中间件，队列任务，和路由闭包。所以，你可以直接通过在构造函数中使用类型提示来构造接口依赖，这样在类被解析时会自动的获取契约的实例，我们通过一个简单的例子来看一下:

```php
<?php

namespace App\Listeners;

use App\User;
use App\Events\NewUserRegistered;
use Illuminate\Contracts\Redis\Database;

class CacheUserInformation
{
  /**
   * The Redis database implementation.
   */
   protected $redis;

   /**
    * Create a new event handler instance.
    *
    * @param Database $redis
    @ @return void
    */
    public function __construct(Database $redis)
    {
      $this->redis = $redis;
    }

    /**
     * Handle the event.
     *
     * @param NewUserRegistered $event
     * @return void
     */
     public function handle(NewUserRegistered $event)
     {
        //
     }
}
```

一旦事件监听器被解析，服务容器就会读构造函数中的类型提示，然后根据类型提示去注入适当的实例。关于更多的在服务容器中进行类的绑定注册，你可以查看服务容器的文档。