title: laravel基础教程 —— 路由
date: 2016-04-19 16:28:42
tags: [php, laravel]
---

# 路由
> 路由（routing）就是通过互联的网络把信息从源地址传输到目的地址的活动。路由发生在OSI网络参考模型中的第三层即网络层。

## 基础路由

一般所有的路由都被定义在`app/Http/routes.php`文件中，其中的内容会被框架自动加载，大多数的基础路由都可以通过简单的传递资源表述地址和`Closure`闭包函数来进行定义。

``` php
Route::get('foo', function () {
  return 'Hello world!';  
});
```

### 基础路由文件

`routes.php`路由文件是在 `app\Providers\RouteServiceProvider.php`中被加载的，在被加载的同时使用了`web`中间件组（version>=5.2), 这个中间件组被定义在`app\Http\kernel.php`的`$middlewareGroups`变量中，该中间件组提供了cookie加密、cookie响应、session启用、自动注入error session 到视图、csrf保护的功能。你的应用程序的大多数路由都应该定义在这个路由文件里。

### 可用的路由方法

路由允许你在注册路由时使用任何http请求方法：

``` php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

有时候你可能需要在注册路由时允许多种请求方式，这个时候你可能需要使用`method`方法，当然你可以使用`any`方法允许任何请求方式:

``` php
Route::match(['get', 'post'], function () {
  // 
});

Route::any('foo', function () {
  //
})
```

## 带参数的路由

### 必要参数的路由

有时候你可能需要捕获路由中的某个片段，比如说用户的id, 那么你就可以定义带参数的路由：

``` php
Route::get('user/{id}', function ($id) {
  return 'user_id is: ' . $id;
});
```

 可能你需要获取到路由中的多个参数, 那么你可以这么做: 

 ```php
 Route::get('posts/{post}/comments/{comment}', function ($post, $comment) {
  //
 });
 ```
 路由参数就是定义在`{}`里的，每当该路由被访问到，相关注册的参数就会被按序的传递进闭包函数里.

 `ps: 路由参数不能含 **-** ，应该使用 **_** 代替，比如 {user_id}`


### 可选的参数的路由

有时候你可能需要定义一个带参数的路由，但是这个参数可有可无，没有的话，可以给予一个默认值，那么这时候就需要用 `{name?}`的方式来定义可选路由了：

``` php
Route::get('user/{name?}', function ($name = null) {
  return $name; 
});

