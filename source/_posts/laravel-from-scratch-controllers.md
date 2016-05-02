title: laravel基础教程 —— 控制器
date: 2016-05-02 08:49:43
tags: [php, laravel]
---

# HTTP 控制器

## 简介

控制器允许你将相应的路由业务逻辑封装在控制器类中进行有效的管理，这样你不必将所有的路由逻辑集中到`routes.php`文件中，导致代码的臃肿与难以维护。
所有的控制器类都被存储在`app/Http/Controllers`目录中.

## 基本控制器

一个基本的控制器应该继承自`App\Http\Controllers\Controller`控制器类:

```php
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller {
  public function showProfile($id) {
    return view('user.profile', ['user' => User::findOrFail($id)]);
  }
}
````

我们可以通过下面的方式把控制器的行为分配到路由:

```php
Route::get('user/{id}', 'UserController@showProfile');
```

一旦将控制器的行为分配到路由之后，每次客户端请求该路由，都会触发控制器的行为。这里即客户端每次请求`user/{id}`路由，`showProfile`方法都会被执行，路由中的参数也会被直接传递到该方法中.

### 控制器 & 命名空间

你应该知道我们在定义控制器路由时是不需要指定控制器的命名空间的，而只需要指定到类名就可以了，这是因为在`RouteServiceProvider`文件中自动加载的`routes.php`文件已经被指定了路由组的根命名空间`App\Http\Controllers`;

如果你想在`App\Http\Controllers`目录下使用php命名空间来嵌套或组织控制器，那么你只需要简单的指定相对于`App\Http\Controllers`部分的类名就可以了。所以如果你的控制器的全部类名为`App\Http\Controllers\Photos\AdminController`,那么你就可以这样来定义控制器路由:

```php
Route::get('foo', 'Photos\AdminController@method');
```

### 命名控制器路由

就像定义命名路由一样，我们也可以给一个控制器路由命名：

```php
Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);
```

一旦你为一个路由进行了命名， 那么你就可以通过`route`帮助方法去快速的生成被命名路由的资源表述地址:

```
$url = route('name');
```

## 控制器中间件

中间件可以这样被分配到控制器路由中:

```php
Route::get('profile', [
  'middleware' => 'auth',
  'uses' => 'UserController@showProfile'
]);
```

当然你也可以在控制器类中直接使用`middleware`方法来进行中间件的分配，你也可以只允许类中的某些行为受到指定中间件的约束:

```php
class UserController extends Controller {
  public function __construct() {
    $this->middleware('auth');

    $this->middleware('log', ['only' => [
      'fooAction',
      'barAction'
    ]]);

    $this->middleware('subscribed', ['except' => [
      'fooAction',
      'barAction'
    ]]);
  }
}
```

## RESTful 资源控制器

资源控制器可以使你快速的构建RESTful型的控制器。你可以使用artisan命令来快速的创建:

```php
php artisan make:controller PhotoController --resource
```

该命令会生成`app\Http\Controllers\PhotoController.php`文件，资源控制器中将包含每个可用的资源操作相应的方法.

你可以通过下面的方式来进行资源路由的注册:

```php
Route::resource('photo', 'PhotoController');
```

这一个简单的声明会创造多条路由用来处理RESTful式的请求.相应的通过命令生成的资源型控制器也为这些请求设置了对应的处理方法.

### 资源控制器所处理的行为

| 请求方式  | 路由地址              | 控制器行为 | 路由命名      |
| :---      | :---                  | :---       | :---          |
| GET       | `/photo`              | index      | photo.index   |
| GET       | `/photo/create`        | create     | photo.create  |
| POST      | `/photo`              | store      | photo.store   |
| GET       | `/photo/{photo}`      | show       | photo.show    |
| GET       | `/photo/{photo}/edit` | edit       | photo.edit    |
| PUT/PATCH | `/photo/{photo}`       | update     | photo.update  |
| DELETE    | `/photo/{photo}`       | destroy    | photo.destroy |


### 部分资源路由

有时候你可能并不想控制器处理全部的请求方式，那么你可以这么做：

```php
Route::resource('photo', 'PhotoController', ['only' => [
  'index', 'show'
]]);

