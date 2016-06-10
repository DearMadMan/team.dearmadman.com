title: laravel 基础教程 —— 事件
date: 2016-06-06 10:58:41
tags: [php, laravel]
---

# 事件

## 简介

laravel 的事件提供了一种简单的观察者实现。它允许你在应用中进行订阅和监听事件。事件类通常都是存储在 `app/Events` 目录中，而他们的监听者都是存储在 `app/Listeners` 目录中。

## 注册事件/监听者

`EventServiceProvider` 提供了一个注册所有事件监听者的方便的场所。它的 `listen` 属性包含了一个所有事件（keys）以及他们的监听者（values）所组成的数组。当然，你可以在这个数组中添加任何你需要的事件。比如，让我们添加 `PodcastWasPurchased` 事件：

```php
/**
 * The event listener mappiings for the application.
 *
 * @var array
 */
 protected $listen = [
   'App\Events\PodcastWasPurchased' => [
     'App\Listeners\EmailPurchaseConfirmation',
   ],
 ],
```

### 生成事件/监听者类

当然，每次都手动的去创建一个事件文件和一个监听者文件是很麻烦的事情，所以，你可以在 `EventServiceProvider` 中添加事件和监听者然后使用 `event:generate` 命令来自动生成。这个命令会生成 `EventServiceProvider` 中的所有列出的事件和监听者，当然，已经存在的事件和监听者不会重新生成：

```php
php artisan event:generate
```

## 手动的注册事件

通常，事件应该被注册在 `EventServiceProvider` 的 `$listen` 数组中，事实上，你可以使用 `Event` 假面的事件分发器或者一个 `Illuminate\Contracts\Events\Dispatcher` 的实现来手动注册事件:

```php
/**
 * Register any other events for your application
 *
 * @param \Illuminate\Contracts\Events\Dispatcher $events
 * @return void
 */
 public function boot(DispatcherContract $events)
 {
   parent::boot($events);

   $events->listen('event.name', function ($foo, $bar) {
     //
   });
 }
```

**事件监听通配符**

你可以在注册监听器的时候使用 `*` 来作为通配符。这允许你来在同一个监听器中监听多种事件。通配符监听器接收整个事件数据数组作为第一个参数：

```php
$events->listen('event.*', function (array $data) {
  // 
});
```

## 定义事件

一个事件类只是简单的数据存储器，它应该持有事件相关的信息.比如，让我们假设我们生成的 `PodcastWasPurchased` 事件应该接收一个 Eloquent ORM 对象：

```php
<?php

namespace App\Events;

use App\podcast;
use App\Events\Event;
use Illuminate\Queue\SerializesModels;

class PodcastWasPurchased extends Event
{
  use SerializesModels;

  public $podcast;

  /**
   * Create a new event instance.
   *
   * @param Podcast $podcast
   * @return void
   */
   public function __construct(Podcast $podcast)
   {
     $this->podcast = $podcast;
   }
}
```

就如你所看到的，这个事件类并没有包含什么业务逻辑。它只是简单的包含了一个被购买的 `Podcast` 对象。`SerializesModels` trait 被用来序列化 Eloquent 模型，如果事件对象是使用 PHP 的 `serialize` 方法被进行序列化，那么 `SerializesModels` 就会优雅的将其内部的 Eloquent 对象序列化。

## 定义监听者

接着，让我们来看一下针对我们上面举例的事件的监听者。事件监听者会在其 `handle` 方法中接收事件的实例。`event:generate` 命令会自动的在 `handle` 方法中引入其对应的事件的类型提示。你可以在 `handle` 方法中来提供对事件的响应逻辑：

```php
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;

class EmailPurchaseConfirmation
{
  /**
   * Create the event listener.
   *
   * @return void
   */
   public function __construct()
   {
     //
   }

   /**
    * Handle the event.
    *
    * @param PodcastWasPurchased $event
    * @return void
    */
    public function handle(PodcastWasPurchased $event)
    {
      // Access the podcast using $event->podcast...
    }
}
```

你的事件监听者也可以在构造函数中进行类型提示来注入依赖。所有的事件监听器都是通过 laravel 的服务容器解析的。所以，它们的依赖可以被自动的注入：

```php
use Illuminate\Contracts\Mail\Mailer;

public function __construct(Mailer $mailer)
{
  $this->mailer = $mailer;
}
```

