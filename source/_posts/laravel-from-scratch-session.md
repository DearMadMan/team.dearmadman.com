title: laravel 基础教程 —— Session
date: 2016-06-14 16:29:28
tags: [php, laravel]
---

# Session

## 简介

由于 HTTP 驱动是一种无状态的协议，这通常意味着服务端并不能清楚的知道当前请求用户与之前请求用户间的关系，而 Session 提供了这种跨请求用户的解决方案。Laravel 附带了简洁统一的 API 来支持各种后端 session 驱动。对于 [Memcached](http://memcached.org/)，[Redis](http://redis.io/) 和数据库这种流行的后端驱动，Laravel 提供了开箱即用的支持。

### 配置

session 的配置文件被存储在 `config/session.php`。你应该确保在使用前阅读该文件中的配置项注释。Laravel 对配置中的每一项都进行了详细的注释。默认的，laravel 配置使用的是 `file` session 驱动，这对于大多数应用来说已经够用了。但是在生产环境中还是建议使用 `memcached` 或者 `redis` 这种内存级驱动，这对于应用的 session 性能来说会是一个很大的提升。

对于每个请求的 session 数据，都会存储在你所定义的 session `driver` 所在的位置中。下列驱动在 laravel 中可以开箱即用：
- `file` - sessions 会存储在 `storage/framework/sessions` 中。
- `cookie` - sessions 会被存储在经过安全加密的 cookies 中。
- `database` - sessions 会被存储在你应用中所使用的数据库中。
- `memcached` / `redis` - sessions 会被存储在更快的内存级存储驱动中。
-  `array` - sessions 会使简单存储在 PHP 的数组中，并且它不能被跨请求访问。

> 注意：`array` 驱动通常是用来进行测试时使用的以避免持久的 session 数据。

### 驱动先决条件

**数据库**

当使用 `database` session 驱动时，你需要先创建一个表来存储 session 项。下面的例子是数据库 session 驱动表的架构声明：

```php
Schema::create('sessions', function ($table) {
  $table->string('id')->unique();
  $table->integer('user_id')->nullable();
  $table->string('ip_address', 45)->nullable();
  $table->text('user_agent')->nullable();
  $table->text('payload');
  $table->integer('last_activity');
});
```

你也可以使用 `session:table` Artisan 命令来生成一个数据库会话驱动的迁移表：

```php
php artisan session:table

composer dump-autoload

php artisan migrate
```

**Redis**

在使用 Redis session 驱动之前，你需要通过 Composer 安装 `predis/predis`（~1.0）。

**其他 Session 注意事项**

Laravel 框架内部使用了 `flash` 作为 session 其中的一个键，所以你应该避免使用该名称的键到 session 中。

如果你需要存储安全加密后的 session 数据，你可以在配置文件中将 `encrypt` 选项设置为 `true`。

## 基础用法

**访问 Session**

首先，让我们来访问 session。我们可以通过 HTTP 请求来访问 session 的实例。HTTP 请求可以通过在控制器方法中使用类型提示来进行依赖注入。你应该记得，控制器方法的依赖会通过 laravel 的服务容器进行注入：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Show the profile for the given user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
   public function showProfile(Request $request, $id)
   {
     $value = $request->session()->get('key');

     //
   }
}
```

当你从 session 中检索值时，你也可以指定一个默认的值到 `get` 方法的第二个参数，默认值会在 session 未检索到相应键的值时返回。你也可以传递 `Closure` 作为默认值，如果找不到相应的键，`Closure` 执行的结果所返回的值将作为默认值：

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
  return 'default'; 
});
```

如果你希望从 session 中检索出所有的值，你可以使用 `all` 方法：

```php
$data = $request->session()->all();
```

你可以可以使用全局帮助方法 `session` 来进行值的检索和存储：

```php
Route::get('home', function () {
  // Retrieve a piece of data from the session...
  $value = session('key');

  // Store a piece of data in the session...
  session(['key' => 'value']);
});
```

**判断 session 是否存在某项**

你可以使用 `has` 方法来判断 session 中是否含有某项，如果存在则返回 `true`:

```php
if ($request->session()->has('user')) {
  //
}
```

