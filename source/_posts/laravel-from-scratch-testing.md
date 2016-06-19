title: laravel 基础教程 —— 测试
date: 2016-06-19 10:29:55
tags: [php, laravel]
---

# 测试

## 简介

测试是 Laravel 构建的核心理念。事实上，Laravel 开箱即用的支持 PHPUnit 测试，你的应用的根目录包含了 `phpunit.xml` 文件。同时，Laravel 也附带了一些方便的帮助方法可以使你丰满应用的测试。

在 `tests` 目录中提供了一个 `ExampleTest.php` 文件。在安装完成 Laravel 应用之后，你只需要在根目录运行 `phpunit` 命令就可以执行测试。

### 测试环境

当运行测试时，Laravel 会自动的设置配置环境为 `testing`。同时，Laravel 会自动的配置 session 和 缓存为 `array` 驱动。这意味着会话或者缓存数据在测试期间不会被持久化。

如果你需要，你完全可以自己创建一个测试环境。`testing` 环境变量是在 `phpunit.xml` 文件中被配置的，但是你要确保在运行测试之前先运行 `config:clear` Artisan 命令来清除配置缓存。


### 定义 & 运行测试

你可以使用 `make:test` Artisan 命令来创建一个测试用例：

```php
php artisan make:test UserTest
```

这个命令会在 `tests` 目录下创建一个新的 `UserTest` 类。你可以像使用 PHPUnit 一样接着在类中定义一些测试方法。然后你只需要在终端执行 `phpunit` 命令就可以运行测试：

```php
<?php

use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseMigrationgs;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class UserTest extends TestCase
{
  /**
   * A basic test example.
   *
   * @return void
   */
   public function testExample()
   {
     $this->assetTrue(true);
   }
}
```

> 注意：如果你在测试用例中定义了自己的 `setUp` 方法，你需要确保优先调用 `parent::setUp` 方法。

## 应用测试

Laravel 提供非常顺畅的 API 来构建 HTTP 请求检查输出甚至是填充表格。比如，让我们看下 `tests` 目录下的 `ExampleTest.php` 文件：

```php
<?php

use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ExampleTest extends TestCase
{
  /**
   * A basic functional test example
   *
   * @return void
   */
   public function testBasicExample()
   {
     $this->visit('/')
          ->see('Laravel 5')
          ->dontSee('Rails');
   }
}
```

`visit` 方法可以在应用中构造一个 `GET` 请求。`see` 方法会断言我们应该从应用的响应中看到所给定的文本。`dontSee` 方法刚好与 `see` 相反，它断言我们不应该看到给定的文本。这就是 Laravel 提供的最基本的应用测试。

### 与应用进行交互

当然，你可以比这种简单的断言响应给定的文本做的更多。让我们来看一些点击链接和填充表单的例子：

**点击超链**

在这个测试，我们会为应用构造一个请求，在响应中会返回一个可点击的超链，并且我们会断言它应该导向给定的 URL。比如，让我们假设应用中返回了一个文本为 "About Us" 的超链:

```html
<a href="/about-us">About Us</a>
```

现在，让我们编写一个测试并断言点击链接会跳转到相应的页面：

```php
public function testBasicExample()
{
  $this->visit('/')
       ->click('About Us')
       ->seePageIs('/about-us')
}
```

**与表单交互**

Laravel 同样也提供了多种方法来进行表单测试。`type`，`select`，`check`，`attach`，和 `press` 方法允许你与所有的表单输入进行交互。比如，让我们来想象一下一个存在于注册页面中的表单：

```html
<form action="/register" method="POST">
  {{ csrf_field() }}
  <div>
    Name: <input type="text" name="name">
  </div>
  <div><input type="checkbox" value="yes" name="terms">Accept Terms</input></div>

  <div><input type="submit"></div>
</form>
```

我们可以编写一个测试来完成表单并检查结果：

```php
public function testNewUserRegistration()
{
  $this->visit('/register')
       ->type('Taylor', 'name')
       ->check('terms')
       ->press('Register')
       ->seePageIs('/dashboard');
}
```

当然，如果你的表单中包含了一些单选按钮或者下拉框之类的其他输入，你同样可以轻松的进行填充这些类型的字段。下面列出了所有的表单操作方法：

