title: laravel 基础教程 —— Redis
date: 2016-06-10 19:53:11
tags: [php, laravel]
---

# Redis

## 简介

[Redis](http://redis.io/) 是一个开源高效的键值对存储系统。它通常用作为一个数据结构服务器来存储键值对，它可以支持字符串，散列，列表，集合，和有序集合。在 laravel 中使用 Redis 之前，你需要通过 Composer 来安装 `predis/predis` 包（~1.0）。

## 配置

Redis 在应用中的配置文件存储在 `config/database.php`。在这个文件中，你可以看到一个包含了 Redis 服务信息的 `redis` 数组：

```php
'redis' => [
  'cluster' => false,

  'default' => [
    'host' => '127.0.0.1',
    'port' => 6379,
    'database' => 0,
  ],
],
```

对于开发来说，默认的配置已经完全可以满足大部分的应用了。但是，你可以自由的在你环境中修改这个配置。你可以简单的添加 Redis 服务的名称，并且指定相应的服务器地址和端口。

`cluster` 选项会告诉 Laravel Redis 客户端在你的 Redis 集群中进行客户端的分片，这样就可以构成节点池并且创建大量有效的 RAM。但是，你需要注意的是客户端分片并不能处理故障转移。因此，它主要用来从一个主要数据存储地址获取可用的缓存数据。

另外，你可以在你的 Redis 连接定义里添加一个 `options` 数组，这样你可以指定 Predis 的 [客户端选项](https://github.com/nrk/predis/wiki/Client-Options)。

如果你的 Redis 服务器引入了认证机制，那么你需要在你的 Redis 服务配置数组中添加一个 `password` 配置项来提供密码。

> 注意：如果你通过 PECL 来安装的 Redis PHP 扩展，那么你需要在 `config/app.php` 文件中对 Redis 的别名进行重命名。

## 基础用法

你可以通过使用 `Redis` 假面的各种方法来与 Redis 进行交互。`Redis` 假面支持动态方法，这意味着你可以在 `Redis` 假面上调用任何的 Redis [命令](http://redis.io/commands)，假面会直接将命令传递给 Redis。比如，我们通过 `Redis` 假面的 `get` 方法来调用 `GET` 命令：

```php
<?php

namespace App\Http\Controllers;

use Redis;
use App\Http\Controllers\Controler;

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
     $user = Redis::get('user:profile:'. $id);

     return view('user.profile', ['user' => $user]);
   }
}
```

当然，就如我们前面所提到的，你可以通过 `Redis` 假面调用任何的 Redis 命令。Laravel 使用魔术方法来将其传递到 Redis 服务器，所以，可以直接简单的传递命令所需的参数即可：

```php
Redis::set('name', 'Taylor');

$value = Redis::lrange('names', 5, 10);
```

另外，你也可以通过 `command` 方法来将命令传递到服务器，它接收第一个参数作为命令名称，第二个参数数组作为传递命令的参数值：

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

**使用多 Redis 连接**

你可以通过使用 `Redis::connection` 方法来获取 Redis 的实例：

```php
$redis = Redis::connection();
```

这会返回默认的 Redis 服务器的实例。如果你没有使用集群服务，你可以传递配置文件中所定义的服务名称到 `connection` 方法中：

```php
$redis = Redis::connection('other');
```

### 流水线命令

管道流水线可以允许你在一个操作中对 Redis 服务器执行多个命令。`pipeline` 方法接收一个参数: `Closure` ，它会接收 Redis 的实例。你可以在闭包中发布所有的命令，它们将会在一个操作中进行处理：

```php
Redis::pipeline(function ($pipe) {
  for ($i = 0; $i < 1000; $i++) {
    $pipe->set("key:$i", $i);
  } 
});
```

## 发布、订阅

Laravel 也对 Redis 的 `publish` 和 `subscribe` 命令提供了一种方便的接口。这些 Redis 命令允许你通过给定的频道来监听信息。你可以从其他应用中向频道中发布信息，更甚者你可以使用其它的语言来进行发布，你可以轻而易举的在应用 / 进程间进行通讯:

```php
<?php

namespace App\Console\Commands;

use Redis;
use Illuminate\Console\Command;

class RedisSubscribe extends Command
{
  /**
   * The name and signature of the console command.
   *
   * @var string
   */
   protected $signature = 'redis:subscribe';

   /**
    * The console command description.
    *
    * @var string
    */
    protected $description = 'Subscribe to a Redis channel';

    /**
     * Execute the console command.
     *
     * @return mixed
     */
     public function handle()
     {
       Redis::subscribe(['text-channel'], function ($message) {
         echo $message;
       });
     }
}
```

现在，你可以使用 `publish` 方法来向频道中发布信息：

```php
Route::get('publish', function () {
  // Route logic...

  Redis::publish('test-channel', json_encode(['foo' => 'bar']));
})
```

**订阅通配符**

你可以使用 `psubscribe` 方法来订阅统配符匹配的频道，这通常用来捕获频道中的所有消息。`$channel` 变量会自动的传递到 `Closure` 的第二个参数：

```php
Redis::psubscribe(['*'], function ($message, $channel) {
  echo $message; 
});

Redis::psubscribe(['users.*'], function ($message, $channel) {
  echo $message; 
});
```