**在 session 中存储数据**

一旦你获取了 session 的实例，你就拥有了和底层数据进行交互的能力，你可以通过各种实例的方法来进行交互。比如，你可以使用 `put` 方法来在 session 中存储一个新的数据项：

```php
$request->session()->put('key', 'value');
```

**推送内容到 session 中值为数组的项**

你可以使用 `push` 方法来将一个新的值推送到 session 中值为数组的项中。比如，如果 `user.teams` 键表明了数组在 session 中的路径，你可以像这样推送新的值到该数组中：

```php
$request->session()->push('user.teams', 'developers');
```

**检索并删除某项**

`pull` 方法会在检索的同时在 session 中删除该项：

```php
$value = $request->session()->pull('key', 'default');
```

**删除 session 中的某项**

你可以使用 `forget` 方法来删除 session 中的某一项数据，如果你想要移除 session 中的所有数据，你可以使用 `flush` 方法：

```php
$request->session()->forget('key');

$request->session()->flush();
```

**重新生成 session ID**

如果你需要重新生成 session ID，你可以使用 `regenerate` 方法：

```php
$request->session()->regenerate();
```

### 闪存数据

有时候你希望在 session 中存储某些数据，但是只允许下一次请求时使用，使用后即消除。那么就可以使用 `flash` 方法。这方法在 session 中存储的数据只能在随后的请求中进行访问，并且在之后删除。闪存的数据通常用来做一些临时的状态消息：

```php
$request->session()->flash('status', 'Task was successful!');
```

如果你需要维持闪存的数据到更多的请求，你可以使用 `reflash` 方法，它会维持所有的闪存数据到额外的请求中。如果你只需要维持指定的闪存数据，你可以使用 `keep` 方法：

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

## 添加自定义的 Session 驱动

你需要使用 `Session` 假面的 `extend` 方法才能添加额外的 session 驱动到 Laravel 中。你可以在服务提供者的 `boot` 方法中来调用 `extend` 方法：

```php
<?php

namespace App\Providers;

use Session;
use App\Extensions\MongoSessionStore;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
  /**
   * Perform post-registration booting of services.
   *
   * @return void
   */
   public function boot()
   {
     Session::extend('mongo', function ($app) {
       // Return implementation of SessionHandlerInterface...
       return new MongoSessionStore;
     });
   }

   /**
    * Register bindings in the container.
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

你应该注意你的自定义 session 驱动应该实现 `SessionHandlerInterface` 接口。该接口仅包含了几个简单的需要实现的方法。一个实现了的 MongoDB 驱动的结构应该像下面一样：

```php
<?php

namespace App\Extensions;

class MongoHandler implements SessionHandlerInterface
{
  public function open($savePath, $sessionName) {}
  public function close() {}
  public function read($sessionId) {}
  public function write($sessionId, $data) {}
  public function destroy($sessionId) {}
  public function gc($lifetime) {}
}
```

由于这些方法并不像缓存的 `StoreInterface` 那样难于理解。所以，我们就来快速的过一下吧：
- `open` 方法一般用于基于文件的 session 存储系统。由于 laravel 已经附带了 `file` session 驱动，所以你几乎不需要在这个方法中添加任何的代码。你可以直接放置其为空方法。事实上，这是一种可怜的接口设计（我们将会在后面讨论它），PHP 必须要我们实现这个方法。
- `close` 方法，类似于 `open` 方法，你也可以对其进行忽视，对于大多数驱动来说根本不需要。
- `read` 方法应该根据给定的 `$sessionId` 返回 session 中关联数据的字符串版本。你不需要在存储或检索时做任何的序列化或其它的编码操作，因为 laravel 会为你执行序列化服务。
- `destroy` 方法应该从 session 存储中删除所关联的数据。
- `gc` 方法应该根据给定的 `$lifetime` UNIX 时间戳删除 session 所关联的过期数据。对于拥有自动过期能力的系统，比如 Memcached 和 Redis，你可以直接让这个方法为空。

一旦你的 session 驱动被注册。你就可以在 `config/session.php` 配置文件中使用 `mongo` 驱动了。
