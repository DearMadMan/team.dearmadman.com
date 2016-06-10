title: laravel 基础教程 —— 帮助方法
date: 2016-06-07 23:45:45
tags: [php, laravel]
---

# 帮助方法

## 简介

Laravel 包含了多中“帮手” PHP 函数，很多方法都在框架中进行了使用，如果你发现他们很方便，你也可以在自己的应用中使用。


## 方法名单

## 数组

### array_add()

`array_add` 方法用来在数组中添加键值对，它仅会在数组中不存在所给定的键时才会添加：

```php
$array = array_add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
```

### array_collapse()

`array_collapse` 方法会将坍塌数组到一个单一的数组中。

```php
$array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### array_divide()

`array_divide` 方法将会分列数组，它会返回两个数组，一个数组包含了原数组的所有的键，另一个数组包含原数组所有的值：

```php
list($keys, $values) = array_divide(['name' => 'Desk']);

// $keys: ['name']

// $values: ['Desk']
```

### array_dot()

`array_dot` 方法将数组从多维降低为一维数组，它使用 `.` 符号来表明其深度：

```php
$array = array_dot(['foo' => ['bar' => 'baz']]);

// ['foo.bar' => 'baz'];
```

### array_except()

`array_except` 方法从数组中移除指定的键值对：

```php
$array = ['name' => 'Desk', 'price' => 100];

$array = array_except($array, ['price']);

// ['name' => 'Desk']
```

### array_first()

`array_first` 方法返回数组回调迭代中第一个返回真值的元素:

```php
$array = [100, 200, 300];

$value = array_first($array, function ($key, $value) {
  return $value >= 150; 
});

// 200
```

你也可以在第三个参数中传递一个默认值，如果迭代结束仍未返回真值，将返回默认值:

```php
$value = array_first($array, $callback, $default);
```

### array_flatten()

`array_flatten` 方法会将多维数组降为一维数组：

```php
$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$array = array_flatten($array);

// ['Joe', 'PHP', 'Ruby'];
```

### array_forget()

`array_forget` 方法可以使用 `.` 语法来删除数组中嵌套的键值对:

```php
$array = ['products' => ['desk' => ['price' => 100]]];

array_forget($array, 'products.desk');

// ['products' => []]
```

### array_get()

`array_get` 方法可以使用 `.` 语法来从数组中检索嵌套的值：

```php
$array = ['products' => ['desk' => ['price' => 100]]];

$value = array_get($array, 'products.desk');

// ['price' => 100]
```

`array_get` 方法也可以接收第三个参数，用来作为默认值，如果数组中并没有检索到相应的值，将会返回默认值：

```php
$value = array_get($array, 'names.john', 'default');
```

### array_has()

`array_has` 方法允许使用 `.` 语法来检查数组中是否含有给定的项：

```php
$array = ['products' => ['desk' => ['price' => 100]]];

$hasDesk = array_has($array, 'products.desk');

// true
```

### array_only()

`array_only` 方法会给定的数组中返回指定的键值对：

```php
$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

$array = array_only($array, ['name', 'price']);

