title: laravel 基础教程 —— 假面
date: 2016-05-18 21:39:07
tags: [php, laravel]
---

# 假面（Facades）

## 简介

假面提供了一种类似‘静态’接口的方式来从服务容器中提取可用类的方法。laravel 自身附带了许多的假面，你可能在还不知道它的时候就已经使用过它了。laravel 的假面服务就好像是一个从服务容器中取出底层类的代理，它提供了一种简洁语法的同时保持了比传统静态方法更高的可测试性和灵活性。

## 使用假面

在 laravel 中，假面是一种能够访问服务容器中对象的类。而使这项工作能够运行的就是 `Facade` 类。laravel 中的假面或者任何你所自定义的假面都必须要继承 `Illuminate\Support\Facades\Facade` 类。

一个假面类仅仅只需要去实现一个单一的 `getFacadeAccessor` 方法就可以了。而 `getFacadeAccessor` 方法的任务就是定义需要从服务容器中返回什么对象。`Facade` 基类使用了 `__callStatic()` 魔术方法让假面来延迟访问容器中相应的对象。

在下面的例子中，使用了 laravel 缓存系统的一个方法调用。如果只是匆匆的看一眼，你很可能会觉得这是在调用缓存类的静态方法 `get`:

```php
<?php

namespace App\Http\Controllers;

use Cache;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Show the profile for the given user.
   *
   * @param int $id
   * @return Response
   */
   public function showProfile($id)
   {
     $user = Cache::get('user' . $id);

     return view('profile', ['user' => $user]);
   }
}
```

你应该注意到我们在文件的顶部引入的是 `Cache` 假面。这个假面服务相当于访问 `Illuminate\Contracts\Cache\Factory` 接口底层实现的代理。所有使用假面调用的方法都会在 laravel 缓存服务的底层实现进行传递调用。

如果我们看一下 `Illuminate\Support\Facades\Cache` 类，你会发现这里并没有静态方法 `get`:

```php
class Cache extends Facade
{
  /**
   * Get the registered name of the component.
   *
   * @return string
   */
   protected static function getFacadeAccessor() 
   {
     return 'cache';
   }
}
```

事实上，`Cache` 假面继承自基类 `Facade` 并且定义了 `getFacadeAccessor()` 方法。这个方法的主要任务就是从服务容器中返回所绑定名称的对象。当用户调用任何 `Cache` 假面的静态方法时，laravel 从服务容器中返回绑定了 `cache` 为名称的对象，并且使用这个对象调用所请求的方法。

## 假面类参考

在下面的表格中你会找到所有的假面及其所对应的底层实现类。这是一个可以迅速挖掘给定假面 API 的有用工具。在服务容器中所给的绑定键也包括在其中。

