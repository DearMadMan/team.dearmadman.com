title: 从百草堂到三味书屋 —— 依赖注入
date: 2016-07-08 15:38:47
tags: [php, laravel]
---

# Dependency Injection 依赖注入

Laravel 框架的基础是一个功能强大的控制反转容器（IoC container）。为了真正理解本框架，需要好好掌握该容器。然而我们需要了解，控制反转容器只是一种用于方便实现“依赖注入”的工具。但要实现依赖注入并不一定需要控制反转容器，只是用容器会更方便和容易一点。

首先来看看我们为何要使用依赖注入，它能带来什么好处。考虑下列代码：

```php
class UserController extends BaseController {
  public function getIndex() {
    $user = User::all();
    return View::make('users.index', compact('users'));
  }
}
```

这段代码很简短，但我们要想测试这段代码的话就一定会和实际的数据库发生联系。也就是说，Eloquent ORM（译者注：Laravel 的数据库对象模型库）和该控制器有着紧耦合。如果不使用 Eloquent ORM，不连接到实际数据库，我们就没有办法运行或者测试这段代码。这段代码同时也违背了“关注分离”这个软件设计原则。简单讲：这个控制器知道的太多了。控制器不需要去了解数据是从哪儿来的，只要知道如何访问就行。控制器也不需要知道这数据是从 MySQL 或哪儿来的，只需要知道这数据目前是可用的。

> Separation Of Concerns 关注分离
> 每一个类都应该有单独的职责，并且该职责应完全被这个类封装。（译者注：我认为就是不要让多个类负责同样的职责）

关注分离的好处就是能让 Web 控制器和数据访问解耦。这会使得实现存储迁移更容易，测试也会更容易。“Web”就仅仅是为你真正的应用做数据的传输了。

想象一下你有一个类似于监视器的程序，有着很多的线缆接口（HDMI，VGA，DVI 等等）。你可以通过不同的接口访问不同的监视器。把 Internet 想象成另一个插进你进程线缆接口。大部分显示器的功能是与线缆接口互相独立的。线缆接口只是一种传输机制就像 HTTP 是你程序的一种传输机制一样。所以我们不想把传输机制（控制器）和业务逻辑混在一起。这样的好处是很多其他的传输机制比如 API 调用、移动应用等都可以访问我们的业务逻辑。

那么我们就别再将控制器和 Eloquent ORM 耦合在一起了。咱们注入一个资料类库。

## Build A Contract 建立约定

首先我们定义一个接口，然后实现该接口。

```php
interface UserRepositoryInterface
{
  public function all();
}

class DbUserRepository implements UserRepositoryInterface
{
  public function all()
  {
    return User::all()->toArray();
  }
}
```

然后我们将该接口的实现注入我们的控制器。

```php
class UserController extends BaseController
{
  public function __construct(UserRepositoryInterface $users)
  {
    $this->users = $users;
  }

  public function getIndex()
  {
    $user = $this->users->all();
    return View::make('users.index', compact('users'));
  }
}
```

现在我们的控制器就完全和数据层面无关了。在这里无知是福！我们的数据可能来自 MySQL，MongoDB 或者 Redis。我们的控制器不知道也不需要知道他们的区别。仅仅做出了这么小小的改变，我们就可以独立于数据层来测试 Web 层了，将来切换存储实现也会很容易。

> Respect Boundaries 严守边界
> 记得要保持清醒的责任边界。控制器和路由是作为 HTTP 和你的应用程序之间的中间件来用的。当编写大型应用程序时，不要将你的领域逻辑混杂在其中（控制器、路由）。

为了巩固学到的知识，咱们来写一个测试案例。首先，我们要模拟一个资料库然后绑定到应用的 IoC 容器里。然后，我们要保证控制器正确的调用了这个资料库：

