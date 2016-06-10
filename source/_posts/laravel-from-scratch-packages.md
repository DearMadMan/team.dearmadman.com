title: laravel 基础教程 —— 包开发
date: 2016-06-08 21:46:33
tags: [php, laravel]
---

# 包开发

包是在 laravel 中添加功能的主要方式。包可以是任何形式，比如处理日期的 [Carbon](https://github.com/briannesbitt/Carbon) 或者是整个 BDD 测试框架 [Behat](https://github.com/Behat/Behat)。

当然，包是有不同类型的。有些包是独立的包。意味着它可以在任何框架中使用，而不仅仅是在 laravel 中。Carbon 和 Behat 就是独立的包。这些包你可以简单的通过在 `composer.json` 文件中进行引入安装使用。

而在另一方面，有些包是专门用于 laravel 使用的。这些包可能拥有路由，控制器，视图和配置文件，他们配合起来旨在提高 laravel 的功能。这篇指南就是来告诉你如何开发增强 laravel 功能的包。


## 服务提供者

服务提供者是 laravel 和包的连接点。服务提供者主要负责包内容在服务容器中的绑定以及应用应该如何加载资源文件如视图，配置文件和语言文件。

一个服务提供者应该继承 `Illuminate\Support\ServiceProvider` 类并且包含两个方法： `register` 和 `boot`。基类 `ServiceProvider` 类位于 Composer 包的 `illuminate/support` 中。你应该在你的包中添加这个依赖。


## 路由

你可以在你的服务提供者的 `boot` 方法中简单的 `require` 路由文件来定义包的路由。在你的路由文件中，你可以使用 `Route` 假面来注册路由，其方式就如普通的 laravel 应用一样：

```php
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
 public function boot()
 {
   if (! $this->app->routesAreCached(){
     requir __DIR__.'/../../routes.php';
   })
 }
```

## 资源

### 视图

你需要告诉 laravel 视图的位置才能使 laravel 加载包中的视图。你可以通过服务提供者的 `loadViewsFrom` 方法。`loadViewsFrom` 方法接受两个参数：视图的路径和包的名称。比如，如果你的包名称是“courier”，你应该像下面一样在 `boot` 中添加：

```php
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
 public function boot()
 {
   $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
 }
```

包视图的使用方式是通过 `package::view` 类似的语法引用的。所以，你可以像这样从 `courier` 包中引入 `admin` 视图：

```php
Route::get('admin', function () {
  return view('courier::admin'); 
});
```

**覆盖包视图**

当你使用 `loadViewsFrom` 方法来加载视图时，laravel 实际上是注册了两个视图位置：一个是应用的 `resources/views/vendor` 目录，另一个是你指定的目录。所以，我们还使用上面的例子：当请求引入包视图时，laravel 会首先检查 `resources/views/vendor/courier` 目录中是否有相应的视图，然后，如果没有才会通过 `loadViewFrom` 方法来加载指定的目录下的视图。这就引入了一种自定义包视图的简便方式。

**发布视图**

如果你希望有一种简单的方式将包的视图发布到 `resources/views/vendor` 目录。你可以在服务提供者中使用 `publishes` 方法。`publishes` 方法接收一个包视图路径和其发布地址路径所组成的数组：

```php
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
 public function boot()
 {
   $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

   $this->publishes([
     __DIR__./path/to/views' => resource_path('views/vendor/courier'),
   ]);
 }
```

现在，当你通过命令行工具使用 `vendor:publish` Artisan 命令时，你的包视图会被复制到其所指定的位置。


### 译文

如果你的包中包含了翻译文件。你可以使用 `loadTranslationsFrom` 方法来指导 laravel 如何载入它们。比如，如果你的包名称为 'courier'，你应该使用如下的方式在你的服务提供者的 `boot` 方法中添加：

```php
/**
* Perform post-registration booting of services
*
* @return void
*/
public function boot()
{
  $this->loadTranslationsFrom(__DIR__./path/to/translations', 'courier');
}
```

包译文的引入使用 `package::filer.line` 类似的语法。所以，你可以像这样来加载 `courier` 包中的 `messages` 文件的 `welcome` 键：

```php
echo trans('courier::messages.welcome');
```

**发布译文**

如果你希望发布包的译文到应用的 `resources/lang/vendor` 目录，你可以使用服务提供者的 `publishes` 方法。`publishes` 方法接收一个包含包路径和其相应的发布路径所组成的数组。比如，发布 `courier` 包中的译文：

```php
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
 public function boot()
 {
   $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

   $this->publishes([
     __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier')
   ]);
 }
```

现在，当你通过命令行工具使用 `vendor:publish` Artisan 命令时，包中的译文会自动的发布到指定的位置。

### 配置文件

通常，你希望发布你的包配置文件到应用的 `config` 目录。这样就允许用户通过简单的配置来覆盖默认的配置选项。你可以使用 `publishes` 方法来发布配置文件：

```php
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
 public function boot()
 {
   $this->publishes([
     __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
   ]);
 }
```

现在当使用你包的用户使用 laravel 的 `vendor:push` 命令时，你的配置文件将会复制到指定的位置。当然，一旦你的配置文件被发布到 `config` 目录，它就可以像访问其他配置文件一样被访问：

```php
$value = config('courier.option');
```

**默认的包配置**

你也可以选择合并包的默认配置和应用复制的配置。这样就允许用户只引入其真实需要变更的配置选项。你可以使用 `mergeConfigFrom` 方法来在你的服务提供者的 `register` 方法中进行合并：

```php
/**
 * Register bindingsin the container.
 *
 * @return void
 */
 public function register()
 {
   $this->mergeConfigFrom(
     __DIR__.'/path/to/config/courier.php', 'courier'
   );
 }
```

## 公共资源文件

你的包文件中可能含有一些资源文件比如 JavaScript，CSS，图片。你同样可以在你的服务提供者中使用 `publishes` 方法来发布这些资源到应用的 `public` 目录。我们同样可以添加一个 `public` 组标签，这样可以选择在发布时只发布 `public` 标签组的文件：

```php
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
 public function boot()
 {
   $this->published([
     __DIR__.'/path/to/assets' => public_path('vendor/courier'),
   ], 'public');
 }
```

现在，使用你的包的用户在执行 `vendor:publish` 命令时，你的资源文件会被发布到指定的位置。如果你需要每次发布资源时覆盖之前发布的内容，你可以使用 `--force` 标识：

```php
php artisan vendor:publish --tag=public --force
```

如果你希望你的公共资源总是在发布时是最新的版本，你应该在你的 `composer.json` 文件中的 `post-update-cmd` 列中添加上述命令。


## 发布组文件

你可能希望将前端资源和后端资源进行分组分开发布。比如，你可能希望用户在发布配置文件的同时并不重新发布更新公共资源文件。你可以通过在使用 `publishes` 方法时对发布项打标签的方式来对发布进行分组：

```php
/**
 * perform post-registration booting of services.
 *
 * @return void
 */
 public function boot()
 {
   $this->publishes([
     __DIR__.'/../config/package.php' => config_path('package.php')
   ], 'config');

   $this->publishes([
     __DIR__.'/../database/migrations/' => database_path('migrations')
   ], 'migrations');
 }
```

现在，你的包用户可以在发布文件时通过组标签来分开发布了：

```php
php artisan vendor:publish --provider="Vendor\Providers\PackageServiceProvider"
```