Route::resource('photo', 'PhotoController', ['except' => [
  'create', 'store', 'update', 'destroy'
]]);
```

### 命名资源路由

默认的，所有的资源控制器行为都被进行了相应的路由命名，你可以通过`names`参数来进行重命名:

```php
Route::resource('photo', 'PhotoController', ['names' => [
  'create' => 'photo.build'
]]);
```

### 命名资源路由参数

默认的，资源路由的路由参数都被命名为相应的资源名称，你可以用过`parameters`参数来进行重命名:

```php
Route::resource('user', 'AdminUserController', ['parameters' => [
  'user' => 'admin_user'
]]);

// /user/{admin_user}
```
有时候你可能希望资源路由的路由参数并不需要像默认的资源名称一样采取复数的形式，那么你可以通过传递`parameters`的选项设置为`singular`:

```php
Route::resource('users.photos', 'PhotoController', [
  'parameters' => 'singular'
]);

// /users/{user}/photos/{photo}
```

另外，你也可以全局设置你的资源路由参数为单数形式或者全局进行资源路由参数的命名映射:

```php
Route::singularResourceParameters();

Route::resourceParameters([
  'user' => 'person',
  'photo' => 'image'
])
```
当你对资源路由参数进行定制时，你应该清楚的知道命名的顺序优先级:
1. 参数被直接的传递给`Route::resource`
2. 通过 `Router::resourceParameters` 进行全局参数映射
3. 通过`parameters`数组选项传递给`Route::resource` 或者 通过 `Route::singularResoureParameters` 进行单数形式参数设置
4. 默认行为

### 资源控制器中意外的行为

如果你必须在资源控制器中添加额外的行为去注册相应的路由，那么你一定要在使用`Route::resource`之前进行注册,否则该行为很可能会被资源控制器意外的覆盖掉.

```php
Route::get('photos/popular', 'PhotoController@method');

Route::resource('photos', 'PhotoController');
```

## 依赖注入 & 控制器

### 构造器注入

laravel的服务容器支持所有的laravel控制器的解析。由于这个原因，所以你可以在控制器的构造函数中添加你所需要依赖的相应类型提示，这些依赖会被自动的解析并注入进控制器实例.

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller {
  protected $users;

  public function __construct(UserRepository $users) {
    $this->users = $users;
  }
}
```

当然，你也被允许添加一些laravel `contract`的类型提示，只要服务容器能够正确的解析，你都可以被允许添加。

### 方法注入

除了在构造函数中进行依赖注入，你也可以在控制器的行为方法中进行依赖注入，比如，将`Illuminate\Http\Reqeust`实例注入到控制器的store方法中：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller {
  public function store (Request $request) {
    $name = $request->input('name');
  }
}
```
如果你的控制器方法也接收从路由传递过来的参数，那么他们会在其它依赖解析完毕之后被传递，比如你的路由是这么定义的:

```php
Route::put('user/{id}', 'UserController@update');
```
那么你可以这么修正你的控制器行为,来进行参数的接收:

```php
<?php

namespace App\Http\controllers;

use Illuminate\Http\Request;

class UserController extends Controller {
  public function update (Request $request, $id) {
    // 
  }
}
```

## 缓存路由

> 注意：缓存路由不支持闭包函数定义的路由，如果你想使你的路由被缓存，那么你应该使用控制器来管理你的路由.

如果你所有的路由都是基于控制器的路由，那么你应该使用laravel推荐的缓存路由，你可以简单的通过artisan命令来缓存所有路由注册到同一个文件里，它会替代`routes.php`文件被解析，使用这种缓存注册路由的方式在某些情况下注册路由的时间将被大大的减少，从而提高了应用的响应速度。
但是每次添加新的路由或者删除路由时，为了使路由生效，你需要重新生成一次缓存路由:

```php
php artisan route:cache
```

你可以通过下面的方式去删除路由缓存：

```php
php artisan route:clear
```