Route::get('user/{name?}', function ($name = 'john'){
 return $name; 
});
```

### 用正则表达式约束参数

你可以使用正则表达式去约束你的参数，这里你可以使用`where`方法，当参数与约束规则不匹配时就不会匹配到该路由：

```php
Route::get('user/{name}', function ($name){
  //
})
->where('name', '[a-zA-Z]+');
Route::get('user/{id}', function ($id){
 // 
})
->where('id', '[0-9]+');
Route::get('user/{id}/{name}', function ($id, $name){
  //
})
->where(['id' => '[0-9]+', 'name' => '[a-zA-Z]+');
```

#### 全局的约束表达式

你可以定义全局的约束表达式，这样所有匹配到的路由参数都会受到相同的约束条件, 这个时候你需要在路由启动之前就有注册好约束，你需要在路由服务提供者文件里在启动路由前增加约束。文件是`app\Providers\RouteServiceProvider.php`

```php
public function boot(Router $router) {
  $router->pattern('id', '[0-9]+');

  parent::boot($router);
}
```
当然，在5.2版本里，路由服务提供者在引入`router.php`文件时已经注入了`$router`变量，所以你也可以在`router.php`文件里这么提供批量约束表达式 :

```php
$router->pattern('id','[0-9]+');

Route::get('user/{id}', function ($id) {
 // 
});
```

但是要注意注册的前后顺序，只有在注册之后引入的路由才受约束，而在 `$router-pattern('id', '[0-9]+')` 之前注册的路由是不受约束的.

## 命名路由

命名路由可以方便的生成资源表述地址，可以方便的将路由重定向到指定的路由地址。你可以在注册路由时，传递第二个参数为数组参数，并添加`as`索引：
``` php
Route::get('user/profile', ['as' => 'profile', function () {
  //
}]);
```
当然你也可以命名使用控制器行为的路由：
``` php
Route::get('user/profile', ['as' => 'profile', 'use' => 'UserController@showProfile']);
```

有时候数组写起来比较麻烦，我们还可以使用`name`方法去命名路由，当然它是支持链式调用的:
```php
Route::get('user/profile', 'UserController@showProfile')->name('profile');
```
### 路由组 & 命名组路由

如果你使用了路由组，你可能想在整个组中增加一个组命名前缀，当然，laravel是支持这么做的：
```php
Route::group(['as' => 'admin::'], function () {
  Route::get('dashboard', ['as' => 'dashboard', function () {
    // route named 'admin::dashboard'
    // you can redirect to this use route('admin::dashboard')
  }]) ;
});
```
### 使用命名路由生成urls

一旦你命名了一个路由，那么你就可以使用全局帮助函数`route`来生成路由url地址，也可以用来做为重定向时生成重定向地址:
```php
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```

如果你是对一个参数路由进行命名，那么你可以在使用`route`方法获取路由url时传递第二个参数，该参数是一个数组，你所传递的参数会正确的按索引匹配到相应的位置,如果参数索引与路由参数不一致，则会优先匹配相同的参数存放到相应位置，不一致的会按序存入：
``` php
Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
 // 
}]);
$url = route('profile', ['id' => 1]);
```

## 路由组

路由组的使用方便共享路由的一些能力，比如共享中间件，或者命名空间，这样就不必每一个路由都要单独去定义一次相同的中间件或命名空间。路由组使用 `Route::group`去定义，并共享第一个参数中的能力.该参数为数组.

### 共享中间件

你可以使用`middleware`索引来建立共享中间件,这样路由组内定义的路由都会在匹配时引入该中间件:

``` php
Route::group(['middleware' => 'auth'], function () {
  Route::get('/', function () {
    // Uses Auth Middleware
  });

  Route::get('user/profile', function () {
    // Uses Auth Middleware
  });
});
```

### 共享命名空间

路由组另外一个常用的例子就是在控制器中共享命名空间，这个时候就要用到了`namespace`索引:

```php
Route::group(['namespace' => 'Admin'], function () {
   // Controllers Within The 'App\Http\Controllers\Admin' Namespace 

   Route::group(['namespace' => 'User'], function () {
     //  Controllers Within The 'App\Http\Controllers\Admin\User' Namespace
   });
});
```

因为我们大部分路由都是定义在`routes.php`文件里，而路由服务在提供服务时将该文件引入到路由组中,并共享了`App\Http\Controllers`命名空间，这样，在该文件中定义的控制器路由，可以不使用`App\Http\Controllers`前缀.

```php
// Already Within The 'App\Http\Controllers' Namespace
Route::get('user/profile', 'UserController@showProfile');
```

### 子域名路由

子域名路由可以做域名匹配路由，也就是说可以约束域名做匹配，如果不是匹配到的域名则不会注册该组内路由， 当然你可以对子域名进行类参数路由的约束，这样允许你捕获域名的一部分或全部:

```php
Route::group(['domain' => '{account}.myapp.com'], function () {
  Route::get('user/{id}', function ($account, $id) {
    //
  });
});
```
### 路由前缀

使用`prefix`索引可以在路由组中定义路由前缀，这样在该组内的路由都会自动使用该前缀：

``` php
Route::group(['prefix' => 'admin'], function () {
  Route::get('dashboard', function () {
    // Matches The 'admin/dashboard' URL
  });
});
```
当然你可以在路由前缀中使用参数：

``` php
Route::group(['prefix' => 'accounts/{account_id}'], function () {
  Route::get('detail', function ($account_id) {
    // Matchs The '/accounts/{account_id}/detail' URL
  });
});
```
## CSRF 保护

> 跨站请求伪造（英语：Cross-site request forgery），也被称为 one-click attack 或者 session riding，通常缩写为 CSRF 或者 XSRF， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。跟跨网站脚本（XSS）相比，XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。
> 跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。这利用了web中用户身份验证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

laravel框架自带csrf攻击保护措施，它被以中间件的形式引入到`web`路由组中，所以每个在`routes.php`文件中定义的路由被访问时都会经过csrf中间件的验证。你可以查看`vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/VerifyCsrfToken.php`文件，阅读其实现。
`csrf(跨站请求伪造)`是一种恶意的攻击行为，它可以绕过用户的授权去执行未授权的行为。

laravel会为每个活跃用户生成一个csrf token,而这个token就是用来验证请求是不是真实用户授权的。所以你应该在每个表单中都使用隐藏的input并录入csrf token,这样提交表单的请求才能通过csrf中间件的验证.
laravel提供了方便的形式去在表单中生成csrf token:

``` php
// Vanilla PHP
<?php echo csrf_field(); ?>

// Blade Template Syntax
{{ csrf_field() }}
```
`csrf_field()`方法会生成以下html节点：

``` php
<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">
```

你并不需要去手动的验证这些能够修改状态的http请求，比如 post, put, delete, 因为VerifyCsrfToken中间件已经自动处理这些请求了，它会自动的根据请求中的token与会话中存储的token进行匹配验证。


### 排除csrf保护

有时候我们可能需要排除个别url，让他不受csrf的保护，因为可能我们需要挂接一些其他的第三方系统，这个时候系统之间的通讯是不应该使用csrf机制去验证的，比如说，你使用了支付宝的即时付款机制，那么在用户付款的时候服务器与服务器之间会有一个回馈机制，用于支付宝通知我们的应用用户已经成功付款，这个时候我们可能需要排除csrf对反馈路由的保护。那么我们可以在`VerifyCsrfToken`中间件中修改`$except`属性中增加一条规则:

``` php
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

