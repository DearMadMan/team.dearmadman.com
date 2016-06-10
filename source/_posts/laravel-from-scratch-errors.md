title: laravel 基础教程 —— 错误和日志
date: 2016-06-05 21:28:09
tags: [php, laravel]
---

# 错误和日志

## 简介

当你开始一个新的 laravel 项目时，你一定会需要到对错误和异常的处理，而这些 laravel 都已经为你配置好了。另外，laravel 还集成了 [Monolog](https://github.com/Seldaek/monolog) 日志组件库，它提供了各种强大的日志处理器。

## 配置

**错误详情**

你的应用中通过浏览器来展示的错误详情程度是通过你的 `config/app.php` 配置文件中的 `debug` 选项来进行配置的。默认的该配置项遵从 `.env` 文件中的 `APP_DEBUG` 环境变量。

**日志模式**

larvel 提供了几种开箱即用的日志模式：`single`，`daily`，`syslog` 和 `errorlog`。比如，如果你希望使用每日日期文件记录日志来替换默认的单文件记录方式。你可以简单的在 `config/app.php` 配置文件中该设置 `log` 选项的值：

```php
'log' => 'daily'
```

当使用 `daily` 日志模式时，laravel 默认只会保留 5 天内的日志文件。如果你需要保留更多的日志文件，你需要在 `config/app.php` 文件中添加 `log_max_files` 选项：

```php
'log_max_files' => 30
```

**自定义 Monolog 配置**

如果你想要在应用中对 Monolog 的配置拥有完全的控制，你可以使用应用的 `configureMonologUsing` 方法。你应该在 `bootstrap/app.php` 文件中进行方法的调用，并且你应该把它放置在返回 `$app` 之前：

```php
$app->configureMonologUsing(function ($monolog) {
  $monolog->pushHandler(...); 
});

return $app;
```

默认的，laravel 会记录所有等级的日志。在生产环境中，你可能会想要只记录某些等级的日志，你可以通过在 `app.php` 配置文件中添加 `log_level` 选项来做到。laravel 会记录这个等级的日志和比这个等级日志更高级的日志。比如，设置 `log_level` 为 `error`，那么它会记录 `error`，`critical`，`alert` 和 `emergency` 等级的消息：

```php
'log_level' => app_env('APP_LOG_LEVEL', 'debug'),
```

## 异常处理

所有的异常都会在 `App\Exceptions\Handler` 类中进行处理。该类包含了两个方法：`report` 和 `render`。我们将会对这个两个方法进行详细的剖析。

### Report 方法

`report` 方法被用来记录异常或者发送异常到其他的服务中去，比如 [BugSnag](https://bugsnag.com/) 或者 [Sentry](https://github.com/getsentry/sentry-laravel)。默认的，`report` 方法只是简单的传递异常到异常记录的基类中。事实上，你可以自由的按照自己的希望去记录异常。

比如，如果你希望用不同的方式来记录不同类型的异常，你可以使用 PHP 的 `instanceof` 方法来进行筛选操作：

```php
/**
 * Report or log an exception.
 *
 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
 *
 * @param \Exception $e
 * @return void
 */
 public function report(Exception $e)
 {
   if ($e instanceof CustomException) {
     //
   }

   return parent::report($e);
 }
```

**根据类型来忽略异常**

异常处理类的 `$dontReport` 属性包含了一个需要忽略的异常类数组。这个数组中类型的异常将不会被记录。默认的 404 类型的错误就不会记录在日志文件中，你可以按需在这个数组中进行追加忽略的异常类名。

### Render 方法

`render` 方法用来将异常转换为传递回浏览器的响应信息。默认异常会被传递到基类并返回一个响应。事实上，你可以自由的按需进行定制化响应：

```php
/**
 * Render an exception into an HTTP response.
 *
 * @param \Illuminate\Http\Request $request
 * @param \Exception $e
 * @return \Illuminate\Http\Response
 */
public function render($request, Exception $e)
{
  if ($e instanceof CustomException) {
    return response()->view('errors.custom', [], 500);
  }

  return parent::render($request, $e);
}
```

## HTTP 异常

有些异常是直接从服务器来描述 HTTP 的错误代码。比如，这可能是一个“页面没有找到”的错误（404），一个“未授权”的错误（401）或者是一个“开发错误”（500）。为了在你的应用中快速的生成这种类型的响应，你可以使用下面的方式：

```php
abort(404);
```

`abort` 方法会立即的提出一个异常，该异常会在异常处理中被渲染。你也可以在该方法中提供一个可选的响应文本：

```php
abort(403, 'Unauthorized action.');
```

该方法可以在请求的生命周期中的任意时间点使用。

### 自定义 HTTP 错误页面

laravel 使根据 HTTP 响应状态码来创建自定义的错误页面非常简单。比如，你可能希望自定义一个错误页面来提供给 404 HTTP 状态码。你可以创建一个 `resoucres/views/errors/404.blade.php` 文件。该文件会在应用抛出 404 错误时自动的提供服务。

在该目录下的视图的命名应该与 HTTP 状态码相对应。

## 日志

laravel 的日志系统是基于强大的 [Monolog](http://github.com/seldaek/monolog) 类库的。默认的，laravel 设置了 `storage/logs` 目录来存放日志文件。你可以使用 `Log` 假面来记录日志信息：

```php
<?php

namespace App\Http\Controllers;

use Log;
use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller 
{
  /**
   * Show the profile for the given user.
   * 
   * @param int $id
   * @return Response
   */
   public function showProfile($id)
   {
     Log::info('Showing user profile for user: ' . $id);

     return view('user.profile', ['user' => User:findOrFail($id)]);
   }
}
```

日志记录器根据 [RFC 5424](http://tools.ietf.org/html/rfc5424) 规范定义了 8 中日志等级：**emergency**,**alert**,**critical**,**error**,**warning**,**notice**,**info** 和 **debug**。

```php
Log::emergency($error);
Log::alert($error);
Log::critical($error);
Log::error($error);
Log::warning($error);
Log::notice($error);
Log::info($error);
Log::debug($error);
```

**上下文信息**

你可以在入日志方法中传递一个上下文数据的数组，这个上下文数据将会被格式化并在日志信息中显示：

```php
Log::info('User failed to login.', ['id' => $user->id]);
```

**访问底层的 Monolog 实例**

Monolog 拥有多种额外日志处理方法。如果你需要，你可以在 laravel 中使用下面的方式访问底层的 Monolog 实例：

```php
$monolog = Log::getMonolog();
```