方法                                       | 描述
---                                        | ---
`$this->type($text, $elementName)`         | 对给定的字段进行输入
`$this->select($value, $elementName)`      | 选中一个单选按钮或者下拉框
`$this->check($elementName)`               | 选中复选框
`$this->uncheck($elementName)`             | 取消选中复选框
`$this->attach($pathToFile, $elementName)` | 附加一个文件到表单
`$this->press($buttonTextOrElementName)`   | 单击给定的文本或名称的按钮

**与附件交互**

如果你的表单中包含了 `file` 输入类型，你可以使用 `attach` 方法来附加附件：

```php
public function testPhotoCanBeUploaded()
{
  $this->visit('/upload')
       ->type('File Name', 'name')
       ->attach($absolutePathToFile, 'photo')
       ->press('Upload')
       ->see('Upload Successful!');
}
```

## 测试 JSON APIs

Laravel 同样也对测试 JSON APIs 和它们的响应提供了多种帮助方法。比如，`get`，`post`，`put`，`patch`，和 `delete` 方法可以用来发布一个相应 HTTP 行为方式的请求。你可以轻松的传递数据和头信息到这些方法中。在开始之前，让我们来编写一个 `POST` 请求的测试来断言 `/user` 会返回给定数组的 JSON 格式：

```php
<?php

class ExampleTest extends TestCase
{
  /**
   * A basic functional test example.
   *
   * @return void
   */
   public function testBasicExample()
   {
     $this->json('POST', '/user', ['name' => 'Sally'])
          ->seeJson([
            'created' => true,
          ]) ;
   }
}
```

`seeJson` 方法会转换给定的数组为 JSON，并且会验证应用响应的完整 JSON 中是否会出现相应的片段。所以，如果响应中还含有其他 JSON 属性，那么这个测试依然会被通过。

**验证完全匹配的 JSON**

如果你希望验证完整的 JSON 响应，你可以使用 `seeJsonEquals` 方法，除非 JSON 相应于所给定的数组完全匹配，否则该测试不会被通过：

```php
<?php

class ExampleTest extends TestCase
{
  /**
   * A basic functional test example.
   *
   * @return void
   */
   public function testBasicExample()
   {
     $this->json('POST', '/user', ['name' => 'Sally'])
          ->seeJsonEquals([
            'created' => true,
          ]);
   }
}
```

**验证匹配 JSON 结构**

验证 JSON 响应是否采取给定的结构也是可以的。你可以使用 `seeJsonStructure` 方法并传递嵌套的键列表：

```php
<?php

class ExampleTest extends TestCase
{
  /**
   * A basic functional test example.
   *
   * @return void
   */
   public function testBasicExample()
   {
     $this->get('/user/1')
          ->seeJsonStructure([
            'name',
            'pet' => [
              'name', 'age'
            ]
          ]);
   }
}
```

在上面的例子中表明了期望获取一个含有 `name` 和 `pet` 属性的 JSON，并且 `pet` 键是一个含有 `name` 和 `age` 属性的对象。如果含有额外的键，`seeJsonStructure` 方法并不会失败。比如，如果 `pet` 还含有 `weight` 属性，那么测试依然会被通过。

你可以使用 `*` 来断言所返回的 JSON 结构中的每一项都应该包含所列出的这些属性：

```php
<?php

class ExampleTest extends TestCase
{
  /**
   * A basic functional test example.
   *
   * @return void
   */
   public function testBasicExample()
   {
     // Assert that each user in the list has at least an id, name and email attribute.
     $this->get('/users')
          ->seeJsonStructure([
            '*' => [
              'id', 'name', 'email'
            ]
          ]);
   }
}
```

你也可以嵌套使用 `*`，下面的例子中，我们断言 JSON 响应返回的每一个用户都应该包含所列出的属性，并且 `pet` 属性应该也包含所给定的属性：

```php
$this->get('/users')
     ->seeJsonStructure([
       '*' => [
         'id', 'name', 'email', 'pets' => [
           '*' => [
             'name', 'age'
           ]
         ]
       ]
     ]);
```

### 会话 / 认证

Laravel 为测试期间的会话提供了多种帮助方法。首先，你需要通过 `withSession` 方法来根据给定的数组设置 session 数据，这通常用来在测试应用的请求到在之前先设置一些 session 数据：

```php
<?php

class ExampleTest extends TestCase
{
  public function testApplication()
  {
    $this->withSession(['foo' => 'bar'])
         ->visit('/');
  }
}
```