```php
public function testIndexActionBindsUsersFromRepository()
{
  // Arrange...
  $repository = Mockery::mock('UserRepositoryInterface');
  $repository->shouldReceive('all')->once()->andReturn(array('foo'));
  App::instance('UserRepositoryInterface', $repository);
  // Act...
  $response = $this->action('GET', 'UserController@getIndex');

  // Assert...
  $this->assertResponseOk();
  $this->assertViewHas('users', array('foo'));
}
```

> Are you Mocking Me 你在模仿我么？
> 在上面的例子里，我们使用了名为 `Mockery` 的模仿库。这个库提供了一套整洁且富有表达力的方法，用来模仿你写的类。Mockery 可以通过 Composer 安装。

## Taking It Further 更进一步

让我们考虑另一个例子来巩固理解。可能我们想要去提醒用户该交钱了。我们会定义两个接口，或者约定。这些约定使我们在更改实际实现时更加灵活。

```php
interface BillerInterface {
  public function bill(array $user, $amount);
}

interface BillingNotifierInterface {
  public function notify(array $user, $amount);
}
```

接下来我们要写一个 `BillerInterface` 的实现：

```php
class StripeBiller implements BillerInterface {
  public function __construct(BillingNotifierInterface $notifier)
  {
    $this->notifier = $notifier;
  }

  public function bill(array $user, $amount)
  {
    // Bill the user via Stripe...
    $this->notifier->notify($user, $amount);
  }
}
```

只要遵守了每个类的职责划分，我们很容易将不同的提示器（notifier）注入到账单类里面。比如，我们可以注入一个 `SmsNotifier` 或者 `EmailNotifier`。账单类只要遵守了约定，就不用再考虑如何实现提示功能。只要是遵守约定（接口）的类，账单类都能用。这不仅仅是方便了我们的开发，而且我们还可以通过模拟 `BillingNotifierInterface` 来进行无痛测试。

> Be The Interface 使用接口
> 写接口可能看上去挺麻烦，但实际上能加速你的开发。你不用实现任何接口，就能使用模拟库来模拟你的接口，进而测试整个后台逻辑！

那我们如何做依赖注入呢？很简单：

```php
$biller = new StripeBiller(new SmsNotifier);
```

这就是依赖注入。biller 不需再考虑提醒用户的事儿，我们直接传给他一个提示器（notifier）。这种微小的改动能使你的应用焕然一新。你的代码马上就变得更容易维护，因为明确指定了类的职责边界。并且更容易测试，你只需使用模拟依赖即可。

那 IoC 容器呢？难道依赖注入不需要 IoC 容器么？当然不需要！在接下来的章节里面你会了解到，容器使得依赖注入更易于管理，但是容器不是依赖注入所必须的。只要遵循本章提出的原则，你可以在你任何的项目里面实施依赖注入，而不必管该项目是否使用了容器。

## Too Much Java？太像 Java 了？

有人会说使用接口让 PHP 代码看上去太像 Java 了 —— 即代码太啰嗦了 —— 你必须定义接口然后实现它，要多按好多下键盘。

对于小而简单的应用来说，以上说法也对。接口通常是不必要的。将代码耦合到那些你认为不会改变的地方也是可以的。在你确信不会改变的地方就没有必要使用接口了。架构师说“不会改变的地方是不存在的”。不过话说回来，有时候的确不会改。

在大型应用中接口是很有帮助的。和提升的代码灵活性、可测试性比起来，多敲键盘费的功夫就微不足道了。当你迅速的切换了代码实现的时候，你的经理一定会被你的神速吓一跳的。你也可以写出更适应变化的代码。

总而言之，记住本书提倡“简单”架构。如果你在写小程序的时候无法遵守接口原则，别觉得不好意思。要记住我们写代码是要快乐的写。如果你不喜欢写接口，那就先简单的写代码吧。日后再精进即可。


译文地址：[http://my.oschina.net/zgldh/blog/389246](http://my.oschina.net/zgldh/blog/389246)