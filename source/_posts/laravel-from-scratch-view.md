title: laravel 基础教程 —— 视图
date: 2016-05-14 10:09:03
tags: [php, laravel]
---

# 视图

视图为表现逻辑与应用逻辑的分离提供了便利，laravel 中所有的视图都被存储在 `resources/views` 目录下。

## 基础用法

一个简单的视图看起来是这样的:

```html
<!-- View stored in resources/views/greeting.php -->

<html>
  <body>
    <h1>Hello, <?php echo $name; ?></h1>
  </body>
</html>
```

我们可以使用全局的帮助函数 `view` 来返回视图 `resources/views/greeting.php` 的渲染的结果：

```php
Route::get('/', function () {
  return view('greeting', ['name' => 'James']);
});
```

你可以看到，`view` 方法接收的第一个参数是相对于目录 `resources/views` 的文件名。该方法也允许传递数组作为第二个参数，数组中的键值对会作为相应的变量传递到视图。上面的例子中就会输出 `Hello, James`

### 假如视图存在

如果你需要判断视图是否存在，那么你可以使用 `exists` 方法来进行验证，你可以通过使用无参数的全局帮助函数 `view()` 来进行链式操作，如果视图存在则会返回 `true`：

```php
if (view()->exists('email.customer')) {
  // 
}
```

当调用 `view` 方法而不传递任何参数时，会返回一个 `Illuminate\Contracts\View\Factory` 的实例，你可以访问该契约的任何方法。


## 视图数据

### 传递数据到视图

从上面的例子你就可以看出，你可以非常方便的使用一个数组作为数据来传递到视图：

```php
return view('greetings', ['name' => 'Victoria']);
```

使用这种方法传递数据，`$data` 必须应该是键值对的形式，在视图中，你可以使用相应的‘键’来访问相应的数值。当然你可以使用 `with` 方法来传递额外的数据：

```php
return view('greetings')->with('name', 'Victoria');
```

### 共享数据到所有视图

有时候，你可能需要共享一部分数据到所有的视图中，你可以使用视图工厂方法 `share` 来做到这件事情。通常情况下，你应该是在服务容器中的 `boot` 方法中去做数据共享操作。你可以在 `AppServiceProvider` 或者 独立生成一个分离的服务提供者来做这件事：

```php
namespace App\Providers;

class AppServiceProvider extends ServiceProvider
{
  /**
   * Bootstrap any application services.
   *
   * @return void
   */
   public function boot()
   {
      view()->share('key', 'value');
   }

   /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

## 视图 Composers

视图 composers （演奏者）就是当视图被渲染时会被触发的回调方法或者类的方法。如果你想要视图在每次被渲染时都自动的绑定一些数据到视图，那么视图 composer 就可以分离这些逻辑处理。

那么现在让我们在服务提供者中注册我们自己的视图 composers。我们将使用 `view` 帮助方法来返回一个 `Illuminate\Contracts\View\Factory` 契约的实现实例。laravel 中并没有提供一个默认的目录去管理这些视图 composers，你可以自主的按照自己的喜欢去管理这些，你当然也可以创建一个 `App\Http\ViewComposers` 目录：

```php
<?php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
  /**
   * Register bindings in the container.
   *
   * @return void
   */
   public function boot()
   {
      // Using class based composers...
      view()->composer(
        'profile', 'App\Http\ViewComposers\ProfileComposer'
      );

      // Using Closure based composers...
      view()->composer('dashboard', function ($view){
        //
      });
   }

   /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

如果你创建了一个服务提供者来包含你所有的视图 composers 注册，你千万不要忘记在 `config/app.php` 文件中的 `providers` 数组中进行追加注册该提供者。

现在我们有了已经注册了的 composer，方法 `ProfileComposer@compose` 将在视图 `profile` 每次被渲染时自动执行。接着，我们来定义 `ProfileComposer` 类：

```php
<?php
namespace App\Http\ViewComposers;

use Illuminate\View\View;
use App\Repositories\UserRepository;

class ProfileCompoer
{
  /**
   * The user repository implementation.
   *
   * @var UserRepository
   */
   protected $users;

   /**
    * Create a new profile composer.
    *
    * @param UserRepository $users
    * @return void
    */
    public function __construct(UserRepository $users)
    {
      // Dependencies automatically resolved by service container...
      $this->users = $users;
    }

    /**
     * Bind data to the view.
     *
     * @param View $view
     * @return void
     */
     public function compose(View $view)
     {
       $view->with('count', $this->users->count());
     }
}
```

每当视图被渲染之前，composer 的 `compose` 方法都会被调用，并且会传递一个 `Illuminate\View\View` 的实例，你可以使用 `with` 方法添加额外的数据到视图。

> 注意：所有的视图 composer 都是通过服务容器被解析的，所以你可以在 composer 的构造函数中添加类型提示信息来进行任意的依赖注入。

### 附加 composer 到多个视图

你可以一次性的绑定一个 composer 到多个视图，你只需要简单的在 `composer` 方法中的第一个参数传递一个包含视图名称的数组:

```php
view()->composer(
  ['profile', 'dashboard'],
  'App\Http\ViewComposers\MyViewComposer'
);
```

事实上 `composer` 方法也接受 `*` 字符作为通配符，允许你附加 composer 到所有的视图中:

```php
view()->composer('*', function ($view) {
  // 
});
```

### 视图创造者

视图创造者和视图 composer 类似，只不过它是在视图实例化后就立即触发注册的方法，而不是等到视图渲染时。你可以使用 `creator` 方法来注册视图创造者：

```php
view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
```