当然，会话常见的用途就是保留用户的状态，比如用户认证。`actingAs` 帮助方法提供了一种方式来使用给定的用户作为已认证的当前用户。比如，我们可以使用模型工厂来生成一个认证用户：

```php
<?php

class ExampleTest extends TestCase
{
  public function testApplication()
  {
    $user = factory(App\User::class)->create();

    $this->actingAs($user)
         ->withSession(['foo' => 'bar'])
         ->visit('/')
         ->see('Hello' . $user->name);
  }
}
```

你也可以传递一个给定的守卫名称到 `actingAs` 方法的第二个参数来选择用户认证的守卫：

```php
$this->actingAs($user, 'backend')
```

### 禁用中间件

当测试应用时，你可以方便的在测试中禁中间件。这使你可以隔离的测试路由和控制器而免除中间件的顾虑。你可以简单的引入 `WithoutMiddleware` trait 来在测试类中禁用所有的中间件：

```php
<?php

use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ExampleTest extends TestCase
{
  use WithoutMiddleware;

  //
}
```

如果你希望只在个别的测试方法中禁用中间件，你可以在方法中使用 `withoutMiddleware` 方法：

```php
<?php

class ExampleTest extends TestCase
{
  /**
   * A basic functional test example.
   *
   * @return void
   */
   public funcion testBasicExample()
   {
     $this->withoutMiddleware();

     $this->visit('/')
          ->see('Laravel 5');
   }
}
```

### 定制化 HTTP 请求

如果你希望构造一个自定义的 HTTP 请求，并且返回完整的 `Illuminate\Http\Response` 对象，你可以使用 `call` 方法：

```php
public function testApplication()
{
  $response = $this->call('GET', '/');

  $this->assertEquals(200, $response->status());
}
```

如果你构造 `POST`，`PUT`，或者 `PATCH` 请求，你可以传递一个数组来作为请求的输入数据。当然，这些数据可以通过 Request 实例来在你的路由和控制器中可用：

```php
$response = $this->call('POST', '/user', ['name' => 'Taylor']);
```

### PHPUnit 断言

Laravel 对 PHPUnit 测试提供了多种额外的断言方法：

方法                                                              | 描述
---                                                               | ---
`->assertResponseOk();`                                           | 断言客户端响应会返回 OK 状态码
`->assertResponseStatus($code)`                                   | 断言客户端响应给定的状态码
`->assertViewHas($key, $value = null);`                           | 断言响应的视图中是否含有给定的数据片段。
`->assertViewHasAll(array $bindings);`                            | 断言视图中是否含有所有的绑定数据
`->assertViewMissing($key);`                                      | 断言视图中是否缺失所给定的片段
`->assertRedirectedTo($uri, $with = []);`                         | 断言客户端是否重定向到给定的 URI
`->assertRedirectedToRoute($name, $parameters = [], $with = []);` | 断言客户端是否重定向到给定的动作
`->assertSessionHas($key, $value = null);`                        | 断言会话中是否含有给定的键
`->assertSessionHasAll(array $bindings);`                         | 断言会话中是否包含给定列表的数据
`->assertSessionHasErrors($bindings = [], $format = null);`       | 断言会话中是否包含错误数据
`->assertHasOldInput();`                                          | 断言会话中是否包含旧的输入
`->assertSessionMissing($key);`                                   | 断言会话中是否缺失给定的键

## 与数据库协作

Laravel 也提供了各种有用的工具来使测试基于数据库驱动的应用更简单。首先，你可以使用 `seeInDatabase` 方法来断言给定的规范是否能在数据库中匹配到相应的数据。比如，如果你希望验证 `users` 表中是否含有 `email` 为 `sally@example.com` 的记录，那么你可以这么做：

```php
public function testDatabase()
{
  // Make call to application...

  $this->seeInDatabase('users', ['email' => 'sally@example.com']);
}
```

当然，类似 `seeInDatabase` 等方法仅仅是为了测试的方便。你可以自由的使用任意的 PHPUnit 自带的断言方法来补充你的测试。

### 在每个测试后重置数据库

我们通常要在每次测试后还原数据库，以便之前的测试数据不会对随后的测试产生干扰。

**使用迁移**

一种选择是在下个测试之前进行数据库回滚和迁移操作。Laravel 提供了简洁的 `DatabaseMigrations` trait 来自动的处理这些，你只需要在测试类引入该性状：

