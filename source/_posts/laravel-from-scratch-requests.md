title: laravel 基础教程 —— 请求
date: 2016-05-02 13:42:16
tags: [php, laravel]
---

# HTTP请求 

## 访问请求

为了通过依赖注入能够方便的获取http请求实例，你应该在控制器的构造函数或者控制函数中写入类型声明`Illuminate\Http\Request`。当前请求的实例会自动的从服务容器中注入:

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


如果你的控制器方法也需要接收来自路由的参数，那么你需要在进行依赖注入的参数之后添加要接收的参数。例如，你的路由是这么定义的：

```php
Route::put('user/{id}', 'UserController@update');
```

那么你应该先声明依赖注入`Illuminate\Http\Reqeust`,然后再按序的传递路由参数:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller {
  public function update (Request $request, $id) {
    // 
  }
}
```

### 基础的请求信息

`Illuminate\Http\Request`的实例为你的应用提供了多种检查HTTP请求的方法，它继承自`Symfony\Component\HttpFoundation\Request`。这里列举了一些常用的方法:


#### 检索请求的URI

`path`方法返回请求的URI.所以，当请求的目标地址是`http://domain.com/foo/bar`时，`path`方法将返回`foo/bar`:

```php
$uri = $request->path();
```

`is`方法允许你来效验所请求的URI是否匹配所提供的模式。你可以使用 `*`字符来作为通配符:

```php
if ($request->is('admin/*')) {
  //
}
```

如果你想得到请求的完整路径，那么你可以使用 `url` 或 `fullUrl` 方法：

```php
// Without Query String...

$url = $request->url();

// With Query String...

$url = $request->fullUrl();
```

你也可以在获取完整请求路径的同时追加请求的参数信息，例如，如果请求的目标是 `http://domain.com/foo`, 下面的方法将返回`http://domain.com/foo?bar=baz`:

```php
$url = $request->fullUrlWithQuery(['bar' => 'baz']);
```

#### 检索请求的方式

`method`方法将返回HTTP请求的方式。你也可以使用`isMethod`方法来进行http请求方式的匹配效验:

```php
$method = $request->method();

if ($request->isMethod('post')) {
  //
}
```


### PSR-7 请求

[PSR-7](http://www.php-fig.org/psr/psr-7/)标准为http指定了一些消息接口，包括请求和响应.如果你想获得PSR-7请求的实例，你需要先安装一些支持库，laravel使用了 Symfon HTTP Bridge 组件来转换典型的laravel请求和响应为PSR-7兼容的实现:

```php
composer require symfony/psr-http-message-bridge

composer require zendframework/zend-diactoros
```

一旦你安装了这些库，你就可以简单在你的路由或控制器中使用类型提示来获取PSR-7请求:

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
  // 
});
```

如果路由或控制器返回的是PSR-7响应的实例， 那么它会自动的转换为laravel响应实例。

## 获取输入值

### 获取输入的值

你可以通过`Illuminate\Http\Request`实例的一些方法来简便的获取用户的输入值.而且你并不需要去关心用户所使用的HTTP请求方式，你可以通过`input`方法获取所有请求方式的值:

```php
$name = $request->input('name');
```

你也可以传递第二个参数到`input`方法，如果该值并不存在于请求中，将作为默认值返回:

```php
$name = $request->input('name', 'Sally');
```

当表单提交的一些输入值是数组时,你可以使用`.`操作符来访问请求中的数组值:

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');

```


### 获取JSON请求的输入值

你同样可以使用input方法来访问json请求，只要json请求被设置了正确的请求头 `Content-Type: application/json`， 那么你就可以使用 `.` 语法来深入访问json中的数组:

```php
$name = $request->input('user.name');
```

### 判断输入值是否存在

你可以使用`has`方法来判断，请求中是否包含了用户的某个输入值,如果该值不是空的字符串，那么`has`方法就会返回`true`:

