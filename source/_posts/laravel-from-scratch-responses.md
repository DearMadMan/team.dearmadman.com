title: laravel基础教程 —— 响应
date: 2016-05-03 07:11:03
tags: [php, laravel]
---

# HTTP 响应

## 基础响应

所有的路由和控制器都应该返回某种响应发送回给用户的浏览器，laravel提供了多种不同的方式来返回响应。最基本的响应就是简单的在路由或控制器中返回字符串:

```php
Route::get('/', function () {
  return 'Hello World';
});
```

这里返回的字符串会被laravel自动的转换为HTTP响应发送出去.

### 响应对象

大多数时候，路由或者控制器的行为都应该返回`Illuminate\Http\Response`实例或者`view`。`Response`实例允许你便利的对响应头和响应的状态进行定制化.一个`Response`实例继承自`Symfony\Component\HttpFoundation\Response`类，该类提供了多种方法来生成一个HTTP响应:

```php
use Illuminate\Http\Response;

Route::get('home', function () {
  return (new Response($content, $status))
                ->header('Content-Type', $value); 
});
```

laravel也提供了便利的`response`全局帮助函数:

```php
Route::get('home', function () {
  return response($content, $status) 
           ->header('Content-Type', $value);
});
```

> 所有可以使用的`Response`实例的方法，你可以查看 [API 文档](http://laravel.com/api/master/Illuminate/Http/Response.html) 和 [Symfony API文档](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Response.html).

###  附加响应头

`Response`实例的大多数方法都是允许进行链式调用的，你可以使用链式调用来流利的构建响应。例如，你可以使用`header`方法去添加多种响应头在其被发送到用户客户端之前:

```php
return response($content)
         ->header('Content-Type', $type)
         ->header('X-Header-One', 'Header Value')
         ->header('X-Header-Two', 'Header Value');
```

或者你可以使用`withHeaders`方法来通过指定的数组添加请求头到响应:

```php
return response($content)
         ->withHeaders([
           'Content-Type' => $type,
           'X-Header-One' => 'Header Value',
           'X-Header-Two' => 'Header value'
         ]);
```

### 添加cookie到响应

`Response`实例的`cookie`可以简单的追加cookie信息到响应。例如，你可以使用`cookie`方法来生成cookie并附加到响应中:

```php
return response($content)
         ->header('Content-Type', $type)
         ->cookie('name', 'value');
```

`cookie`方法允许你添加额外的参数来进一步定制化你的cookie属性:

```php
->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```

默认的,laravel中所有的cookie都是经过签证加密的，所以客户端的用户是没办法修改或者解读的.所以如果你想在生成某些cookie时，禁用默认的加密，你需要在`App\Http\Middleware\EncreyptCookies`中间件的`$except`属性中进行追加:

```php
protected $except = [
  'cookie_name'
];
```

## 其它响应类型

帮助方法`response`也可以用来便利的生成其它类型的响应实例，如果你使用`response`帮助方法不传递任何参数，那么它会返回一个实现了`Illuminate\Contracts\Routing\ResponseFactory` contract的实例。该契约提供了多种有用的方法来生成响应.

### 视图响应

如果你想控制响应的响应头与状态，并且你也需要返回一个视图作为响应的内容，那么你可以使用`view`方法：

```php
return response()
         ->view('hello', $data)
         ->header('Content-Type', $type);
```


当然，如果你并不需要定制化响应的状态或者响应头，那么你可以直接使用全局帮助方法`view`。


### JSON响应

`json`方法会自动的设置响应头的`Content-Type`为`application/json`,以及使用`json_encode`方法将提供的数组转换为JSON:

```php
return response()->json(['name' => 'Abigail', 'state' => 'CA']);
```

如果你想要生成`jsonp`的响应，那么你可以使用`json`方法然后附加`setCallback`方法:

```php
return response()
         ->json(['name' => 'Abigail', 'state' => 'CA'])
         ->setCallback($request->input('callback'));
```

### 文件下载

`download`方法可以在返回一个强制用户端浏览器进行下载指定路径文件的响应。`download`方法也允许传递第二个参数作为浏览器下载时的文件名,你也可以传递http响应头数组作为第三个参数:

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> 注意：Symfony HttpFoundation需要被下载的文件有一个ASCII文件名

### 文件响应

`file`方法允许你在浏览器直接显示image或pdf类型的文件来代替直接下载。该方法接收文件路径作为第一个参数，也可以接收HTTP响应头数组作为第二个参数:

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```


## 重定向

重定向响应是一个`Illuminate\Http\RedirectResponse`类的实例，它包含了所有重定向到指定URI所需要的响应头信息。laravel提供了多种方式去生成重定向实例.最简单的方式莫过于使用全局帮助方法`redirect`:

```php
Route::get('dashboard', function () {
  return redirect('home/dashboard'); 
});
```

有时候你需要将用户重定向到上一次请求的地址，那么你可以使用全局帮助方法`back`。因为这里用到了session，所以你必须要保证你的路由使用了session中间件，默认的laravel的路由都被包裹在`web`路由组中，`web`中间件组中已经包含了session中间件:

```php
Route::post('user/profile', function () {
  // Validate the request...

  return  back()->withInput();
});
```

###  重定向至命名路由

当你使用`redirect`方法而不传递任何参数时，laravel将返回`Illuminated\Routing\Redirector`的一个实例，该实例允许你使用一些方法来处理重定向信息,例如，为了生成重定向到命名路由，你可以使用`route`方法：

```php
return redirect()->route('login');
```

如果命名路由也含有参数，那么你可以传递第二个参数到`route`方法:

```php
return redirect()->route('profile', ['id' => 1]);
```

如果你需要重定向的路由是使用`Eloquent`模型的`id`作为参数识别的路由,那么你可以直接在`route`方法中传递用户实例，它会在动被解析到id：

```php
return redirect()->route('profile', [$user]);
```


### 重定向至控制器行为

你也可以生成重定向信息到控制器的某个行为.你可以简单的通过`action`方法传递控制器名称和控制器行为来做到这些.你应该注意，你并不需要特别的指出控制器的全部命名空间,因为laravel的`RouteServiceProvider`已经自动的设置了默认的命名空间:

```php
return redirect()->action('HomeController@index');
```

当然，如果你的控制器行为也接收其他的参数，你同样可以在`action`方法中传递第二个参数:

```php
return redirect()->action('UserController@profile', ['id' => 1]);
```

### 重定向并闪存session

你可以通过`RedirectResponse`实例的链式调用来做到在生成重定向信息的同时闪存seession数据，这在执行行为之后存储消息状态的场景尤其有用:

```php
Route::post('user/profile', function () {
  // Update the user's profile...

  return redirect('dashboard')->with('status', 'profile updated'); 
});
```
当然，在用户重定向到新页面时，你是可以访问到闪存的会话信息的，例如，在blade模板中，你可以这么使用:

```php
@if (session('status')) 
  <div class="alert alert-success">
    {{ session('status')}}
  </div>
@endif
```

## 响应宏

如果你想自定义某种可复用的响应，你可以使用`Response` facade的 `macro`方法或者提供`Illuminate\Contracts\Routing\ResponseFactory`的一个实现:

```php
<?php

namespace App\Providers;

use Response;
use Illuminate\Support\ServiceProvider;

class ResponseMacroServiceProvider extends ServiceProvider {
  public function boot() {
    Response::macro('caps', function ($value) {
      return Response::make(strtoupper($value));
    }); 
  }
}
```

`macro`方法接收一个别名作为第一个参数，接收一个闭包作为第二个参数。闭包会在访问`ResponseFactory`实现的实例的动态属性`macro`别名时执行，或者通过全局帮助方法`response`:

```php
return response()->caps('foo');
```