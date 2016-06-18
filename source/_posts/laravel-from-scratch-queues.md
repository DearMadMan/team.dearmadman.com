title: laravel 基础教程 —— 队列
date: 2016-06-10 12:15:50
tags: [php, laravel]
---

# 队列

## 简介

laravel 的队列服务对各种不同的后台队列服务提供了统一的 API。队列允许你延迟执行消耗时间的任务，比如发送一封邮件。这样可以有效的降低请求响应的时间。

### 配置

队列的配置文件被存储在 `config/queue.php` 中。在这个文件中你会发现框架所支持的队列驱动的配置连接示例。这些驱动包括：数据库，[Beanstalkd](http://kr.github.com/beanstalkd)，[Amazon SQS](http://aws.amazon.com/sqs)，[Redis](http://redis.io/)，和一个同步（本地使用）的驱动。

还有一个名为 `null` 的驱动表明不使用队列任务。

### 队列先决条件

**数据库**

如果使用 `database` 队列驱动，你需要添加一个数据表来处理队列任务。你可以使用 `queue:table` Artisan 命令来生成一个迁移表。一旦该迁移表生成完成，你就可以使用 `migrate` 命令来迁移到数据库中：

```php
php artisan queue:table

php artisan migrate
```

**其他队列依赖**

下面列出了其它队列驱动及其相应的依赖：
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`

## 编写任务类

### 生成任务类

默认的，所有的可队列执行的任务都被存储在 `app/Jobs` 目录下，你可以通过 Artisan 命令来生成一个新的队列任务：

```php
php artisan make:job SendReminderEmail
```

该命令会在 `app/Jobs` 目录下生成一个新的类。该类会实现 `Illuminate\Contracts\Queue\ShouldQueue` 接口，该接口表明 laravel 应该将该任务添加到后台的任务队列中，而不是同步执行。

### 任务类结构

任务类是十分简单的，通常它只包含一个 `handle` 方法来在队列任务执行时被调用。我们来看一个简单的任务示例：

```php
<?php

namespace App\Jobs;

use App\User;
use App\Jobs\Job;
use Illuminate\Contracts\Mail\Mailer;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queus\ShouldQueue;

class SendReminderEmail extends Job implements ShouldQueue
{
  use InteractsWithQueue, SerializesModels;

  protected $user;

  /**
   * Create a new job instance.
   *
   * @param User $user
   * @return void
   */
   public function __construct(User $user)
   {
     $this->user = $user;
   }

   /**
    * Execute the job.
    *
    * @param Mailer $mailer
    * @return void
    */
    public function handle(Mailer $mailer)
    {
      $mailer->send('emails.reminder', ['user' => $this->user], function () {
        //
      });

      $this->user->reminders()->create(...);
    }
}
```

在这个例子中，你需要注意的是，我们可以直接的在队列任务的构造函数中传送一个 Eloquent 模型。因为我们引入了 `SerializesModels` trait，所以当队列任务执行时，Eloquent 模型会被优雅的序列化和反序列化。如果队列任务在构造器中接收了 Eloquent 模型，那么队列任务只会序列化模型的 ID。而在任务需要进行处理时，队列系统会从数据库中自动的根据 ID 检索出模型实例。这在应用中完全是透明的，这样就可以避免了序列化完整的模型可能在队列中出现的问题。

`handle` 方法会在队列任务执行时进行调用。你需要知道的是，我们可以在任务的 `handle` 方法中可以使用类型提示来进行依赖的注入。laravel 的服务容器会自动的将这些依赖注入进去。

**有异常发生**

如果任务进行的过程中有异常被抛出。它会自动的将任务释放，同时追加到队列的尾端以使任务可以进行再次尝试。该任务会被持续释放进行尝试除非尝试的次数超出了所设置的最大次数。你可以在队列监听器 `queue:listen` 或者 `queue:work` Artisan 任务命令中添加 `--tries` 选项来设置最大可尝试次数。我们将在后续篇幅中详细的介绍队列监听器。

**手动的释放任务到队列尾端**

你可以使用 `release` 方法来进行手动的释放任务。laravel 命令生成的任务类中已经引入了 `InteractsWithQueue` trait，这个性状提供了访问任务队列的 `release` 方法。该方法接收一个参数：你希望任务再次可用的间隔（秒）：

```php
public function handle(Mailer $mailer)
{
  if (condition) {
    $this->release(10);
  }
}
```

**检查任务已尝试的次数**

就如上面我们所谈到的，如果在任务进行的过程中，异常出现，那么队列会自动的释放任务并将任务推送到队列的尾端使其可以进行再次尝试。你可以通过 `attempts` 方法来检查任务的释放次数：

```php
public function handle(Mailer $mailer)
{
  if ($this->attempts() > 3) {
    //
  }
}
```

## 推送任务到队列

位于 `app/Http/Controllers/Controller.php` 的 laravel 基础控制器使用了 `DispatchesJobs` trait。这个性状提供了一些方法允许你方便的推送任务到队列。比如 `dispatch` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\User;
use Illuminate\Http\Request;
use App\Jobs\SendReminderEmail;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Send a reminder e-mail to a given user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
   public function sendReminderEmail(Request $request, $id)
   {
     $user = User::findOrFail($id);

     $this->dispatch(new SendReminderEmail($user));
   }
}
```

**`DispatchesJobs` 性状**

当然，有时候你可能希望在应用中的任何地方进行任务的分发，而不仅仅只在路由或者控制器中。出于这个原因，你可以使用 `DispatchesJobs` trait 到任何你需要在应用中调用分发任务的类中，这样你就可以在这个类中使用 `dispatch` 方法，下面给出一个简单的使用 trait 的示例：

```php
<?php

namespace App;

use Illuminate\Foundation\Bus\DispatchesJobs;

class ExampleClass
{
  use DispatchesJobs;
}
```

**`dispatch` 方法**

或者，你可以使用 `dispatch` 全局帮助方法：

```php
Route::get('/job', function () {
  dispatch(new App\Jobs\PerformTask); 

  return 'Done!';
});
```

**对任务指定队列**

你也可以指定任务需要被分配到的队列。

你可以对你的队列任务进行分类，来将任务推送到不同的队列中，你甚至是可以分配各种队列工作的优先权。这并不是推送任务到队列配置文件中所定义的不同的队列连接上，而是仅在单个连接中指定的队列进行的操作。你可以使用任务实例的 `onQueue` 方法来指定队列。`onQueue` 来自 `Illuminate\Bus\Queueable` trait，该性状已经被 `App\Jobs\Job` 基类所引入：

```php
<?php

namespace App\Http\Controllers;

use App\User;

use Illuminate\Http\Request;
use App\Jobs\SendReminderEmail;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Send a reminder e-mail to a given user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
  public function sendReminderEmail(Request $request, $id)
  {
    $user = User::findOrFail($id);

    $job = (new SendReminderEmail($user))->onQueue('emails');

    $this->dispatch($job);
  }
}
```

### 延迟任务

有时候，你可能会想要延迟执行队列任务。比如，你可能希望有一个队列任务可以在用户注册后的 5 分钟发送一封提醒邮件。你可以在任务类中使用 `delay` 方法来完成，该方法是通过 `Illuminate\Bus\Queueable` 性状提供的：

```php
<?php

namespace App\Http\Controllers;

use App\User;
use Illuminate\Http\Request;
use App\Jobs\SendReminderEmail;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Send a reminder e-mail to a given user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
   public function sendReminderEmail(Request $request, $id)
   {
     $user = User::findOrFail($id);

     $job = (new SendReminderEamil($user))->delay(60 * 5);

     $this->dispatch($job);
   }
}
```

在这个例子中，我们指定了任务应该在队列中延迟 5 分钟执行。

> 注意： Amazon SQS 服务有最大延迟执行限制，其最多可延迟 15 分钟。

## 任务事件

**任务生命周期事件**

你可以使用 `Queue::before` 和 `Queue::after` 方法来注册一个回调在队列任务开始之前或者队列执行成功之后调用。回调为添加额外的日志，持续执行子任务或者增加统计信息等提供了非常好的机会。比如，我们可以在 laravel 的 `AppServiceProvider` 中定义一个任务成功执行后的事件监听，并附加一个回调到事件中：

```php
<?php

namespace App\Providers;

use Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobProcessed;

class AppServiceProvider extends ServiceProvider
{
  /**
   * Bootstrap any application services.
   *
   * @return void
   */
   public function boot()
   {
     Queue::after(function (JobProcessed $event) {
       // $event->connectionName
       // $event->job
       // $event->data
     });
   }

   /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

## 运行队列监听器

**开始进行队列监听**

laravel 包含了一个 Artisan 命令来运行推送到队列中的任务的执行。你可以使用 `queue:listen` 命令来运行监听器：

```php
php artisan queue:listen
```

你也可以指定监听哪一个连接的队列：

```php
php artisan queue:listen connection-name
```

你需要注意的是，这个命令执行，它会持续的运行，除非你手动的进行停止。你可以通过使用 [Supervisor](http://supervisord.org/) 进程监控来确保队列监听器的运行。

**队列优先级**

你可以通过使用 `,` 来分割连接的队列，以确定队列的运行优先级：

```php
php artisan queue:listen --queue=high,low
```
在这个例子中，任务会优先执行 `high` 的队列，然后才会运行 `low` 队列中的任务。

**指定任务的超时时间**

你可以设置任务的超时时间：

```php
php artisan queue:listen --timeout=60
```

**指定队列的睡眠时间**

另外，你可以指定队列轮询新的任务需要等待的时间（秒）：

```php
php artisan queue:listen --sleep=5
```

你需要注意的是，队列只会在队列中没有需要执行的任务时才会进行睡眠。如果队列中有多个可执行的任务，那么队列会持续的进行任务的执行，而不会进行睡眠操作。

**执行队列的首个任务**

你可以使用 `queue:work` 命令来只执行队列的首个任务：

```php
php artisan queue:work
```

### Supervisor 配置

Supervisor 是 Linux 操作系统的一个进程监控器，并且它可以自动的在 `queue:listen` 或者 `queue:work` 命令失败时进行重启。你可以使用下面的命令在 Ubuntu 中安装 Supervisor:

```php
sudo apt-get install supervisor
```

Supervisor 配置文件通常都存储在 `/etc/supervisor/conf.d` 目录中。在这个目录中，你可以创建任意数量的配置文件来指导 supervisor 来管理监控进程。比如，让我们创建一个 `laravel-worker.conf` 文件来开始和监控 `queue:work` 进程：

```php
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command= php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --daemon
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```

上面的例子中，`numprocs` 会指导 Supervisor 运行 8 个 `queue:work` 进程并且对其进行监控，它会自动的在其失败时进行重启。当然，你可以修改命令的 `queue:work sqs` 部分来使用你所期望的队列驱动。

一旦配置文件创建完成，你可以使用下面的命令来更新 Supervisor 的配置文件和启动进程：

```php
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start laravel-worker:*
```

关于更多的 Supervisor 的配置信息，请参考 [Supervisor documentation](http://supervisord.org/index.html)。另外，你也可以使用 [Laravel Forge](https://forge.laravel.com/) 来从一个方便的 Web 界面自动的配置和管理你的 Supervisor 配置。

### 队列监控器守护进程

`queue:work` Artisan 命令包含了一个 `--daemon` 选项来强迫队列工作持续的执行任务而不重新引导框架。这相比较 `queue:lsten` 命令来说会显著的减少 CPU 的消耗：

```php
php artisan queue:work connection-name --daemon

php artisan queue:work connection-name --daemon --sleep=3

php artisan queue:work connection-name --daemon --sleep=3 --tries=3
```

就如你所看到的，`queue:work` 任务支持和 `queue:listen` 差不多的选项。你可以通过使用 `php artisan help queue:work` 命令来显示可用选项。

**对于守护进程的编码注意事项**

队列任务的守护进程不会在每个任务执行之前重新引导框架，所以，你应该注意在你的任务完成时清除一些比较重的资源消耗。比如，如果你使用 GD 类库来处理图片，你应该在任务执行完成后使用 `imagedestroy` 来将其从内存中进行释放。

### 部署具有守护进程的队列监听器

因为队列工作的守护进程是一个常驻进程。它不会再你的代码改变时进行重启。所以，你应该使用部署脚本来在代码变更时重新部署使用守护进程的队列工作：

```php
php artisan queue:restart
```

该命令会优雅的指导队列完成当前的任务后死亡，所以不会存在任务的遗漏。你应该注意的是，当你执行 `queue:restart` 命令时，队列工作就会死亡，所以你应该使用一种进程管理器，比如 Supervisor，你可以配置它来自动的重启队列工作。

> 注意：该命令依赖于缓存系统来制定重启时间表。默认的 APCu 不支持在 CLI 中运行任务。如果你使用 APCu，你应该添加 `apc.enable_cli=1` 到你的 APCu 配置中。

## 与失败的任务进行交互

由于很多事情并不能如计划中的那样进行，有时候队列任务的执行可能会失败，不要担心，它发生对我们来说是最好的！laravel 包含了一种便捷的方式来指定任务应该重复尝试的次数。如果任务被重复执行到指定的次数，它就会被记录到 `failed_jobs` 表中。这个表的名字你可以通过 `config/queue.php` 配置文件进行定制。

你可以使用 `queue:failed-table` 命令来生成一个 `failed_jobs` 迁移表：

```php
php artisan queue:failed-table
```

当你运行队列监听器时，你可以使用 `--tries` 选项来指定任务的最大尝试次数：

```php
php artisan queue:listen connection-name --tries=3
```

### 失败任务事件

如果你希望在队列任务失败时执行某些操作，你可以使用 `Queue::failing` 方法来注册一个监听事件。这个事件对通过 email 或者 [HipChat](https://www.hipchat.com/) 通知你的团队提供了一个很好的机会。比如，你可以在 `AppServiceProvider` 在事件中附加一个回调：

```php
<?php

namespace App\Providers;

use Queue;
use Illuminate\Queue\Events\JobFailed;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
  /**
   * Bootstrap any application service.
   *
   * @return void
   */
   public function boot()
   {
     Queue::failing(function (JobFailed $event) {
       // $event-connectionName
       // $event->job
       // $event-data
     });
   }

   /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

**任务类中的失败方法**

你可以在任务类中定义一个 `failed` 方法来进行更为精细的控制，这允许你在任务出现失败时来执行一些指定的动作：

```php
<?php

namespace App\Jobs;

use App\Jobs\Job;
use Illuminate\Contracts\Mail\Mailer;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendReminderEmail extends Job implements ShouldQueue
{
  use InteractsWithQueue, SerializesModels;

  /**
   * Execute the job.
   *
   * @param Mailer $mailer
   * @return void
   */
   public function handle(Mailer $mailer)
   {
     //
   }

   /**
    * Handle a job failure.
    *
    * @return void
    */
    public function failed()
    {
      // Called when the job is failing...
    }
}
```

### 重新执行失败的任务

你可以使用 `queue:failed` Artisan 命令来显示数据库 `failed_jobs` 表中失败的任务：

```php
php artisan queue:failed
```

`queue:failed` 命令会列出任务的 ID，连接，队列，和失败时间。任务 ID 可以用来进行失败任务的尝试。比如，你可以尝试重新执行 ID 为 5 的任务：

```php
php artisan queue:retry 5
```

你可以使用 `queue:retry all` 命令来重启所有的任务：

```php
php artisan queue:retry all
```

如果你希望删除失败的任务，你可以使用 `queue:forget` 命令：

```php
php artisan queue:forget 5
```

你可以使用 `queue:flush` 命令来清除所有失败的任务：

```php
php artisan queue:flush
```