**停止传递事件**

有时候，你可能希望停止传递事件到后续的监听者中。你可以在监听者的 `handle` 方法中返回 `false`，这样后续的监听者将不再进行事件的响应。

### 监听者队列化

需要对事件监听者进行队列化？没有比这更简单的了。直接添加 `ShouldeQueue` 接口到监听者类中就可以了。通过 `event:generate` Artisan 命令生成的监听者已经在当前的命名空间中添加了对这个接口的支持，所以你立即就可以使用：

```php
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation implements ShouldQueue
{
  //
}
```

就这么简单！当事件触发时，事件分发器会使用 laravel 的队列系统对监听器进行自动的队列化调用。如果监听器队列化执行的过程中没有异常出现，队列任务会自动的在监听器进程完成后进行删除。

**手动的访问队列**

如果你需要手动的访问底层队列任务的 `delete` 和 `release` 方法。那么你就可以直接去使用它们。`Illuminate\Queue\InteractsWithQueue` trait 对默认生成的监听器提供了访问这些方法的权限：

```php
<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation implements ShouldQueue
{
  use InteractsWithQueue;

  public function handle(PodcastWasPurchased $event)
  {
    if (true) {
      $this->release(30);
    }
  }
}
```

## 触发事件

你可以使用 `Event` 假面来进行事件的触发，你需要传递一个事件实例到 `fire` 方法中。`fire` 方法会分发事件到所有的监听器中：

```php
<?php

namespace App\Http\Controllers;

use Event;
use App\Podcast;
use App\Events\PodcastWasPurchased;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Show the profile for the given user.
   *
   * @param int $userId
   * @param int $podcastId
   * @return Response
   */
  public function purchasePodcast($userId, $podcastId)
  {
    $podcast = Podcast::findOrFail($podcastId);

    // Purchase podcast logic...

    Event::fire(new PodcastWasPurchased($podcast));
  } 
}
```

此外，你还可以通过使用全局 `event` 帮助函数来触发事件：

```php
event( new PodcastWasPurchased($podcast));
```

## 广播事件

很多现代化的应用中，会使用 web sockets 来实现实时交互的用户接口。当一些数据在服务端变更时，一条消息会通过 websocket 连接来传递到客户端进行处理。

为了帮助你构建这种类型的应用。laravel 使通过 websocket 连接进行广播事件变的非常简单。laravel 允许你广播事件来共享事件的名称到你的服务端和客户端的 JavaScript 框架。

### 配置

