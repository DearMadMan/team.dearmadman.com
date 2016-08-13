title: Laravel 即时应用的一种实现方式
date: 2016-08-08 10:17:43
tags: [php, laravel]
---

## 即时交互的应用

在现代的 Web 应用中很多场景都需要运用到即时通讯，比如说最常见的支付回调，与三方登录。这些业务场景都基本需要遵循以下流程：

* 客户端触发相关业务，并产生第三方应用的操作（比如支付）
* 客户端等待服务端响应结果（用户完成第三方应用的操作）
* 第三方应用通知服务端处理结果（支付完成）
* 服务端通知客户端处理结果
* 客户端依据结果做出反馈 （跳转到支付成功页面）

在过去，为了实现这种即时通讯，能让客户端正确响应处理结果，最为常用的技术就是轮询，因为 HTTP 协议的单向性，客户端只能一遍一遍的主动询问服务端的处理结果。这种方式有显见的缺陷，占用服务端资源不说，还不能实时获得服务端处理结果。

现在，我们可以使用 WebSocket 协议来处理实时交互，它是一种双向协议，允许服务端主动推送信息到客户端。本篇我们将借助 Laravel 强大的事件系统来构建实时的交互。你将需要用到以下知识：

* Laravel Event
* Redis
* Socket.io
* Node.js

## Redis

在开始之前，我们需要开启一个 redis 服务，并在 Laravel 应用中进行配置启用，因为在整个流程中，我们需要借助 redis 的订阅和发布机制来实现即时通讯。

Redis 是一个开源高效的键值对存储系统。它通常作为一个数据结构服务器来存储键值对，它可以支持字符串，散列，列表，集合和有序结合。在 Laravel 中使用 Redis 你需用通过 Composer 来安装 `predis/predis` 包文件。

### 配置

Redis 在应用中的配置文件存储在 `config/database.php`，在这个文件中，你可以看到一个包含了 Redis 服务信息的 `redis` 数组:

```
'redis' => [
  'cluster' => false,

  'default' => [
    'host' => '127.0.0.1',
    'port' => 6379,
    'database' => 0,
  ],
]
```

如果你修改了 redis 服务的端口，请保持配置文件中的端口一致。

## Laravel Event

这里我们需要借助 Laravel 强大的事件广播能力：

> **广播事件**
> 
> 很多现代化的应用中，会使用 Web Sockets 来实现实时交互的用户接口。当一些数据在服务端变更时，一条消息会通过 WebSocket 连接来传递到客户端进行处理。
> 为了帮助你构建这种类型的应用。Laravel 使通过 WebSocket 连接进行广播事件变的非常简单。Laravel 允许你广播事件来共享事件的名称到你的服务端和客户端的 JavaScript 框架。

### 配置

所有的事件广播配置选项都被存储在 `config/broadcasting.php` 配置文件中。Laravel 附带了几种可用的驱动如 Pusher，Redis，和 Log，我们将使用 Redis 作为广播驱动，这里需要依赖 `predis/predis` 类库。

由于默认的广播驱动使用的是 [pusher](https://pusher.com/)，所以我们需要在 `.env` 文件中设置 `BROADCAST_DRIVER=redis`。

我们创建一个 `WechatLoginedEvent` 事件类用来在用户扫描微信登录后进行广播：

```php
<?php

namespace App\Events;

use App\Events\Event;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class WechatLoginedEvent extends Event implements ShouldBroadcast
{
    use SerializesModels;

    public $token;
    protected $channel;

    /**
     * Create a new event instance.
     *
     * @param  string $token
     * @param  string $channel
     * @return void
     */
    public function __construct($token, $channel)
    {
        $this->token = $token;
        $this->channel = $channel;
    }

    /**
     * Get the channels the event should be broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return [$this->channel];
    }

    /**
     * Get the name the event should be broadcast on.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'wechat.login';
    }
}
```

其中你需要注意 `broadcastOn` 方法应返回一个数组，它表示所需广播的频道，而 `broadcastAs` 返回的是一个字符串，它表示广播所触发的事件，Laravel 默认的是返回事件类的全类名，这里是 `App\Events\WechatLoginedEvent`.

最重要的是你需要手动的让该类实现 `ShouldBroadcast` 契约。Laravel 在生成事件时，已经自动添加了该命名空间，该契约只约束 `broadcastOn` 方法。

事件完成接下来就是触发事件了，简单的一行代码就可以：

``` php
event(new WechatLoginedEvent($token, $channel));
```

这个操作会自动的触发事件的执行并将信息广播出去。该广播操作底层借助了 redis 的订阅和发布机制。`RedisBroadcaster` 会将事件中的允许公开访问的数据通过给定的频道发布出去。如果你想对公开的数据拥有更多的控制，你可以在事件中添加 `broadcastWith` 方法，它应该返回一个数组：

``` php
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

## Node.js 和 Socket.io

对于发布出去的信息，我们需要一个服务来对接，让其能对 redis 的发布能够进行订阅，并且能把信息以 WebSocket 协议转发出去，这里我们可以借用 Node.js 和 socket.io 来非常方便的构建这个服务：

``` javascript
// server.js
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
  socket.on('message', function (message) {
    console.log(message)
  })
  socket.on('disconnect', function () {
    console.log('user disconnect')
  })
});


redis.psubscribe('*', function (err, count) {
});

redis.on('pmessage', function (subscrbed, channel, message) {
  message = JSON.parse(message);
  io.emit(channel + ':' + message.event, message.data);
});
```

这里我们使用 Node.js 引入 socket.io 服务端并监听 6001 端口，借用 redis 的 psubscribe 指令使用通配符来快速的批量订阅，接着在消息触发时将消息通过 WebSocket 转发出去。

## Socket.io 客户端

在 web 前端，我们需要引入 Socket.io 客户端开启与服务端 6001 端口的通讯，并订阅频道事件：

``` javascript
// client.js
let io = require('socket.io-client')

var socket = io(':6001')
      socket.on($channel + ':wechat.login', (data) => {
        socket.close()
        // save user token and redirect to dashboard
})
```

至此整个通讯闭环结束，开发流程看起来就是这样的：
* 在 Laravel 中构建一个支持广播通知的事件
* 设置需要进行广播的频道及事件名称
* 将广播设置为使用 redis 驱动
* 提供一个持续的服务用于订阅 redis 的发布，及将发布内容通过 WebSocket 协议推送到客户端
* 客户端打开服务端 WebSocket 隧道，并对事件进行订阅，根据指定事件的推送进行响应。


PS: 欢迎关注简书 [Laravel](http://www.jianshu.com/collection/3dff40fa5135) 专题，也欢迎 Laravel 相关文章的投稿 :)，作者知识技能水平有限，如果你有更好的设计方案欢迎讨论交流，如果有错误的地方也请批评指正，在此表示感谢 :)
