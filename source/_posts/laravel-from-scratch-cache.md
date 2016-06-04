title: laravel 基础教程 —— 缓存
date: 2016-06-02 15:51:28
tags: [php, laravel]
---

# 缓存

## 配置

Laravel 对多种缓存系统提供了统一的 API。缓存的配置文件存放在 `config/cache.php`。你可以在这个文件中指定整个应用默认使用何种缓存驱动。Laravel 支持当前主流的缓存系统如 Memcached 和 Redis。

缓存的配置文件也包含了一些额外的配置选项，这些选项在文件中都有文档注释，你应该确保自己已经读了这些选项注释。默认的，Laravel 配置使用 `file` 缓存驱动，该驱动会在文件系统中存储序列化的缓存对象。对于大型应用，建议使用内存级的缓存，如 Memcached 或者 APC。你甚至可以在 laravel 中配置多种缓存配置到相同的驱动。

### 缓存先决条件

**数据库**

当使用 `database` 缓存驱动时，你需要建立一个表来包含这些缓存项。你可以根据下面的 `Schema` 定义来建立表文件：

```php
Schema::create('cache', function ($table) {
  $table->string('key')->unique();
  $table->text('value');
  $table->integer('expiration'); 
});
```

你也可以通过使用 `php artisan cache:table` Artisan 命令来生成正确的缓存表结构迁移。

**Memcached** 