所有的事件广播配置选项都被存储在 `config/broadcasting.php` 配置文件中。laravel 支持多种开箱即用的广播驱动：[Pusher](https://pusher.com/)，[Redis](https://laravel.com/docs/5.2/redis) 和 `log` 驱动来提供本地开发和调试。在这个配置文件中包含了每种驱动的配置示例。

**广播先决条件**

事件广播需要以下依赖：
- Pusher: `pusher/pusher-php-server ~2.0`
- Redis: `predis/predis ~1.0`

**队列化先决条件**

在进行事件广播之前，你需要先配置好队列监听器。所有的广播都是通过队列任务来异步执行的，这样应用响应就不会受到影响。

### 对事件进行广播

你需要实现 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 接口在事件类中，以通知 laravel 所给定的事件需要被广播。`ShouldBroadcast` 接口只要求你实现一个单一的方法：`broadcastOn`.该方法应该返回一个事件应该被广播的通道名称：

```php
<?php

namespace App\Events;

use App\User;
use App\Events\Event;
use Illuminate\Queue\SerializesModles;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ServerCreated extends Event implements ShouldBroadcast
{
  use SerializesModels;

  public $user;

  /**
   * Create a new event instance.
   *
   * @return void
   */
  public function __construct(User $user)
  {
    $this->user = $user;
  }

  /**
   * Get the channels the event should be broadcast on.
   *
   * @return array
   */
   public function broadcastOn()
   {
     return ['user.' . $this->user->id];
   }
}
```

然后，你只需要像平常那样去触发事件就可以了。一旦事件被触发，队列任务会自动的广播事件到你指定的广播驱动中。

### 广播数据

当一个事件被广播时，事件类的所有 `public` 属性都会被序列化并作为广播的载荷，允许你的前端 JavaScript 应用来访问所有的开放属性。比如，如果事件类中只有一个开放的 `$user` 属性，该属性包含了一个 Eloquent 模型，那么广播的载荷将会像这样：

```php
{
  "user": {
    "id": 1,
    "name": "Jonathan Banks"
    ...
  }
}
```

事实上，你可以通过在事件中添加 `broadcastWith` 方法来对广播载荷拥有更多的控制。该方法应该返回一个数组：

```php
/**
 * Get the data to broadcast.
 *
 * @return array
 */
 public function broadcastWith()
 {
   return ['user' => $this->user->id];
 }
```

### 定制化事件广播

**定制事件名称**

默认的，广播事件的名称会使用事件类的全名。比如，如果事件的类名是 `App\Events\ServerCreated`，那么广播事件的名称就是 `App\Events\ServerCreated`。你可以通过定义 `broadcastAs` 方法来自定义广播事件的名称：

```php
/**
 * Get the broadcast event name
 *
 * @return string
 */
 public function broadcastAs()
 {
   return 'app.server-created';
 }
```

**定制化队列**

默认的，所有的事件广播都是通过 `queue.php` 配置文件中所指定的默认队列配置来进行的。你可以通过在事件广播类中添加一个 `onQueue` 方法来指定所使用的队列。该方法应该返回一个你所期望使用的队列的名称：

```php
/**
 * Set the name of the queue the event should be placed on.
 *
 * @return string
 */
 public function onQueue()
 {
   return 'your-queue-name';
 }
```

### 对接广播事件

**Pusher**

你可以使用 [Pusher](https://pusher.com/) 驱动的 JavaScript SDK来轻松的对接广播事件。比如，让我们消费前面例子中的 `App\Events\ServerCreated` 事件：

```javascript
this.pusher = new Pusher('pusher-key');

this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

this.pusherChannel.bind('App\\Events\\ServerCreated', function (message) {
  console.log(message.user); 
});
```

**Redis**

如果你使用的是 Redis 广播。那么你可能需要编写自己的 Redis 发布/订阅消费者来接收消息，你可以自由的选择使用 websocket 技术来进行事件的广播。比如，你可以选择使用基于 Node 受欢迎的 [Socket.io](http://socket.io/) 类库。

你可以使用 `socket.io` 和 `ioredis` Node 类库来快速的编写事件广播员来发布所有的应用事件：

```javascript

var app = require('http').createServer(handler);
var io = require('socket.io')(app);

var Redis = require('ioredis');
var redis = new Redis();

app.listen(6001, function () {
  console.log('Server is running!') ;
});

function handler(req, res) {
  res.writeHead(200);
  res.end('');
}

io.on('connection', function (socket) {
  // 
});

redis.psubscribe('*', function (err, count) {
  // 
});

redis.on('pmessage', function (subscrbed, channel, message) {
  message = JSON.parse(message);
  io.emit(channel + ':' + message.event, message.data);
});
```

## 事件订阅

事件订阅允许一个类在其内部订阅多个事件。这允许你在同一个文件中对多个事件进行处理。订阅者应该定义 `subscribe` 方法。该方法会被传递一个事件分发器实例：

```php
<?php

namespace App\Listeners;

class UserEventListener
{
  /**
   * Handle user login events.
   */
   public function onUserLogin($event) {}

   /**
    * Handle user logout events.
    */
    public function onUserLogout($event) {}

    /**
     * Register the listeners for the subscriber.
     *
     * @param Illuminate\Events\Dispatcher $events
     */
     public function subscribe($events)
     {
       $events->listen(
         'App\Events\UserLoggedIn',
         'App\Listeners\UserEventListener@onUserLogin'
        );
       $events->listen(
         'App\Events\UserLoggedOut',
         'App\Listeners\UserEventListener@onUserLogout'
        );
     }
}
```

**注册一个事件订阅者**

当你的订阅者被定义完成之后，它应该在事件分发器中进行注册。你可以使用 `EventServieProvider` 的 `$subscribe` 属性来注册订阅者。比如，让我们来添加一个 `UserEventListener`:

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
  /**
   * The event listener mapping for the application.
   *
   * @var array
   */
   protected $listen = [
     //
   ];

   /**
    * The subscriber classes to register.
    *
    * @var array
    */
    protected $subscribe = [
      'App\Listeners\UserEventListener',
    ];
}
```