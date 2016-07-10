title: 从百草园到三味书屋 —— 应用结构
date: 2016-07-10 15:11:54
tags: [php, laravel]
---

# Application Structure 应用结构

## Introduction 介绍

这个类要写到哪儿？这是一个在用框架写应用程序时十分常见的问题。大量的开发人员都有这个疑问。他们被灌输“Model”就是“Database”，在控制器里面到处 HTTP 请求，在模型里操作数据库，视图里包含了要显示的 HTML。不过，发送电子邮件的类要写到哪儿？数据验证的类要写到哪儿？调用外部 API 的类要写到哪儿？在这一章节，我们将学习如何写结构优美的 Laravel 应用，打破长久以来掣肘开发人员的普遍思维惯性这个拦路虎，最终做出好的设计。

## MVC Is Killing You 

为了做出好的程序设计，最大的拦路虎就是一个简单的缩写词：M-V-C。模型、视图、控制器主宰了 Web 框架的思想已经好多年了。这种思想的流行某种程度上是托了 Ruby on Rails 愈加流行的福。然而，如果你问一个开发人员“模型”的定义是什么。通常你会听到他嘟哝着什么“数据库”之类的东西。这么说，模型就是数据库了。不管这意味着什么，模型里包含了关于数据库的一切。但是，你很快就会知道，你的应用程序需要的不仅仅是一个简单的数据库访问类。他需要更多的逻辑如：数据验证、调用外部服务、发送电子邮件，等等更多。

> **What Is A Model？模型是啥？**
>
> 单词“model”的含义太模糊了，很难说其具体的含义。更具体来讲，模型是用来将我们的应用划分成更小、更清晰的类，使得各代码部分有着明确的权责。

所以怎么解决这个问题（译者注：上文中“更多的业务逻辑”）呢？很多开发者开始将业务逻辑包装到控制器里面。当控制器庞大到一定规模，他们将会需要重用业务逻辑。大部分开发人员没有将这些业务逻辑提取到别的类里面，而是错误的臆想他们需要在控制器里面调用别的控制器。这种模式通常被称为“HMVC”。不幸的是，这种模式通常也预示着糟糕的程序设计，并且控制器已经太复杂了。

> **HMVC（Usually）Indicates Poor Design HMVC （通常）预示着糟糕的设计**
>
> 你觉得需要在控制器里面调用其他的控制器？这通常预示着糟糕的程序设计并且你的控制器里面业务逻辑太多了。把业务逻辑抽出来放到一个新的类里面，这样你就可以在其他任何控制器里面调用了。

有一种更好的程序结构。但首先我们要忘掉以往我们被灌输的关于“模型”的一切。干脆点，让我们直接删掉 model 目录，重新开始吧！

## Bye, Bye Models 再见，模型

删掉你的 `modles` 目录了么？还没删就赶紧删了！我们将要在 `app` 目录下创建个新的目录，目录名就以我们这个应用的名字来命名，这次我们就叫 `QuickBill` 吧。在后续的讨论中，我们在前面写的那些接口和类都会出现。

> **Remember The Context 注意使用场景**
> 
> 记住，如果你在写一个很小的 Laravel 应用，那在 `models` 目录下写几个 Eloquent 模型其实挺合适的。但在本章节，我们主要关注如何开发更有合适“层次”架构的大型复杂项目。

这样我们现在有了个 `app/QuickBill` 目录，它和应用目录下的其他目录如 `controllers` 还有 `views` 都是平级的。在 `QuickBill` 目录下我们还可以创建几个其他的目录。我们来在里面创建个 `Repositories` 和 `Billing` 目录。目录都创建好以后，别忘了在 `composer.json` 文件里加入 PSR-0 的自动载入机制：

```php
"autoload": {
  "psr-o" : {
    'QuickBill': "app/"
  }
}
```

> 译者注：PSR-0 也可以改成 PSR-4，PSR-0 标准目前已经废弃。

现在我们把继承自 Eloquent 的模型类都放到 `QuickBill` 目录下面。这样我们就能很方便的以 `QuickBill\User`，`QuickBill\Payment` 的方式来使用它们。`Repositories` 目录属于 `paymentRepository` 和 `UserRepository` 这种类，里面包含了所有对数据的访问功能比如 `getRecentPayments` 和 `getRichestUser`。`Billing` 目录应当包含调用第三方支付服务（如 Stripe 和 Balanced）的类。整个目录结构应该类似这样：