```php
<?php

use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ExampleTest extends TestCase
{
  use DatabaseMigrations;

  /**
   * A basic functional test example.
   *
   * @return void
   */
   public function testBasicExample()
   {
     $this->visit('/')
          ->see('Laravel 5');
   }
}
```

**使用事务**

另外一种选择就是在每个测试用例中都使用数据库的事务进行包装，这一次，Laravel 提供了方便的 `DatabaseTransactions` trait 来自动的处理这些：

```php
<?php

use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ExampleTest extends TestCase
{
  use DatabaseTransactions;

  /**
   * A basic functional test example.
   *
   * @return void
   */
   public function testBasicExample()
   {
     $this->visit('/')
          ->see('Laravel 5');
   }
}
```

> 注意： 这一性状只会包装默认数据库连接的事务。

### 模型工厂

当测试时，通常需要在执行测试之前在数据库中添加一些测试所需要的数据记录。Laravel 允许你使用 "factories" 来对你的 Eloquent 模型定义一些默认的属性设置来取代这些手动的添加数据的行为。在开始之前，我们来看一下应用的 `database/factories/ModelFactory.php` 文件。该文件中只包含了一个工厂的定义：

```php
$factory->define(App\User::class, function (Faker\Generator $faker) {
  return [
    'name' => $faker->name,
    'email' => $faker->email,
    'password' => bcrypt(str_random(10)),
    'remember_token' => str_random(10),
  ] ;
});
```