// ['name' => 'Desk', 'price' => 100]
```

### array_pluck()

`array_pluck` 方法摘取数组中所给定的键值对：

```php
$array = [
  ['developer' => ['id' => 1, 'name' => 'Taylor']],
  ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$array = array_pluck($array, 'developer.name');

// ['Taylor', 'Abigail'];
```

你可以可以指定希望返回的结果如何键化：

```php
$array = array_pluck($array, 'developer.name', 'developer.id');

// [1 => 'Taylor', 2 => 'Abigail'];
```

### array_prepend()

`array_prepend` 方法会在数组的起始端加入项：

```php
$array = ['one', 'two', 'three', 'four'];

$array = array_prepend($array, 'zero');

// $array: ['zero', 'one', 'two', 'three', 'four']
```

### array_pull()

`array_pull` 方法从数组中返回键值对并将其在数组中进行剔除：

```php
$array = ['name' => 'Desk', 'price' => 100];

$name = array_pull($array, 'name')

// $name: Desk
// $array: ['price' => 100]
```

### array_set()

`array_set` 方法使用 `.` 语法对数组中的项进行设置值：

```php
$array = ['products' => ['desk' => ['price' => 100]]];

array_set($array, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

### array_sort()

`array_sort` 方法会根据给定闭包所返回的结果对数组进行排序：

```php
$array = [
  ['name' => 'Desk'],
  ['name' => 'Chair'],
];

$array = array_values(array_sort($array, function ($value) {
  return $value['name'];
}));

/*
   [
    ['name' => 'Chair'],
    ['name' => 'Desk'],
   ]
 */
```

### array_sort_recursive()

`array_sort_recursive` 方法会对数组进行递归的使用 `sort` 方法排序：

```php
$array = [
  [
    'Roman',
    'Taylor',
    'Li',
  ],
  [
    'PHP',
    'Ruby',
    'JavaScript',
  ],
];

$array = array_sort_recursive($array);

/*
  [
    [
      'Li',
      'Roman',
      'Taylor',
    ],
    [
      'JavaScript',
      'PHP',
      'Ruby',
    ],
  ];
 */
```

### array_where()

`array_where` 方法会根据给定的闭包对数组进行过滤：

```php
$array = [100, '200', 300, '400', 500];

$array = array_where($array, function ($key, $value) {
  return is_string($value);
});

// [1 => 200, 3 => 400]
```

### head()

`head` 方法简单的从数组中返回其首个元素：

```php
$array = [100, 200, 300];

$first = head($array);

// 100
```

### last()

`last` 方法返回所给定数组中的最后一个元素：

```php
$array = [100, 200, 300];

$last = last($array);

// 300
```

## Paths

### app_path()

`app_path` 方法返回 `app` 目录的完整路径：

```php
$path = app_path();
```

你也可以使用 `app_path` 方法来生成相对于应用目录的完整路径：

```php
$path = app_path('Http/Controllers/Controller.php');
```

### base_path()

`base_path` 方法返回项目根目录的完整路径：

```php
$path = base_path();
```

你也可以使用 `base_path` 方法来返回相对于根目录的完整路径：

```php
$path = base_path('vendor/bin');
```

### config_path()

`config_path` 方法用来返回应用的配置文件目录的完整路径：

```php
$path = config_path();
```

### database_path()

`database_path` 方法返回应用的数据库目录的完整路径：

```php
$path = database_path();
```

### elixir()

`elixir` 方法返回版本化的文件路径：

```php
elixir($file);
```

### public_path()

`public_path` 方法返回 `public` 目录的完整路径：

```php
$path = public_path();
```

### storage_path()

`storage_path` 方法返回 `storage` 目录的完整路径：

```php
$path = storage_path();
```

你也可以使用 `storage_path` 方法来生成相对目录的完整路径：

```php
$path = storage_path('app/file.txt');
```

## Strings

### camel_case()

`camel_case` 方法将给定字符串转换成 `camelCase` 格式：

```php
$camel = camel_case('foo_bar');

// fooBar
```

### class_basename()

`class_basename` 反回所给定类移除命名空间之后的类名：

```php
$class = class_basename('Foo\Bar\Baz');

// Baz
```

### e()

`e` 方法使用 `htmlentities` 方法来过滤给定字符串：

```php
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
```

### ends_with()

`ends_with` 方法用来判断给定的字符串是否以给定的值结尾：

```php
$value = ends_with('This is my name', 'name');

// true
```

### snake_case()

`snake_case` 方法将字符串转换成 `snake_case` 格式：

```php
$snake = snake_case('fooBar');

// foo_bar
```

### str_limit()

`str_limit` 方法用来限制字符串的长度。该方法的第一个参数应该是一个字符串，而第二个参数应该是允许返回结果的最大长度：

```php
$value = str_limit('The PHP framework for web artisans.', 7);

// The PHP...
```

### starts_with()

`starts_with` 方法用来判断给定的字符串是否已给定的值起始：

```php
$value = starts_with('This is my name', 'This');

// true
```

### str_finish()

`str_finish` 方法可以在字符串的结尾添加独特的值（如果字符串不是以该值结尾）：

```php
$string = str_finish('this/string', '/');

// this/string/
```

### str_is()

`str_is` 方法用来判断给定的字符串是否匹配给定的模式。可以使用星号作为通配符：

```php
$value = str_is('foo*', 'foobar');

// true

$value = str_is('baz*', 'foobar');

// false
```

### str_plural()

`str_plural` 方法将字符串转换为其相应的复数形式，该方法目前只支持英语：

```php
$plural = str_plural('car');

// cars

$plural = str_plural('child');

// children
```

你可以传递一个整型值到第二个参数来表明返回单数或复数形式：

```php
$plural = str_plural('child', 2);

// children

$plural = str_plural('child', 1);

// child
```

### str_random()

`str_random` 方法根据指定的长度生成随机字符串：

```php
$string = str_random(40);
```

### str_singular()

`str_singular` 方法将字符串转换为单数形式，目前只支持英语：

```php
$singular = str_singular('cars');

// car
```

### str_slug()

`str_slug` 方法使用给定的胶连字符对给定的字符串生成 URL:

```php
$title = str_slug('Laravel 5 Framework', '-');

// laravel-5-framework
```

### studly_case()

`studly_case` 方法转换给定的字符串到 `StudlyCase` 格式：

```php
$value = studly_case('foo_bar');

// FooBar
```

### trans()

`trans` 方法根据本地文件中的语言来进行翻译：

```php
echo trans('validation.required');
```

### trans_choice()

`trans_choice` 方法来转义到给定的语言，并使用相应的单复数形式：

```php
$value = trans_choice('foo.bar', $count);
```

## URLs

### action()

`action` 方法根据给定的控制器动作生成相应的 URL。你不需要传递完整的命名空间。默认的所传递的控制器类名是相对于 `App\Http\Controllers` 的命名空间：

```php
$url = action ('HomeController@getIndex');
```

如果方法接受路由参数，你可以传递第二个参数到该方法：

```php
$url = action('UserController@profile', ['id' => 1]);
```

### asset()

根据当前的请求方式来返回指定资源的地址：

```php
$url = asset('img/photo.jpg');
```

### secure_asset()

使用 HTTPS 生成给定资源的 URL：

```php
echo secure_asset('foo/bar.zip', $title, $attributes = []);
```

### route()

`route` 方法根据给定的路由名称来生成 URL：

```php
$url = route('routeName');
```

如果路由接受参数，你可以传递第二个参数到方法：

```php
$url = route('routeName', ['id' => 1]);
```

### url()

`url` 方法根据指定的路径生成完整的路径：

```php
echo url('user/profile');

echo url('user/profile', [1]);
```

如果没有路径指定，将返回 `Illuminate\Routing\UrlGenerator` 的实例：

```php
echo url()->current();
echo url()->full();
echo url()->previous();
```

## Miscellaneous

### auth()

`auth` 方法返回一个认证实例。你可以方便的替换 `Auth` 假面：

```php
$user = auth()->user();
```

### back()

`back` 方法生成重定向响应到用户之前的地址：

```php
return back();
```

### bcrypt()

`bcrypt` 方法使用 Bcrypt 加密来哈希化给定的值。你也可以通过 `Hash` 假面来调用：

```php
$password = bcrypt('my-secret-password');
```

### collect()

`collect` 方法根据给定的项目组来生成 `collection` 实例：

```php
$collection = collect(['taylor', 'abigail']);
```

### config()

`config` 根据给定的值来获取配置项的值。你可以使用 `.` 语法来获取配置项的值。也可以传递第二个参数作为配置项未找到时的默认值：

```php
$value = config('app.timezone');

$value = config('app.timezone', $default);
```

你也可以在运行时使用键值对的方式对配置进行设置：

```php
config(['app.debug' => true]);
```

### csrf_field()

`csrf_field` 方法用来生成一个 `hidden` 文本框字段来包含 CSRF token。你可以在 Blade 模板中使用：

```php
{{ csrf_field() }}
```

### csrf_token()

`csrf_token` 方法返回当前的 CSRF token 值：

```php
$token = csrf_token();
```

### dd()

`dd` 方法打印输出给定的变量并终止执行之后的代码：

```php
dd($value);
```

如果你不想停止执行之后的代码，你应该使用 `dump` 方法：

```php
dump($value);
```

### dispatch()

`dispatch` 方法在 laravel 的任务对了中添加一个新的任务：

```php
dispatch(new App\Jobs\SendEmails);
```

### env()

`env` 方法用来获取环境变量，也可以在未设置环境变量时返回默认值：

```php
$env = env('APP_ENV');

// Return a default value if the variable doesn't exist...
$env = env('APP_ENV', 'production');
```

### event()

`event` 方法分发给定的事件到它的监听器中：

```php
event(new UserRegistered($user));
```

### factory()

`factory` 方法创建一个模型工厂构造器来生成给定的类名的实例。你可以在写单元测试或者 seeding 时使用：

```php
$user = factory(App\User::class)->make();
```

### method_field()

`method_field` 方法生成一个 `hidden` 文本框来包含一个欺骗性的 HTTP 请求动词。你可以在 Blade 模板中使用：

```php
<form method="POST">
  {{ method_field('DELETE') }}
</form>
```

### old()

`old` 方法返回 session 中闪存的旧的文本值：

```php
$value = old('value');

$value = old('value', 'default');
```

### redirect()

`redirect` 方法返回一个重定向的实例来做重定向响应：

```php
return redirect('/home');
```

### request()

`request` 方法返回当前的请求实例或者检索请求中的输入项：

```php
$request = request();

$value = request('key', $default = null)
```

### response()

`response` 方法生成一个响应实例或者从响应工厂中获得一个实例：

```php
return response('Hello World', 200, $headers);

return response()->json(['foo' => 'bar'], 200, $headers);
```

### session()

`session` 方法可以用来获取或者设置 session 值：

```php
$value = session('key');
```

你可以通过传递键值对来进行 session 值的设置：

```php
session(['chairs' => 7, 'instruments' => 3]);
```

如果调用的是无参数的 `session` 方法，将返回 session 存储器：

```php
$value = session()->get('key');

session()->put('key', $value);
```

### value()

`value` 方法会简单的返回所给定的值。但是，如果你传递的是一个 `Closure` ，`Closure` 将会被执行，其结果将被返回：

```php
$value = value(function () {
  return 'bar';
});
```

### view()

`view` 方法用来检索 `view` 实例：

```php
return view('auth.login');
```

### with()

`with` 方法返回给定的值。它主要用于方法的链式调用：

```php
$value = with(new Foo)->work();
```