Facade               | Class                                                                                                                        | Service Container Binding
---                  | ---                                                                                                                          | ---
App                  | [Illuminate\Foundation\Application](http://laravel.com/api/5.2/Illuminate/Foundation/Application.html)                       | app
Artisan              | [Illuminate\Contracts\Console\Kernel](http://laravel.com/api/5.2/Illuminate/Contracts/Console/Kernel.html)                   | artisan
Auth                 | [Illuminate\Auth\AuthManager](http://laravel.com/api/5.2/Illuminate/Auth/AuthManager.html)                                   | auth
Blade                | [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/5.2/Illuminate/View/Compilers/BladeCompiler.html)           | blade.compiler
Bus                  | [Illuminate\Contracts\Bus\Dispatcher](http://laravel.com/api/5.2/Illuminate/Contracts/Bus/Dispatcher.html)                   |
Cache                | [Illuminathe\Cache\Repository](http://laravel.com/api/5.2/Illuminate/Cache/Repository.html)                                  | cache
Config               | [Illuminate\Config\Repository](http://laravel.com/api/5.2/Illuminate/Config/Repository.html)                                 | config
Cookie               | [Illuminate\Cookie\CookieJar](http://laravel.com/api/5.2/Illuminate/Cookie/CookieJar.html)                                   | cookie
Crypt                | [Illuminate\Encryption\Encrypter](http://laravel.com/api/5.2/Illuminate/Encryption/Encrypter.html)                           | encrypter
DB                   | [Illuminate\Database\DatabaseManager](http://laravel.com/api/5.2/Illuminate/Database/DatabaseManager.html)                   | db
DB (Instance)        | [Illuminate\Database\Connection](http://laravel.com/api/5.2/Illuminate/Database/Connection.html)                             |
Event                | [Illuminate\Events\Dispatcher](http://laravel.com/api/5.2/Illuminate/Events/Dispatcher.html)                                 | events
File                 | [Illuminate\Filesystem\Filesystem](http://laravel.com/api/5.2/Illuminate/Filesystem/Filesystem.html)                         | files
Gate                 | [Illuminate\Contracts\Auth\Access\Gate](http://laravel.com/api/5.1/Illuminate/Contracts/Auth/Access/Gate.html)               |
Hash                 | [Illuminate\Contracts\Hashing\Hasher](http://laravel.com/api/5.2/Illuminate/Contracts/Hashing/Hasher.html)                   | hash
Lang                 | [Illuminate\Translation\Translator](http://laravel.com/api/5.2/Illuminate/Translation/Translator.html)                       | translator
Log                  | [Illuminate\Log\Writer](http://laravel.com/api/5.2/Illuminate/Log/Writer.html)                                               | log
Mail                 | [Illuminate\Mail\Mailer](http://laravel.com/api/5.2/Illuminate/Mail/Mailer.html)                                             | mailer
Password             | [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/5.2/Illuminate/Auth/Passwords/PasswordBroker.html)         | auth.password
Queue                | [Illuminate\Queue\QueueManager](http://laravel.com/api/5.2/Illuminate/Queue/QueueManager.html)                               | queue
Queue (Instance)     | [Illuminate\Contracts\Queue\Queue](http://laravel.com/api/5.2/Illuminate/Contracts/Queue/Queue.html)                         | queue
Queue (Base Class)   | [Illuminate\Queue\Queue](http://laravel.com/api/5.2/Illuminate/Queue/Queue.html)                                             |
Redirect             | [Illuminate\Routing\Redirector ](http://laravel.com/api/5.2/Illuminate/Routing/Redirector.html)                              | redirect
Redis                | [Illuminate\Redis\Database](http://laravel.com/api/5.2/Illuminate/Redis/Database.html)                                       | redis
Request              | [Illuminate\Http\Request](http://laravel.com/api/5.2/Illuminate/Http/Request.html)                                           | request
Response             | [Illuminate\Contracts\Routing\ResponseFactory](http://laravel.com/api/5.2/Illuminate/Contracts/Routing/ResponseFactory.html) |
Route                | [Illuminate\Routing\Router](http://laravel.com/api/5.2/Illuminate/Routing/Router.html)                                       | router
Schema               | [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/5.2/Illuminate/Database/Schema/Blueprint.html)                 |
Session              | [Illuminate\Session\SessionManager ](http://laravel.com/api/5.2/Illuminate/Session/SessionManager.html)                      | session
Session (Instance)   | [Illuminate\Session\Store ](http://laravel.com/api/5.2/Illuminate/Session/Store.html)                                        |
Storage              | [Illuminate\Contracts\Filesystem\Factory](http://laravel.com/api/5.2/Illuminate/Contracts/Filesystem/Factory.html)           | filesystem
URL                  | [Illuminate\Routing\UrlGenerator](http://laravel.com/api/5.2/Illuminate/Routing/UrlGenerator.html)                           | url
Validator            | [Illuminate\Validation\Factory](http://laravel.com/api/5.2/Illuminate/Validation/Factory.html)                               | validator
Validator (Instance) | [Illuminate\Validation\Validator](http://laravel.com/api/5.2/Illuminate/Validation/Validator.html)                           |
View                 | [Illuminate\View\Factory](http://laravel.com/api/5.2/Illuminate/View/Factory.html)                                           | view
View (Instance)      | [Illuminate\View\View](http://laravel.com/api/5.2/Illuminate/View/View.html)                                                 |