```php
// app
  // QuickBill
    // Repositories
      -> UserRepository.php
      -> PaymentRepository.php
    // Billing
      -> BillerInterface.php
      -> StripeBiller.php
    // Notifications
      -> BillingNotifierInterface.php
      -> SmsBillingNotifier.php
    User.php
    Payment.php
```

> **What About Validation 数据验证怎么办？**
>
> 在哪儿进行数据验证常常困扰着开发人员。可以考虑将数据验证方法写进你的“实体”类里面（好比 `User.php` 和 `Payment.php`）。方法名可以设为 `validForCreation` 或 `hasValidDomain`。或者你也可以专门创建个验证器类 `UserValidator`，放到 `Validation` 命名空间下，然后将这个验证器类注入到你的 repository 类里面。两种方式你都可以试试，看哪个你更喜欢！

摆脱了 `models` 目录后，你通常就能克服心理障碍，实现好的设计。使你能创建一个更合适的目录结构来为你的应用服务。当然，你建立的每一个应用程序都会有一定的相似之处，因为每个复杂的应用程序都需要一个数据访问（repository）层，一些外部服务层等等。

> **Don't Fear Directories 别害怕目录**
>
> 不要惧怕建立目录来管理应用。要常常将你的应用切割成小组件，每一个组件都要有十分专注的职责。跳出“模型”的框框来思考。比如我们之前就说过，你可以创建个 `Repositories` 目录来存放你所有的数据访问类。

## It's All About The Layers 核心思想就是分层

你可能注意到，优化应用的设计结构的关键就是责任划分，或者说是创建不同的责任层次。控制器只负责接收和响应 HTTP 请求然后调用合适的业务逻辑层的类。你的业务逻辑/领域逻辑层才是你真正的程序。你的程序包含了读取数据，验证数据，执行支付，发送电子邮件，还有你程序里任何其他的功能。事实上你的领域逻辑层不需要知道任何关于“网络”的事情！网络仅仅是个访问你程序的传输机制，关于网络和 HTTP 请求的一切不应该超出路由和控制器层。做出好的设计的确很有挑战性，但好的设计也会带来可持续发展的清晰的好代码。

举个例子。与其在你的业务逻辑类里面直接获取网络请求，不如你直接把网络请求从控制器传给你的业务逻辑类。这个简单的改动将你的业务逻辑类和“网络”分离开了，并且不必担心怎么去模拟网络请求，你的业务逻辑类就可以简单的测试了：

```php
class BillingController extends BaseController {
  public function __construct(BillerInterface $biller)
  {
    $this->biller = $biller;
  }
  public function postCharge()
  {
    $this->biller->chargeAccount(Auth::user(), Input::get('amount'));
    return View::make('charege.success');
  }
}
```

现在 `chargeAccount` 方法更容易测试了。我们把 `Request` 和 `Input` 从 `BillingInterface` 里提出来，然后在控制器里把方法需要的支付金额直接传过去。

编写拥有高可维护性应用程序的关键之一，就是责任分割。要时常检查一个类是否管得太宽。你要常常问自己“这个类需不需要关心 XXX 呢？”如果答案是否定的，那么把这块逻辑抽出来放到另一个类里面，然后用依赖注入的方式进行处理。（译者注：依赖注入的不同方式还记得么？调用方法传参、构造函数传参、从 IoC 容器获取等等。）

> Single Reason To Change
>
> 如何判断一个类是否管得太宽，有一个有用的方法就是检查你为什么要改这块儿代码。举个例子：当我们想调整通知逻辑的时候，我们需要修改 `Biller` 的实现代码么？当然不需要，`Biller` 的实现仅仅需要考虑支付，它与通知逻辑应该仅通过约定来进行交互。使用这种思路过一遍代码，会让你很快找出应用中需要改进的地方。

## Where To Put "Stuff" 东西都放哪儿？

当用 Laravel 开发应用时，你可能迷惑于应该把各种“东西”都放在哪儿。比如，辅助函数要放在哪儿？事件监听器要放在哪儿？视图组件要放在哪儿？答案可能会出乎你的意料 —— “想放哪儿都行！”Laravel 并没有很多在文件系统上的约定。不过这个答案的确不能让人满意，所以下面我们就这个问题展开探索，探索这些“东西”究竟可以放在哪儿。

