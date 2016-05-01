title: laravel基础教程 —— 中间件 
date: 2016-04-30 09:36:38
tags: [php, laravel]
---

# 中间件

## 简介

HTTP 中间件为你的应用提供了一种便利的机制去过滤客户端的请求,比如说laravel中自带的用来验证用户是否已经认证的中间件，如果用户的认证没有通过，那么他将被重定向到登录视图。而如果用户已经通过认证，那么他的请求就会被认证中间件通过，并将请求传递给应用。

中间件可以处理多种任务，不仅仅只是用于验证用户认证。比如你可以创建一个跨同源策略的中间件，用来处理每个请求在被响应前添加正确的响应头，你还可以创造一个日志中间件，在应用被请求时优先记录下请求信息。

Laravel框架本身提供了一些中间件，它们包括维护、认证、csrf保护、session等中间件，这些中间件都被定义在`app\Http\Middleware`目录中。


## 定义中间件

为了创建一个新的中间件，你可以直接使用laravel提供的 `make:middleware` artisan命令:

```
php artisan make:middleware AgeMiddleware
```

这条命令会在`app\Http\Middleware`目录下创建一个`AgeMiddleware.php`文件。我们创造这么一个中间件，让只有年龄大于200的路由通过:

```php
<?php

namespace App\Http\Middleware;

use Closure;

class AgeMiddleware {
  public function handle ($request, Closure $next) {
    if ($request->get('age') > 200) {
      return $next($request);
    }
    return redirect('home'); 
  }
}
```

你可以看到，如果请求中所提供的年龄小于等于200,请求将被直接返回一个重定向信息到客户端，而如果年龄大于200，请求将被中间件继续传递给应用。为了在中间件中将请求转交给应用，你可以使用`$next`回调函数，并将`$request`传递进去。

你可以建立一系列的中间件来过滤客户端的请求，这样每一层中间件都可以检查请求，如果通过，则将请求转交到下一层，如果不通过则直接被驳回。


### 前行/后行 中间件

其实，在中间件中不仅仅可以定义前行中间件，即在请求被转交到应用之前进行处理的中间件。

```php
<?php 

namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware {
  public function handle ($request, Closure $next) {
    // Perform action

    return $next($request);
  }
}
```

也可以定义优先转交请求给应用的后执行中间件。

```php
<? php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware {
  public function handle ($request, Closure $next) {
    $response = $next($request);
    // Perform action
    return $response;
  }
}
```

## 注册中间件

### 全局中间件

如果你需要一个可以过滤所有请求的中间件，那么你可以注册一个全局中间件。你需要先定义好中间件，然后在`app/Http/kernel.php`中的`$middleware`数组属性中进行追加注册。

### 分配中间件到路由

如果你想要分配中间件到特定的路由，那么你需要在`app/Http/kernel.php`文件中`$routeMiddleware`属性中进行追加注册，在这里你应该定义一个短字符的别名，以便于你在路由分配时快速指定。

```php
// Within App\Http\Kernel Class...

protected $routeMiddleware = [
  'auth' => \App\Http\Middleware\Authenticate::class,
  'auth.basic' => \App\Http\Middleware\AuthenticateBasicAuth::class,
  'gust' => \App\Http\Middleware\RedirectIfAuthenticated::class,
  'throttle' => \App\Http\Middleware\ThrottleRequest::class,
];
```
一旦你的中间件被注册在了`kernel`文件中，那么你就可以在定义路由时使用`middleware`选项进行中间件分配：

```php
Route::get('admin/profile', ['middleware' => 'auth', function () {
  // 
}]);
```
你可以通过这么做来分配多个中间件:

```php
Route::get('/', ['middleware' => ['first', 'second'], function () {
  // 
}]);
```
当然laravel也允许你通过链式方法`middleware`去进行中间件分配:

```php
Route::get('/', function () {
 // 
})->middleware(['first', 'second']);
```
事实上，你也可以使用完全类名来进行中间件分配：
```php
use App\Http\Middleware\FooMiddleware;

Route::get('admin/profile', ['middleware' => FooMiddleware::class, function () {
  // 
}]);
```

### 中间件组

有时候你可能希望在分配路由时，可以通过一个别名来分配一系列的中间件到路由。你可以在`kernel`文件中使用`$middlewareGroups`属性来进行注册.

laravel自带了`web`和`api`中间件组:

```php
protected $middlewareGroups = [
  'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
    ],

    'api' => [
        'throttle:60,1',
        'auth:api',
    ],
];
```
一旦注册了中间件组，你可以使用相同语法去分配中间件组到路由:
```php
Route::get('/', ['middleware' => ['web'], function () {
  // 
}]);
```

事实上，laravel自带的web中间件组已经被默认启用，所有在`routes.php`中被定义的路由都被分配了此中间件。你可以在`RouteServiceProvider.php`文件中进行修改.

## 带参数的中间件

中间件也可以接收额外的自定义参数。比如说你可能需要一个中间件来验证已认证的用户的权限问题。你可能需要传递一个角色名称参数来执行相应的行为.那么你需要创建一个`RoleMiddleware`来接收一个角色名称作为额外的参数.
额外的参数将会被传递在`$next`参数之后:

```php
<?php

namesapce App\Http\Middleware;

use Closure;

class RoleMiddleware {
  public function handle ($request, Closure $next, $role) {
    if (!$request->user()->hasRole($role)) {
      // Redirect...
    }

    return $next($request);
  }
}
```

带参数的中间件在分配给路由时需要在中间件别名之后跟`:`来分割别名和参数，多个参数需要使用`,`分隔:

```php
Route::post('post/{id}', ['middleware' => 'role:editor', function ($id) {
  // 
}]);
```

## 末端中间件

有时候你可能需要在响应被发送到客户端之后继续处理一些任务，比如说 `session`中间件在laravel中就是响应被发送出去之后才将session信息进行存储操作。这时候你可以通过在中间件中添加`terminate`方法来定义一个末端中间件:

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;

class StartSession {
  public function hanlde ($request, Closure $next) {
    return $next($request);
  }

  public function terminate($request, $response) {
    // Store the sessin data...
  }
}
```

`terminate`方法会接收请求和响应，一旦你定义了一个末端中间件，你应该在`kernel`文件中将其添加到全局中间件中.

每当中间件中的`terminate`方法被调用，laravel都会从服务容器中返回一个新的中间件实例，如果你想使用同一个实例，你应该将其注册在服务容器中并使用`singleton`方法注册.








































