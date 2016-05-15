title: laravel 基础教程 —— 服务提供者
date: 2016-05-15 21:34:31
tags: [php, laravel]
---

# 简介

服务提供者是 laravel 应用启动的中心。你自己的应用以及 laravel 的核心服务都是通过服务提供者来启动的。

但是，我们所指的启动是什么意思？通常情况下，这意味着注册，包含注册服务的绑定，事件监听，中间件，和路由。服务提供者是应用配置的中心。

如果你打开 `config/app.php` 文件，你会发现 `providers` 数组。这些都是你的应用将要加载的服务提供者类。当然，很多是延迟加载的提供者，意思就是并不是所有的请求都会加载这些提供者，而是只有需要的时候才会加载。

在此篇，你将学会如果去编写自己的服务提供者并且将其注册到应用中。

## 编写服务提供者

所有的服务提供者都继承自 `Illuminate\Support\ServiceProvider` 类。这个抽象类要求你的提供者必须最少定义一个 `register` 方法。在 `register` 方法中你应该只绑定内容到服务容器。你永远不要尝试在其中注册任何的事件监听，路由或者其它功能。

Artisan CLI 可以通过 `make:provider` 命令来非常便捷的生成一个新的提供者:

```php
php artisan make:provider RiakServiceProvider
```


### 注册方法

就如前面所提到的，在 `register` 方法中，你应该只做一件事，那就是绑定事物到服务容器中。不要做其它的事情。否则，可能你所使用的提供者提供的服务还没有被注册。
现在，让我们来看一个最基础的服务提供者：

```php
<?php

namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
  /**
   * Register bindings in the container.
   *
   * @return void
   */
   public function register()
   {
     $this->app->singleton(Connection::class, function ($app) {
         return new Connection(config('riak'));
     });
   }
}
```

这个服务提供者仅仅只定义了一个 `register` 方法，并且使用这个方法在服务容器中定义了一个 `Riak\Connection` 的实现。如果你并不理解服务容器是如何工作的，你可以看一下服务容器的文档。

### 启动方法

如果我们想在服务提供者中注册一个视图 composer，那么我们应该在 `boot` 方法中做这些。`boot` 方法会在所有的服务提供者都注册完成之后才会被执行。这意味着你有访问所有服务的权限:

```php
<?php
namespace App\Providers;

use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
  // Other Service Provider Properties...

  /**
   * Register any other events for your application.
   *
   * @param \Illuminate\Contracts\Events\Dispatcher $events
   * @return void
   */
   public function boot(DispatcherContract $events)
   {
     parent::boot($events);

     view()->composer('view', function () {
       //
     });
   }
}
```

### 启动方法的依赖注入

你可以在服务提供者的 `boot` 方法中使用类型提示来标识依赖。服务容器将会自动的将所需要的依赖注入进去：

```php
use Illuminate\Contracts\Routing\ResponseFactory;

public function boot(ResponseFactory $factory)
{
  $factory->macro('caps', function ($value)){
    //
  };
}
```

## 注册提供者

所有的服务提供者都被注册在 `config/app.php` 配置文件中。这个文件包含了 `providers` 数组，这数组列出了所有服务提供者的名字。默认的，laravel 中的核心服务都被注册在这个数组里。这些提供者启动了 laravel 中的核心组件，如邮件，队列，缓存和其他。

你可以在该数组中进行添加注册提供者：

```php
'providers' => [
  // Other Service Providers

  App\Providers\AppServiceProvider::class,
],
```

## 延迟加载提供者

如果你的提供者只是在服务容器中注册一些绑定信息，那么你可以选择推迟注册，这样这些服务只有在真正被需要用到时才会进行注册。推迟注册能够提高你的应用的性能。因为它不会在所有请求到来时都通过文件系统来加载。

为了推迟提供者的加载，你可以在提供者类中设置 `defer` 属性为 `true` 并且定义一个 `provides` 方法。`provides` 方法会返回提供者注册的服务容器的绑定:

```php
<?php

namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
  /**
   * Indicates if loading of the provider is deferred.
   *
   * @var bool
   */
   protected $defer = true;

   /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
      $this->app-singleton(Connection::class, function ($app) {
        return new Connection($app['config']['riak']);
      });
    }

    /**
     * Get the services provided by the provider.
     *
     * @return array
     */
     public function provides()
     {
        return [Connection::class];
     }
}
```

laravel 会编译和存储所有延迟加载的服务提供者的服务列表及服务提供者的类名。然后，只有当你尝试解析其中的某个服务时，laravel 才会加载其服务提供者。