```php
if ($request->has('name')) {
  //
}
```

### 获取所有的输入值

你可以使用`all`方法来获取所有的用户输入值, 该方法返回包含所有用户值的数组:

```php
$input = $request->all();
```


### 获取部分输入值

你可以使用`only`或者`except`方法来获取部分输入值, 这两个方法都接收一个单独的数组或者动态的参数列表:

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

### 动态属性

你可以通过`Illuminat\Http\Request`的实例的动态属性来获取用户的输入值.如果你的应用表单中存在`name`字段，你可以通过下面的方式来获取该请求字段:

```php
$name = $request->name;
```

当使用动态属性时，laravel会首先查找请求中是否包含该值，然后才会检索路由中的参数.


## 旧的输入

laravel允许你在下一次请求期间保持该次请求的输入。这种特性在表单验证出错时尤其有用，它可以使你复用上一次的请求进行自动的填充。如果你使用了laravel的验证服务，那么你不需要手动的调用它们，因为laravel内置的验证机制会自动的调用它们。

### 闪存输入到会话

`Illuminate\Http\Request`实例的`flash`方法会闪存当前请求的输入到会话中，这样可以使应用在接受用户的下次请求时进行复用:

```php
$request->flash();
```

你也可以使用`flashOnly`和 `flashExcept` 方法来闪存部分请求输入到会话：

```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

### 闪存输入到会话接着跳转

一个常用的场景就是你需要连同用户的输入一起返回到上一页中，那么你可以使用`withInput`链式方法:
```php
return redirect('form')->withInput();

return redirect('form')->withInput($request->except('password'));
```

### 获取旧输入

你可以使用`old`方法来获取上一次请求所闪存的用户请求：

```php
$username = $request->old('username');
```

laravel提供了全局`old`帮助方法。你也可以在blade模板中使用该方法，如果上次请求未闪存该输入，则会返回`null`:

```
<input type="text" name="username" value="{{ old('username') }}">
```



## Cookies

### 从请求中检索cookie

在laravel中所有的cookie在被创建时都会经过一个认证码进行签证加密，这就意味着laravel会验证客户端对cookie的修改.你可以使用`Illuminate\Http\Request`实例的`cookie`方法来获取`cookie`值:

```php
$value = $request->cookie('name');
```

### 在响应中附加一个新的cookie

laravel提供了一个全局的`cookie`帮助方法用来生成一个`Symfony\Component\HttpFoundation\Cookie`实例，该实例可以被`Illuminate\Http\Response`实例的`withCookie`附加:

```php
$response = new Illuminate\Http\Response('Hello World');

$response->withCookie('name', 'value', $minutes);

return $response;
```

你可以使用`cookie`方法来创建一个长达5年的长cookie，它要求你使用不带参数的`cookie`帮助方法直接调用`forever`方法:

```php
$response->withCookie(cookie()->forever('name', 'value'));
```


## 文件

### 获取上传的文件

你可以通过`Illuminate\Http\Request`的`file`方法来访问上传的文件。该方法会返回一个`Symfony\Component\HttpFoundation\File\UploadedFile`类的实例，它继承自`SplFileInfo`，提供了多种与文件交互的方法：

```php
$file = $request->file('photo');
```

你可以使用`hasFile`方法来判断文件在请求中是否存在:

```php
if ($request->hasFile('photo')) {
  // 
}
```

### 验证文件是否上传成功

你可以使用`isValid`方法来验证文件上传的过程中是否出错:

```php
if ($request->file('photo')->isValid()) {
  //
}
```

### 移动上传的文件

你可以使用`move`方法来将上传的文件从临时目录中迁移到指定的目录中:

```php
$request->file('photo')->move($destinationPath);

$request->file('photo')->move($destinationPath, $fileName);
```

### 其他文件方法

`UploadedFile`还有其他许多可用的方法，你可以查看 [该类的API文档](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) 来了解更多.