在工厂定义中的闭包中，你可以返回一些模型中的用作测试的默认值。这个闭包会接收一个 [Faker](https://github.com/fzaninotto/Faker) PHP 类库的实例，它会帮你生成一些随机数据用于测试。

当然，你可以自由的在 `ModelFactory.php` 文件中添加额外的工厂。你也可以创建额外的工厂文件来有效的组织管理。比如，你可以在 `database/factories` 目录下创建一个 `UserFactory.php` 和 一个 `CommentFactory.php` 文件。

**多个工厂类型**

有时候，你可能希望在同一个 Eloquent 模型上有多个工厂类型。比如，可能你希望除了普通用户之外还有一个管理员的工厂。你可以使用 `defineAs` 方法来定义这些工厂：

```php
$factory->defineAs(App\User::class, 'admin', function ($faker) {
  return [
    'name' => $faker->name,
    'email' => $faker->email,
    'password' => str_random(10),
    'remember_token' => str_random(10),
    'admin' => true,
  ];
});
```

你可以使用 `raw` 方法来检索基础工厂的属性，这样可用于属性的复用，然后合并添加你所需要的额外属性：

```php
$factory->defineAs(App\User::class, 'admin', function ($faker) use ($factory) {
  $user = $factory->raw(App\User::class);

  return array_merge($user, ['admin' => true]); 
});
```

**在测试中使用工厂**

一旦你定义了工厂，你就可以在你的测试文件或者数据库 seed 文件中使用全局帮助函数 `factory` 工厂方法来生成模型的实例。那么，让我们来看一些创建模型的示例。首先，我们将使用 `make` 方法，它将会创建一些模型但是却不会将其保存到数据库：

```php
public function testDatabase()
{
  $user = factory(App\User::class)->make();

  // Use model in tests...
}
```

你可以通过传递一个数组到 `make` 方法中来覆盖模型中的默认值。只有指定的部分会被替换掉，而其它部分都会被保留为工厂定义的默认值：

```php
$user = factory(App\User::class)->make([
  'name' => 'Abigail',
]);
```
你也可以根据给定的类型创建多个模型的集合：

```php
// Create three App\User instances...
$user = factory(App\User::class, 3)->make();

// Create an App\User "admin" instance...
$user = factory(App\User::class, 'admin')->make();

// Create three App\User "admin" instance...
$user = factory(App\User::class, 'admin', 3)->make();
```

**工厂模型持久化**

`create` 方法不仅仅会创建一个模型实例，它还会使用 Eloquent 的 `save` 方法将其存储到数据库中：

```php
public function testDatabase()
{
  $user = factory(App\User::class)->create();

  // Use model in tests...
}
```

这一次，你也可以传递一个数组到 `create` 方法中来覆盖默认的属性值：

```php
$user = factory(App\User::class)->create([
  'name' => 'Abigail',
]);
```

**为模型添加关系**

你甚至可以持久化多个模型到数据库中。在这个例子中，我们甚至会为创建的模型附加一个关联模型。当使用 `create` 方法来创建多个模型时，会返回一个集合实例，这允许你使用集合实例的方法，比如 `each` :

```php
$users = factory(App\User::class, 3)
           ->create()
           ->each(function ($u) {
             $u->posts()->save(factory(App\Post::class)->make());
           });
```

**关联 & 属性闭包**

你也可以在工厂定义内使用一个闭包来附加关系模型，比如，如果你希望创建一个 `Post` 时也创建一个关联的 `User` 实例，你可以这么做：

```php
$factory->fefine(App\Post::class, function($faker) {
  return [
    'title' => $faker->title,
    'content' => $faker->paragraph,
    'user_id' => function () {
      return factory(App\User::class)->create()->id;
    }
  ];
});
```

这些闭包还会接收工厂所包含的评估属性数组：

```php
$factory->define(App\Post::class, function ($faker) {
  return [
    'title' => $faker->title,
    'content' => $faker->paragraph,
    'user_id' => function () {
      return factory(App\User::class)->create()->id;
    },
    'user_type' => function (array $post) {
      return App\User::find($post['user_id'])->type;
    }
  ]; 
});
```

## 模拟

### 模拟事件

如果你在 Laravel 中大量使用了事件系统，那么你可能会想在进行测试时不触发事件或者模拟运行一些事件。比如，如果你测试用户注册，你可能并不想触发所有 `UserRegistered` 事件，因为这些可能会发送电子邮件等。

Laravel 提供了方便的 `expectsEvents` 方法来验证预期的事件是否触发，但是会避免这些事件的真正的执行：

```php
<?php

class ExampleTest extends TestCase
{
  public function testUserRegistration()
  {
    $this->expectsEvents(App\Events\UserRegistered::class);

    // Test user registration...
  }
}
```

你也可以使用 `doesntExpectEvents` 方法来验证给定的事件没有被触发：

```php
<?php

class ExampleTest extends TestCase
{
  public function testPodcastPurchase()
  {
    $this->expectsEvents(App\Events\PodcastWasPurchased::class);

    $this->doesntExpectEvents(App\Events\PaymentWasDeclined::class);

    // Test purchasing podcast...
  }
}
```

如果你需要避免所有的事件触发，你可以使用 `withoutEvents` 方法：

```php
<?php

class ExampleTest extends TestCase
{
  public function testUerRegistration()
  {
    $this->withoutEvents();

    // Test user registration code...
  }
}
```

### 模拟任务

有时候，你想在构建请求到应用时，简单的测试一下控制器中是否进行了指定任务的分发。这可以使你隔离测试你的路由 / 控制器和你的任务逻辑，当然，你可以再写一个测试用例来单独的测试任务的执行。

Laravel 提供了方便的 `expectsJobs` 方法来验证期望的任务是否被分发，但是任务并不会被真正的分发执行：

```php
<?php

class ExampleTest extends TestCase
{
  public function testPurchasePodcast()
  {
    $this->expectsJobs(App\Jobs\PurchasePodcast::class);

    // Test purchase podcast code...
  }
}
```

> 注意：这个方法仅用来测试通过 `DispatchesJobs` trait 的分发或者 `dispatch` 帮助方法的分发。它并不检测直接使用 `Queue::push` 发布的任务。

### 模拟假面

当测试时，你或许经常需要模拟一个 Laravel 假面的调用。比如，考虑一下下面的控制器操作：

```php
<?php

namespace App\Http\Controllers;

use Cache;

class UserController extends Controller
{
  /**
   * Show a list of all users of the application
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

我们可以使用 `shouldReceive` 方法来模拟调用 `Cache` 假面，它会返回一个 [Mockery](https://github.com/padraic/mockery) 的模拟实例。由于假面实际上是通过 Laravel 的服务容器来解析和管理的，所以它们比一般的静态类更具可测性。比如，让我们来模拟 `Cache` 假面的调用：

```php
<?php

class FooTest extends TestCase
{
  public function testGetIndex()
  {
    Cache::shouldReceive('get')
             ->once()
             ->with('key')
             ->andReturn('value');

    $this->visit('/users')->see('value');
  }
}
```

> 注意：你不应该模拟 `Request` 假面。你应该在运行测试时使用 HTTP 帮助方法如 `call` 和 `post` 来传递你所希望的输入。