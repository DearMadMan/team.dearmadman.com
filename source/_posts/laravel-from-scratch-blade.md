title: laravel 基础教程 —— Blade 模板引擎
date: 2016-05-14 20:46:27
tags: [php, laravel]
---

# Blade 模板引擎

## 简介

Blade 是 laravel 提供的一个简单强大的模板引擎。它不像其他流行的 PHP 模板引擎那样限制你在视图中使用原生的 PHP 代码，事实上它就是把 Blade 视图编译成原生的 PHP 代码并缓存起来。缓存会在 Blade 视图改变时而改变，这意味着 Blade 并没有给你的应用添加编译的负担。Blade 视图文件使用 `.blade.php` 后缀，一般情况下都被存储在 `resources/views` 目录。

## 模板继承

### 定义布局

Blade 带来的两个主要的福利就是模板继承和挂件。为了方便开始，我们来看一个简单的例子。首先，我们建立一个主页面布局。因为多数 web 应用是在不同的页面中使用相同的总体布局，我们可以方便的定义这个布局为单独的 Blade 视图：

```
<!-- Stored in resources/views/layouts/master.blade.php-->

<html>
  <head>
    <title>App Name - @yield('title')</title>
  </head>
  <body>
    @section('sidebar')
      This is the master sidebar.
    @show

    <div class="container">
      @yield('content')
    </div>
  </body>
</html>
```

从上面的例子你可以看到，Blade 模板文件包含了典型的 HTML 标记。你肯定注意到了 `@section` 和 `@yield` 指令。`@section` 指令就如它的名字所暗示的那样定义了一个内容区块，而 `@yield` 指令是用来显示所提供的挂件区块所包含的内容。

现在我们已经定义好了一个基本的布局，接下来我们来构建一个子页面去继承这个布局。

### 扩展布局

我们可以使用 Blade 的 `@extends` 指令来明确的指定继承某个布局。然后使用 `@section` 指令将挂件中的内容挂载到布局中，在上面的例子中，挂件的内容将被挂载到布局中的 `@yield` 部分：

```php
<!-- Stored in resoures/views/child.blade.php -->

@extends('layouts.master')

@section('title', 'Page Title')

@section('sidebar')
  @parent

  <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
  <p>This is my body content.</p>
@endsection
```

在上面的例子作用 `sidebar` 挂件利用 `@parent` 指令来追加布局中的 sidebar 部分的内容，如果不使用则会覆盖掉布局中的这部分。`@parent` 指令会在视图被渲染时替换为布局中的内容。
Blade 视图可以像原生 PHP 视图一样使用全局帮助函数 `view` 来返回渲染后的内容：

```php
Route::get('blade', function () {
  return view('child');
});
```

## 显示数据

你可以使用花括号 `{` 来在视图中显示传递到视图中的变量，例如，你定义了下面的路由：

```php
Route::get('greeting', function () {
  return view('welcome', ['name' => 'Samantha']);
})
```

你可以在视图中这样来输出 `name` 变量的内容：

```php
Hello, {{ $name }}
```

当然，你并没有被限制只允许显示传递到视图中的变量的内容。你也可以从原生 PHP 方法中返回内容。事实上，你可以在 Blade echo 声明中使用任意的 PHP 代码：

```php
The current UNIX timestamp is {{ time() }}
```

> 注意：Blade `{% raw %}{{}}{% endraw %}` 声明中的内容是自动通过 PHP 的 `htmlentities` 方法过滤的，用来防止 XSS 攻击。

### Blade & JavaScript Frameworks

由于很多 JavaScript 框架都使用花括号来表明所提供的表达式应该被显示在浏览器中。所以你可以使用 `@` 符号来告诉 Blade 渲染引擎你需要这个表达式原样保留：

```php
<h1>Laravel</h1>

Hello, @{{ name }}
```

上面的例子中，`@` 符号会在 Blade 渲染时被移除，并且 `{{ name }}` 表达式会被原样保留下来。

### 输出数据假如它存在

有时候你可能希望输出一个变量，但是你并不确定它是否已经被设置，你可以使用这个表达式：

```php
{{ isset($name) ? $name : 'Default' }}
```

Blade 提供了一个便捷的方式来替换这个三元声明：

```php
{{ $name or 'Default' }}
```

上面的例子中，如果变量 `$name` 存在，它将被输入，如果不存在，`Default` 会被输出。

### 显示未转义的数据

默认的，Blade `{% raw %}{{}}{% endraw %}` 声明会自动的使用 PHP 的 `htmlentities` 方法来避免 XSS 攻击。如果你不想你的数据被转义，你可以使用下面的语法：

```php
Hello, {!! $name !!}
```

> 注意：当你在应用中输出用户输入的数据时应该非常的谨慎，你应该总是使用 `{% raw %}{{}}{% endraw %}` 来转义内容中任意的 HTML 实体。

## 控制结构