### Helper Functions 辅助函数

Laravel 有一个文件（`support/helpers.php`）里面都是辅助函数。你或许希望创建一个类似的文件来存储你自己的辅助函数。“start”文件是个不错的入口，该文件会在应用的每一次请求时被访问。在 `start/global.php` 里，你可以引入你自己写的 `helpers.php` 文件，就像这样：

```php
// Within app/start/global.php

require_once __DIR__ . '/../helpers.php';
// 译者注： 该 helpers.php 文件位于 app 目录下，需要你自己创建，你想放到别的地方也可以。
```

### Event Listeners 事件监听器

事件监听器当然不该放到 `routes.php` 文件里面，若直接放到“start”目录下的文件里会比较乱，所以我们要找另外的地方来存放。服务提供者是个好地方。我们之前了解到，服务提供者可不仅仅是用来做依赖注入绑定，还可以干其他事儿。可以将事件监听器用服务提供者来管理起来，让代码更整洁，不至于影响到你应用的主要逻辑代码。视图组件其实和事件差不多，也可以类似的放到服务提供者里面。

例如使用服务提供者进行事件注册可以这样：

```php
<?php
namespace QuickBill\Providers;

use Illuminate\Support\ServicePorivder;

class BillingEventsProvider extends ServiceProvider {
  public function boot () 
  {
    Event::listen('billing.failed', function ($bill) {
      // Handle failed billing event...
    })
  } 
}
```

创建好服务提供者后，就可以将它加入到 `app/config/app.php` 配置文件的 `providers` 数组里。

> **Wear The Boot 注意启动流程**
>
> 记住在上面的例子里面，我们在 `boot` 方法里进行编写是有原因的。`register` 方法只能用来进行依赖注入绑定。

### Error Handlers 错误处理

如果你的应用里面有很多自定义的错误处理方法，那你的“启动”文件可能会很臃肿。和刚才的事件监听器一样，错误处理方法也最好放到服务提供者里面。这种服务提供者可以命名为像 `QuickBillErrorProvider` 这种。然后你在 `boot` 方法里想注册多少错误处理方法都可以了。重申一下精神：让呆板的代码离你应用的业务逻辑越远越好。下方展示了这种服务提供者的一种可能的书写方法：

```php
<?php
namespace QuickBill\Providers;

use App;
use Illuminate\Support\ServiceProvider;

class QuickBillErrorProvider extends ServiceProvider {
  public function register()
  {
    //
  }

  public function boot()
  {
    App::error(function(BillingFailedException $e) {
      // Handle failed billing exceptions ...
    });
  }
}
```

> **The Small Solution 简便做法**
> 
> 当然如果你只有一两条简单的处理错误方法，那么都写在“启动”文件里面也是一种又快又好的简便做法。

### The Rest 其他

通常只要遵循 PSR-0（译者注：或 PSR-4）就可以保持类的整洁。命令式的代码比如事件监听器、错误处理器还有其他“注册”性质的操作都可以放在服务提供者里面。对于什么代码要放在什么地方这个问题，结合你目前为止学到的知识，应当可以给出一个有理有据的答案了。但永远不要害怕试验。

Laravel 最美妙之处就是你可以做出最适合你自己的风格。去探索和发现最适合你自己应用的结构吧，别忘了和他人分享你的见解！

例如你可能注意到我们上面的例子，你可以创建个 `Providers` 的命名空间来存放你自己写的服务提供者，目录就类似于这样：

```php
// app
  // QuickBill
    // Billing
    // Extensions
      //Pagination
        -> Environment.php
    // Providers
      -> EventPusherServiceProvider.php
    // Repositories
    User.php
    Payment.php
```

看上面的例子我们有 `Providers` 和 `Extensions` 两个命名空间（译者注：分别对应两个同名目录）。你自己写的服务提供者可以放到 `Providers` 命名空间下。那个 `Extensions` 命名空间可以用来存放你对框架核心进行扩展的类。

译文地址：[http://my.oschina.net/zgldh/blog/389246](http://my.oschina.net/zgldh/blog/389246)
