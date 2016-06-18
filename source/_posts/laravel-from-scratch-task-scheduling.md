title: laravel 基础教程 —— 任务计划
date: 2016-06-18 09:10:06
tags: [php, laravel]
---

# 任务计划（Task Scheduling）

## 简介

在过去，开发者需要手动的在计划表中添加一行来录入定时执行的任务。这是非常让人头疼的，因为你不得不手动的登录远端服务器去做这些事情，它并不能在代码中有效的控制。Laravel 的命令调度者允许你在 Laravel 中流利通畅的定义你的任务计划，并且，你只需要为此在服务器中增加一个单独的定时任务条目就可以了，之后就可以在代码中进行控制任务计划的数量。

你的任务计划被定义在 `app/Console/Kernel.php` 文件的 `schedule` 方法中。在开始之前，我们先看一个简单的例子。你可以在 `Schedule` 对象中添加任意你希望执行的任务计划。

### 开始运行计划

你只需要在你的服务器的 Cron 项中添加如下条目：

```php
* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

该 Cron 会每分钟调用 Laravel 的命令调度来执行计划任务。Laravel 会自动的评估你的任务计划并执行到期的任务。


## 定义任务

你可以在 `App\Console\Kernel` 类的 `schedule` 方法中定义所有的任务计划。在开始之前，先让我们看一个任务计划的简单例子。在这个例子中，我们会在每天的午夜执行 `Closure`。在 `Closure` 中我们会执行清除数据库表的查询语句：

```php
<?php

namespace App\Console;

use DB;
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
  /**
   * The Artisan commands provided by your application.
   *
   * @var array
   */
   protected $commands = [
     \App\Console\Commands\Inspire::class,
   ];

   /**
    * Define the application's command schedule.
    *
    * @param \Illuminate\Console\Scheduling\Schedule $schedule
    * @return void
    */
    protected function schedule(Schedule $schedule)
    {
      $schedule->call(function () {
        DB::table('recent_users')->delete();
      })->daily();
    }
}
```

除了可以调度 `Closure` 调用，你也可以调度 Artisan 命令和操作系统的命令。比如，你可以使用 `command` 方法来调度一个 Artisan 命令：

```php
$schedule->command('emails:send --force')->daily();
```

你可以使用 `exec` 方法来发布一个命令到操作系统中：

```php
$schedule->exec('node /home/forge/script.js')->daily();
```

### 调度频率选项

当然，这里还有各种可以分配到任务的调度方法：

方法                               | 描述
---                                | ---
`->cron('* * * * * *');`           | 执行自定义的 Cron 任务计划
`->everyMinute();`                 | 每分钟执行一次任务
`->everyFiveMinutes();`            | 每五分钟执行一次任务
`->everyTenMinutes();`             | 每十分钟执行一次任务
`->hourly();`                      | 每一小时执行一次任务
`->daily();`                       | 每天的午夜执行一次任务
`->dailyAt('13:00');`              | 每天的 13:00 执行一次任务
`->twiceDaily(1, 13);`             | 每天的 1:00 & 13:00 执行一次任务
`->weekly();`                      | 每周执行一次任务
`->montyly();`                     | 每月执行一次任务
`->monthlyOn(4, '15:00');`         | 每月的 4 号 15:00 执行一次任务
`->quarterly();`                   | 每季度执行一次任务
`->yearly();`                      | 每年执行一次任务
`->timezone('America/New_York');` | 设置时区

这些方法可以与额外的限制相结合以创建更加细微调整的计划，比如仅在一周的某几天运行。让我们调度一个计划命令来在每周的周一执行：

```php
// Run once per week on Monday at 1 PM...
$schedule->call(function () {
  //
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
$schedule->command('foo')
         ->weekdays()
         ->hourly()
         ->timezone('America/Chicago')
         ->when(function () {
           return date('H') >= 8 && date('H') <= 17;
         });
```

下面列出了额外的计划约束条件：

方法               | 描述
---                | ---
`->weekdays();`    | 限制在平常日内（周六、日除外）
`->sundays();`     | 限制在周日
`->mondays();`     | 限制在周一
`->tuesdays();`    | 限制在周二
`->wednesdays();`  | 限制在周三
`->thursdays();`   | 限制在周四
`->fridays();`     | 限制在周五
`->staturdays();`  | 限制在周六
`->when(Closure);` | 限制基于真值测试


**真值约束**

`when` 方法可以基于给定的真值测试的结果来约束一个任务的执行。换种说法就是，如果给定的 `Closure` 返回 `true`，那么只要其他约束并不阻止任务的执行时，该任务就会被执行：

```php
$schedule->command('emails:send')->daily()->when(function () {
  return true;
});
```

`skip` 方法刚好与 `when` 相反。如果 `skip` 方法返回 `true`，那么调度任务将不会执行：

```php
$schedule->command('emails:send')->daily()->skip(function () {
  return true; 
});
```

当链式调用 `when` 方法时，只有在所有的 `when` 约束返回 `true` 时才会执行调度任务命令。

### 避免任务重叠

默认的，如果前一个任务还在进程中，计划任务还是会再次运行的。你可以使用 `withoutOverlapping` 方法来避免这个：

```php
$schedule->command('emails:send')->withoutOverlapping();
```

在这个例子中，`emails:send` Artisan 命令每分钟都会被调度，但是只有在进程中没有运行该命令时才会再次执行。`withoutOverlapping` 方法对于无法确定执行时间的任务特别有效，这样就可以避免同时执行越来越多的耗时任务增大服务器的压力。

## 任务输出

Laravel 的任务计划提供了多种方便的方法来生成计划任务的输出。首先，你需要使用 `sendOutputTo` 方法，你可以传递一个文件路径到方法以便之后的检查：

```php
$schedule->command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
```

如果希望追加内容到给定的文件，你应该使用 `appendOutputTo` 方法：

```php
$schedule->command('emails:send')
         ->daily()
         ->appendOutputTo($filePath);
```

你可以使用 `emailOutputTo` 方法来将输出发送到你选定的邮箱地址中。但是你需要注意的是，你必须先使用 `sendOutputTo` 方法将输出发送到文件中。并且，在通过邮件发送任务的输出之前，你需要先配置好 laravel 的邮件服务：

```php
$schedule->command('foo')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('foo@example.com')
```

> 注意：`emailOutputTo` 和 `sendOutputTo` 方法只能在 `command` 方法中执行，并不支持 `call` 方法的调用。

## 任务钩子

你可以使用 `before` 和 `after` 方法来在任务计划执行或者完成时执行特定的操作：

```php
$schedule->command('emails:send')
         ->daily()
         ->before(function () {
           // Task is about to start...
         })
         ->after(function () {
           // Task is complete...
         });
```

**Pingings URLs**

使用 `pingBefore` 和 `thenPing` 方法，任务调度可以自动的在任务完成之前或者之后 ping 给定的 URL。这些方法通常用来通知外部的服务。比如 [Laravel Envoyer](https://envoyer.io/)，告知其计划任务将要执行或者已经完成：

```php
$schedule->command('emails:send')
         ->daily()
         ->pingBefore($url)
         ->thenPing($url);
```

使用 `pingBefore($url)` 或者 `thenPing($url)` 方法都需要引入 Guzzle HTTP 类库。你可以通过 Composer 来进行安装：

```php
"guzzlehttp/guzzle": "~5.3|~6.0"
```