class VerifyCsrfToken extends BaseVerifier
{
   /** 
    * The URIs that should be excluded from CSRF verification.
    * 
    * @var array
    */
   protected $except = [
    'alipay/*',
   ];
}
```

### X-CSRF-TOKEN
laravel的`VerifyCsrfToken`中间件不仅允许通过表单中验证_token参数，它也允许通过请求头去进行csrf验证，它会验证请求头中的`X-CSRF-TOKEN`值是否与会话的token值匹配，所以你可以这么做，存储token到`meta`中：
``` html
<meta name="csrf-token" content="{{ csrf_token() }}" >
```
一旦你定义了`meta`标签，你可以使用像`jquery`类似的框架去添加token到所有请求的头部:

``` javascript
  $.ajaxSetup({
    headers: {
      'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
  })
```


### X-XSRF-TOKEN

laravel自动存储了CSRF token 到 cookie中并被命名为 `XSRF-TOKEN`, 你可以在每次请求的请求头里引入`X-XSRF-TOKEN`并将其值设置为 `XSRF-TOKEN`的cookie值来通过csrf验证。有一些javascript框架里已经自动做了这些，像 angularjs. 如果你没有引入这些前端框架，你可能需要自己手动去做了:

``` javascript
  fetch('/user/password', {
    method: 'POST',
    headers: {
      'X-XSRF-TOKEN': 
    }
  })
```
## 绑定模型的路由

绑定模型的路由提供了一种便利的方式去注入模型实例到路由中，比如，你可以通过传递id，来注入匹配id的用户的整个模型到路由中。

### 隐式绑定

laravel会自动的解析定义在路由或者控制器中的`Eloquent`模型, 根据变量名与参数名匹配的，比如`api/users/{user}` 匹配 `$user`变量：
``` php
Route::get('api/users/{user}', function (App\User $user) {
  return $user->email; 
});
```
在上面的例子中，$user是根据url中`{user}`片段的值进行实例生成的.这种绑定模型到路由一般都是默认根据id进行匹配注入实例。比如说 `users/1`，匹配到的就是id = 1的用户实例。如果没有匹配的实例，则会返回404。

### 自定义索引绑定模型到路由

如果你不想用id去绑定模型到路由，laravel提供了一种可自定义的方式，你只需要在相应的模型里重写`getRouteKeyName`方法就行了，比如我们用名字来代替id注入模型到路由:

``` php
public function getRouteKeyName() {
  return 'name';
}
```
这样在路由url中就是类似这种形式了 `users/wang`

### 显式绑定

如果需要注册一种显式的模型绑定到路由，你需要使路由的`model`方法去指定类与参数的映射。并且你应该在路由服务的`boot`方法中进行绑定注册操作：

### 绑定到模型

``` php
public function boot(Router $router) {
  parent::boot($router);

  $router->model('user', 'App\User');
}
```
这样，在`routes.php`文件中注册的路由如果使用了`{user}`的命名路由就可以自动绑定到模型:
``` php
$router->get('profile/{user}', function (App\User $user) {
 // 
});
```
### 自定义绑定解析逻辑

如果你想自定义解析返回实例的逻辑，那么你需要使用`Route::bind`方法，该方法第一个参数为绑定url的参数名，第二个参数为`Closure`, 闭包中会接收一个参数，这个参数是url中对应参数片段的值，你应该在闭包中返回解析后的实例：

``` php
$router->bind('user', function ($value) {
 return App\User::where('name', $value)->first(); 
});
```

### 自定义找不到实例时的行为

如果你想要自定义无法匹配实例时的行为，你可以向`model`方法传递第三个参数，该参数是一个闭包函数，它将在无法匹配到实例时执行：

``` php
$router->model('user', 'App\User', function () {
  throw new NotFoundHttpException;
});
```

## 表单提交方式模拟
由于HTML的表单不能支持像`PUT`,`PATCH`或者`DELETE`的请求方式，所以，当需要访问类似的路由时，你需要在表单中增加一个隐藏的字段`_method`，用来表明你真实的表单请求方式:

``` html
<form action=“/foo/bar" method="POST">
  <input type="hidden" name="_method" value="PUT">
  <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

laravel也提供了便捷生成隐藏字段`_method`的方法，你可以使用`method_field`帮助方法：
``` php
<?php echo method_field(); ?>
```
使用blade模板引擎可以直接这么用：
``` php
{{ method_field('PUT') }}
```

## 访问当前路由

`Route::current()` 会返回一个`Illuminate\Routing\Route`的实例，它允许你在当前路由内处理相关请求：

``` php
$route = Route::current();

$name = $route->getName();

$actionName = $route->getActionName();
```

当然你也可以通过`Route`facade去访问路由的name或action:
``` php
$name = Route::currentRouteName();

$action = Route::currentRouteAction();
```