除了模板继承和数据显示，Blade 为通用的 PHP 控制结构提供了便利的简写方式，比如条件声明和循环。这些简写提供了一个干净简洁的方式来处理 PHP 的控制结构，而且还保持与 PHP 语句的相似性。

### if 声明

你可以通过 `@if`,`@elseif`,`@else`和 `@endif` 指令来使用 `if` 控制结构，这些指令和 PHP 方法保持一致:

```php
@if (count($records) === 1)
  I have one record!
@elseif (count($records) > 1)
  I have multiple records!
@else
  I don't have any records!
@endif
```

为了方便，Blade 也提供了 `@unless` 指令：

```php
@unless (Auth::check())
  You are not signed in.
@endunless
```

你也可以使用 `@hasSection` 指令来判断提供给布局的挂件是否包含了内容:

```php
<title>
  @hasSection('title')
    @yield('title') - App Name
  @else
    App Name
  @endif
</title>
```

### 循环

除了条件声明，Blade 也提供了一些简单指令来支持 PHP 的循环结构。这些指令方法和 PHP 语法相同：

```php
@for ($i = 0; $i < 10; $i++)
  The current value is {{ $i }}
@endfor

@foreach ($users as $user)
  <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
  <li>{{ $user->name }}</li>
@empty
  <p>No users</p>
@endforelse

@while (true)
  <p>I'm looping forever.</p>
@endwhile
```

Blade 也提供了终止迭代或取消当前迭代的指令：

```php
@foreach ($users as $user)
  @if($user->type == 1)
    @continue
  @endif

  <li>{{ $user->name }}</li>

  @if($user->number == 5)
    @break
  @endif
@endforeach
```

你也可以使用指令声明包含条件的方式来达到中断:

```php
@foreach ($users as $user)
  @continue($user->type == 1)

  <li>{{ $user->name }}</li>

  @break($user->number == 5)
@endforeach
```

### 包含子视图

你可以使用 `@include` 指令来包含一个视图的内容，当前视图中的变量也会被共享给子视图：

```php
<div>
  @include('shared.errors')

  <form>
  <!-- Form Contents -->
  </form>
</div>
```

尽管子视图会自动继承父视图中的所有数据变量，你也可以直接传递一个数组变量来添加额外的变量到子视图：

```php
@include('view.name', ['some' => 'data'])
```

> 主要：你应该在 Blade 视图中避免使用 `__DIR__ ` 和 `__FILE__` 常量，因为它们会解析为视图缓存所在的位置。

### 为集合渲染视图

你可以使用 Blade 的 `@each` 指令来在一行中合并引入多个视图:

```php
@each('view.name', $jobs, 'job')
```

第一个参数是数组或集合中每个元素需要被渲染的视图名称。第二个参数是一个数组或集合，被用来提供迭代。而第三个参数是要分配给当前视图的变量名。举个例子，如果你需要遍历一个数组 `jobs`，通常你想要在局部渲染的视图中使用 `job` 作为变量来访问 job 信息。

你也可以传递第四个参数到 `@each` 指令。如果所提供的数组是空数组的话，该参数所提供的视图将会被引入。

```php
@each('view.name', $jobs, 'job', 'view.empty')
```

### 注释

Blade 也允许你在视图中定义注释，但是它不会在渲染时生成 HTML 注释：

```php
{{-- This comment will not be present in the rendered HTML --}}
```

## 堆

Blade 允许你在已命名的堆中压入内容：

```php
@push('scripts')
  <script src="/example.js"></script>
@endpush
```

你可以在你需要的时候压入相同的堆任意的次数,你需要在布局中使用 `@stack` 来渲染堆:

```php
<head>
  <!-- Head Contents -->

  @stack('scripts')
</head>
```

## 服务注入

你可以使用 `@inject` 指令来从服务容器中取回服务，该指令的第一个参数将作为所取回服务存放的变量名，而第二个参数是你想要在服务容器中取回的类或接口名称：

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
  Monthly Revenue: {{ $metrice->monthlyRevenue() }}
</div>
```

## 扩展 Blade

Blade 允许你自定义一些指令，你可以使用 `directvie` 方法来注册指令。当 Blade 编译器遇到该指令时，它会自动的调用该指令注册时提供的回调函数并传递它的参数。

下面的例子创建了 `@datetime($val)` 指令来格式化 `$val`:

```php
<?php
namespace App\Providers;

use Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
  /**
   * Perform post-registration booting of services.
   *
   * @return void
   */
   public function boot()
   {
     Blade::directive('datetime', function ($expression) {
       return "<?php echo with{$express}->format('m/d/Y H:i'); ?>";
     });
   }

   /**
    * Register bindings in the container
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

上面的例子中使用了 Laravel 的 `with` 帮助方法，它只是简单的返回一个所提供的对象或值，并提供方便的链式调用。最终该指令生成的 PHP 代码如下：

```php
  <?php echo with($var)->format('m/d/Y H:i'); ?>
```

在你更新 Blade 指令的逻辑之后，你应该删除所有已缓存的 Blade 视图，你可以使用 `view:clear` Artisan 命令来清除。