使用 Memcached 缓存需要安装 [Memcached PECL package](http://pecl.php.net/package/memcached)。

默认的配置基于 [Memcached::addServer](http://php.net/manual/en/memcached.addserver.php) 使用 TCP/IP：

```php
'memcached' => [
  [
    'host' => '127.0.0.1',
    'port' => 11211,
    'weight' => 100
  ]
],
```

你也可以使用 UNIX socket 路径来设置 `host`。如果你这么做，你需要设置 `port` 为 `0`:

```php
'memcached' => [
  [
    'host' => '/var/run/memcached/memcached.sock',
    'port' => 0,
    'weight' => 100
  ],
],
```

**Redis**

在你使用 Redis 缓存之前，你需要先通过 Composer 安装 `predis/predis`。
关于更多的 Redis 配置信息，你可以参考 [Laravel documentation page](https://laravel.com/docs/5.2/redis#configuration)。

## 缓存使用

### 获取缓存实例

`Illuminate\Contracts\Cache\Factory` 和 `Illuminate\Contracts\Cache\Repository` 契约提供对 Laravel 缓存服务的访问。`Factory` 契约提供了应用中所有缓存驱动的定义。`Repository` 契约通常是一个基于你的 `cache` 配置文件所使用的默认缓存驱动的实现。

事实上，你也可以使用 `Cache` 假面，在这篇文档中，我们都是使用 `Cache` 假面进行举例。`Cache` 假面提供了一种方便简洁的方式来访问 Laravel 底层缓存契约的实现。

例如，让我们引入 `Cache` 假面到控制器：

```php
<?php

namespace App\Http\Controllers;

use Cache;

class UserController extends Controller 
{
  /**
   * Show a list of all users of the application.
   *
   * @return Response
   */
   public function index()
   {
     $value = Cache::get('key');

     //
   }
}
```

**访问多种缓存存储**

你可以通过 `Cache` 假面的 `store` 方法来访问多种缓存存储。传递到 `store` 方法的 key 应该与你的 `cache` 配置文件中的 `stores` 配置项的列表之一相匹配：

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 10);
```

### 获取缓存项

你可以通过使用 `Cache` 假面的 `get` 方法来从缓存中获取相关项的值。如果该项在缓存中并不存在，则返回 `null` 。如果你需要，你也可以传递第二个参数到 `get` 方法，这个参数所传递的值会在缓存中项不存在时被返回:

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

你甚至可以传递 `Closure` 作为默认值。如果缓存的项不存在，`Closure` 所返回的值将被做为默认值。传递闭包的方式可以使你从数据库或者其他外部服务中延迟获取默认值：

```php
$value = Cache::get('key', function () {
  return DB::table(...)->get(); 
});
```

**检查项是否存在**

你可以使用 `has` 方法来检查缓存中是否存在该项：

```php
if (Cache::has('key')) {
  //
}
```

**递增/递减项中的值**

你可以使用 `increment` 和 `decrement` 方法来调整缓存项目中的整型值。这两个方法都可以接受一个数组作为第二个参数来进行相应的数值调整：

```php
Cache::increment('key');

Cache::increment('key', $amount);

Cache::decrement('key');

Cache::decrement('key', $amount);
```

**检索或更新缓存中的项**

有时候，你可能希望从缓存中检索出一个项目，但是当该项不存在的时候，你也想存储一个默认值到该项。比如，你希望从缓存中检索出一个用户。但是他并不存在，所以你需要从数据库中获取到他，然后添加到缓存中。你可以使用 `Cache::remember` 方法来做到这些：

```php
$value = Cache::remember('users', $minutes, function () {
  return DB::table('users')->get(); 
});
```

如果缓存中没有检索到该项，传递到 `remeber` 方法中的闭包将会被执行并且其执行结果将会在缓存中进行替换。

你也可以合并 `remember` 和 `forever` 方法：

```php
$value = Cache::rememberForever('users', function () {
  return DB::table('users')->get(); 
});
```

**检索并删除**

如果你需要检索一个项目，并在检索到的同时从缓存中删除该项，你可以使用 `pull` 方法。就像 `get` 方法一样，如果未检索到该项，将会返回 `null` :

```php
$value = Cache::pull('key');
```

### 存储项目到缓存

你可以使用 `Cache` 假面的 `put` 方法来存储项目到缓存中。当你存储一个项到缓存中时，你需要指定一个该项需要被缓存的分钟值:

```php
Cache::put('key', 'value', $minutes);
```

除了传递一个数值作为缓存过期的分钟值，你也可以通过传递一个 PHP `DateTime` 实例来设置缓存的失效时间：

```php
$expiresAt = Carbon::now()->addMinutes(10);

Cache::put('key', 'value', $expiresAt);
```

`add` 方法只会在相应的项在缓存中不存在时才会被添加进缓存。该方法会在项目被添加到缓存后返回 `true`。否则返回 `false`：

```php
Cache::add('key', 'value', $minutes);
```

`forever` 方法可以用来将项目永久的添加进缓存。该值只有手动的使用 `forget` 方法才能被移除：

```php
Cache::forever('key', 'value');
```

### 从缓存中移除项目

你可以使用 `Cache` 假面的 `forget` 方法来从缓存中移除某项：

```php
Cache::forget('key');
```

你可以使用 `flush` 方法来擦除所有的缓存：

```php
Cache::flush();
```

擦除缓存并不会根据前缀来进行智能擦除，它会移除所有的缓存。所以如果你的应用和其他的应用共享缓存，你应该谨慎的使用该方法。


## 缓存标签

> 注意： 缓存标签并不支持 `file` 或者 `database` 缓存驱动。另外，对于将多种标签标记为永久存储的驱动，性能最好的是能够提供自动清除过期记录的驱动，比如 `memcached`。

### 存储标记了的项目到缓存

缓存标签允许你将相关的项目进行关联标记。并且允许一次性清除所有给定标签的缓存项。你可以通过有序的标签数组来访问被标记的缓存项目。比如，让我们访问被标记的项目并使用 `put` 方法来设置缓存值：

```php
Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);
```

事实上，你并没有被限制只使用 `put` 方法。你可以在标签中使用任意的缓存存储方法。

### 访问被标记的缓存项

为了访问被标记了的缓存项，你需要传递相应的有序列表到 `tags` 方法：

```php
$john = Cache::tags(['people', 'artists'])->get('John');

$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

你可以一次性的擦除分配的标记或者标记列表中的所有项。比如，你可以使用 `flush` 方法来删除所有的 `people` 和 `authors` 标签和两者组成的有序列标签里的所有缓存项。所以，`Anne` 和 `John` 都会被从缓存中移除：

```php
Cache::tags(['people', 'authors'])->flush();
```

下面的语句将会作为上面语句的对照，将只会从缓存中删除 `authors` 标签的项目，所以 `Anne` 会被删除，而 `John` 被保留：

```php
Cache::tags('authors')->flush();
```

## 添加自定义的缓存驱动

为了在自定义的缓存驱动中继承 laravel 的缓存。我们需要使用 `Cache` 假面的 `extend` 方法，该方法被用来绑定自定义缓存到底层管理中。通常这些都是在服务提供者中完成。

比如，注册一个新的缓存驱动并命名为 'mongo':

```php
<?php

namespace App\Providers;

use Cache;
use App\Extensions\MongoStore;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
  /**
   * Perform post-registration booting of services.
   *
   * @return void
   */
   public function boot()
   {
     Cache::extend('mongo', function ($app) {
       return Cache::repository(new MongoStore);
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

第一个被传递到 `extend` 方法中的参数应该是驱动的名称。这个名称应该和你的 `config/cache.php` 配置文件中的 `driver` 选项一致。而第二个参数是一个闭包，该闭包应该返回一个 `Illuminate\Cache\Repository` 的实现。在闭包中将会被传递一个 `$app` 实例，这个实例是 laravel 中的服务容器的实例。

`Cache::extend` 方法的调用应该在 `App\Providers\AppServiceProvider` 的 `boot` 方法中完成。或者你可以创建自己的服务提供者来存储这个扩展。但是不要忘记在 `config/app.php` 文件中进行注册。

为了创建我们自己的缓存驱动，我们首先需要实现 `Illuminate\Constracts\Cache\Store` 契约的接口。所以，我们的 MongoDB 缓存实现应该看起来像这样：

```php
<?php

namespace App\Extensions;

class MongoStore implements \Illuminate\Contracts\Cache\Store
{
  public function get($key) {}
  public function put($key, $value, $minutes) {}
  public function increment($key, $value = 1) {}
  public function decrement($key, $value = 1) {}
  public function forever($key, $value) {}
  public function forget($key) {}
  public function flush() {}
  public function getPrefix() {}
}
```

我们仅仅需要使用 MongoDB 连接来实现这些方法。一旦我们的实现完成，我们就可以完成自己的缓存驱动的注册：

```php
Cache::extend('mongo', function ($app) {
  return Cache::repository(new MongoStore); 
});
```

然后在 `config/cache.php` 配置文件中更新驱动为的名称 `driver` 为你的扩展的名称。

如果你在为自定义的缓存文件应该存放在哪里而疑惑，你可以考虑将其发布到 Packagist！或者，你可以在 `app` 目录中创建一个 `Extensions` 命名空间。事实上，你应该谨记，laravel 并不死板的限制你的目录结构，你应该可以根据自己的习惯自由的管理你的应用目录结构。

## 事件

如果你想在任何缓存被操作时执行额外的代码，你可能需要监听缓存的触发事件。通常的你应该存放这些事件监听器到你的 `EventServiceProvider`:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
 protected $listen = [
  'Illuminate\Cache\Events\CacheHit' => [
    'App\Listeners\LogCacheHit',
  ],
  'Illuminate\Cache\Events\CacheMissed' => [
    'App\Listeners\LogCacheMissed',
  ],
  'Illuminate\Cache\Events\KeyForgotten' => [
    'App\Listeners\LogKeyForgotten',
  ],
  'Illuminate\Cache\Events\KeyWritten' => [
    'App\Listeners\LogKeyWritten',
  ],
 ];
```