title: laravel 基础教程 —— Artisan 命令
date: 2016-05-27 09:07:43
tags: [php, laravel]
---

# Artisan 控制台

## 简介

Artisan 是 laravel 自带的命令行工具接口的名称。它为应用的开发提供了多种有用的命令工具。Artisan 的底层驱动是强大的 Symfony 控制台组件。你可以使用 `list` 命令来查看可用的 Artisan 命令：

```php
php artisan list
```

所有的命令都提供了帮助文档，你可以在相应的命令前使用 `help` 来查看相关命令的选项和参数：

```php
php artisan help migrate
```

## 编写命令

除了 Artisan 自带的命令之外，laravel 也允许你自行定义自己的命令工具，你可以将自定义的命令工具存放到 `app/Console/Commands` 目录下，你当然也可以存放在其他任何想要存放的目录，只要你所存放的位置能基于 `composer.json` 的设置进行自动加载就行。

你可以使用 `make:console` Artisan 命令，来进行新命令工具的生成，这个命令会生成一个命令的样本来帮助你开始：

```php
php artisan make:console SendEmails
```

上面的命令会生成一个 `SendEmails` 类，并存放在 `app/Console/Commands/SendEmails.php`，当你构建命令时，你可以使用 `--command` 参数来设置命令工具所对应的名称。

```php
php artisan make:console SendEmails --command=emails:send
```

### 命令结构

一旦你的命令被生成，你需要在其中填充 `signature` 和 `description` 属性。这些内容会在你使用 `list` 命令时在屏幕上显示。

当你的命令被执行时，会触发 `handle` 方法，你可以在这个方法里编写相应的命令逻辑。让我们来看一个命令示例。

你应该知道我们可以在命令类的构造函数中进行依赖的注入。laravel 服务容器会自动的注入所有在构造函数中使用类型提示的依赖。为了使代码有更好的重用性，保持控制台命令的轻量，让它们推迟到应用服务来完成具体的任务是一种很好的实践。

```php
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
  /**
   * The name and signature of the console command.
   *
   * @var string
   */
   protected $signature = 'email:send {user}';

   /**
    * The console command descritpion
    * 
    * @var string
    */
    protected $description = 'Send drip e-mails to a user';

    /**
     * The drip e-mail service.
     *
     * @var DripEmailer
     */
     protected $drip;

     /**
      * Create a new command instance.
      *
      * @param DripEmailer $drip
      * @return void
      */
      public function __construct(DripEmailer $drip)
      {
        parent::__construct();

        $this->drip = $drip;
      }

      /**
       * Execute the console command.
       * 
       * @return mixed
       */
       public function handle()
       {
         $this->drip->send(User::find($this->argument('user')));
       }

}
```

## 命令 I/O

### 定义期望的输入

当我们编写命令行工具时，通常都是通过用户输入的参数或者选项来收集输入。laravel 使这一切变的非常简单。你可以使用 `signature` 属性在你的命令中定义你所期望得到的输入名称。`signature` 属性允许你使用一行富有表现力类路由的语法来定义命令行中的名称，参数和选项。

所有用户所提供的参数和选项都被包裹在大括号内。在下面的示例中，命令行工具定义了一个必须的参数：`user`：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
 protected $signature = 'email:send {user}'
```

你也可以使参数可选，或者为一个可选的参数定义一个默认值：

```php
// Optional argument...
email:send {user?}

// Optional argument with default value...
email:send {user=foo}
```

选项和参数一样，它们也是用户的一种输入。但是它们在命令行被指定时会使用 `--` 作为前缀。我们可以像下面这样在 signature 中定义选项：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
 protected $signature = 'email:send {user} {--queeue}';
```

在这个示例中，`--queue` 可以在使用 Artisan 命令时被指定。如果指定了 `--queque`，那么这个选项的值将会为 `true`。否则选项的值将为 `false`：

```php
php artisan emial:send 1 --queue
```

你也可以通过在选项后面添加 `=` 号来表明这个选项是需要通过用户输入的：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
 protected $signature = 'email:send {user} {--queue=}';
```

在这个例子中，用户传递一个值到选项像下面这样：

```php
php artisan email:send 1 --queue=default
```

你也可以分配一个默认值到选项：

```php
email:send {user} {--queue=default}
```

你也可以为选项定义一个简写，你需要使用 `|` 来分割简写与选项名称，并将简写像下面的示例一样前置：

```php
email:send {user} {--Q|queue}
```

如果你想要定义参数或选项期望得到的是输入数组，你可以使用 `*` 通配符：

```php
email:send {user*}
email:send {user} {--id=*}
```

**输入说明**

你也可以分配描述信息到选项或者参数中，你需要使用 `:` 来进行分割描述和选项或参数：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
 protected $signature = 'email:send
                         {user: The ID of the user}
                         {--queue= : Whether the job should be queued}';
```

### 检索输入

当你的命令工具存在时，你明显会需要访问命令行中所期望得到的参数或选项的值。你可以使用 `argument` 和 `option` 方法来得到：

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
 public function handle()
 {
   $userId = $this->argument('user');
 }
```

你可以通过调用不带参数的 `argument` 方法来获取所有参数所组成的数组：

```php
$arguments = $this->argument();
```

选项可以非常简单的类似参数一样的通过 `option` 方法进行检索。选项可以像参数那样当调用无参数的 `option` 方法时返回所有选项所组成的数组：

```php
// Retrieve a specific option...
$queueName = $this->option('queue');

// Retrieve all options...
$options = $this->option();
```

如果相应的参数或选项没有被检索到，会返回 `null`。

### 为输入进行提示

除了显示输出之外，你可能也需要在执行你的命令行工具的过程中向用户索求额外的输入。你可以使用 `ask` 方法来进行提示用户输入，这个方法将接收用户的输入并返回输入的内容：

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
 public function handle()
 {
   $name = $this->ask('What is your name?');
 }
```

`secret` 方法与 `ask` 方法非常相似，只是用户在控制台中输入的内容并不是可见的。这个方法通常是询问用户密码相关时使用的：

```php
$password = $this->secret('What is the password?');
```

**要求确认**

如果你只是需要用户的确认，你可以使用 `confirm` 方法。默认的，该方法返回 `false`. 但是如果用户在响应中输入了 `y`，该方法将返回 `true`.

```php
if ($this->confirm('Do you wish to continue? [y|N]')) {
  //
}
```

**给用户一个选择**

`anticipate` 方法可以用来对可选的内容进行自动完成的提示。这里只是对用户有可能选择的内容进行自动完成提示，并非强制要求用户仅选择可选的内容:

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

如果你需要给用户预置选项你可以使用 `choice` 方法。用户必须选中选项中的索引，用户选中相应的索引的答案的值将会被返回。你可以设置一个默认的索引值，这个索引值将在用户没有做出任何选择时返回：

```php
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);
```

### 编写输出

你可以使用 `line`，`info`，`comment`，`question` 和 `error` 方法发送输出到控制台。这些方法会使用相应的 ANSI 颜色来表明相应的目的。

你可以使用 `info` 方法来向用户显示一个信息消息。通常，这条消息在控制台中是一个绿色的文本：

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
 public function handle()
 {
   $this->info('Display this on the screen');
 }
```

你可以使用 `error` 方法来显示一个错误消息。错误消息通常都是红色的：

```php
$this->error('Something went wrong!');
```

 你可以使用 `line` 方法来显示一个原生的控制台输出。`line` 方法并没有对消息设置任何的独特颜色信息：

 ```php
$this->line('Display thie on the screen');
 ```

 **表格布局**

 你可以使用 `table` 方法来简单的对多行或多列的数据进行格式化布局。你只需要向方法中传递头部和行信息到方法中就可以了。宽度和高度将会自动的通过所给定的数据进行计算：

 ```php
 $headers = ['Name', 'Email'];

 $users = App\User::all(['name', 'email'])->toArray();

 $this->table($headers, $users);
 ```

 **进度条**

对于一些耗时的任务来说，有一个进度提示是非常有用的。如果使用输出对象，我们就可以开始，推进和停止进度条。你需要在你开始进度条之前定义步长。然后根据进行的每一步来推进进度条：

```php
$user = App\User::all();

$bar = $this->output->createProgressBar(count($users));

foreach ($users as $user) {
  $this->performTask($user);

  $bar->advance();
}
$bar->finish();
```

你可以通过查看 [Symfony Progress Bar component documentation](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html) 来获取更多的选项信息。

## 注册命令行

一旦你完成了命令行的编写，你还需要注册其在 Artisan 命令中可用。这些需要在 `app/Console/Kernel.php` 文件中完成。

在这个文件中，你会发现 `commands` 属性，它是一个命令行类的列表。当 Artisan 启动时，所有在这个列表中的命令都会通过服务容器解析到 Arisan:

```php
protected $commands = [
  Commands\SendEmails::class
];
```

## 通过代码调用命令

有时候你可能希望在控制台之外执行 Artisan 命令。比如，你希望在控制器的路由中触发 Artisan 命令。你可以使用 `Artisan` 假面的 `call` 方法来完成这些。`call` 方法接收一个命令名称，和一个包含所有参数和选项所组成的数组，命令执行完成之后会返回一个退出代码：

```php
Route::get('/foo', function () {
  $exitCode = Artisan::call('email:send', [
    'user' => 1, '--queue' => 'default'
  ]); 

  //
});
```

通过使用 `Artisan` 假面的 `queue` 方法，你甚至可以队列化 `Artisan` 命令，在后台进程中队列工人会按序的帮你执行完成命令：

```php
Route::get('/foo', function () {
  Artisan::queue('email:send', [
    'user' => 1, '--queue' => 'default'
  ]);

  //
})
```

如果你需要强制指定一个不接受字符串值的选项的值为一个字符串，就像 `migrate:refresh` 命令，你可以使用 `--force` 标识并传递一个布尔值：

```php
$exitCode = Artisan::call('migrate:refresh', [
  '--force' => true,
]);
```

### 在命令行中调用另外的命令

有时候，你可能希望在命令行工具中调用另外一个已经存在的 Artisan 命令。你可以使用 `call` 方法来完成这些。`call` 方法接收命令的名称和一个包含所有参数和选项的数组：

```php
/**
 * Execute the console command.
 *
 * @return mixed
 */
 public function handle()
 {
   $this->call('email:send', [
    'user' => 1, '--queue' => 'default'
   ]);

   //
 }
```

如果你希望调用另外一个控制台命令而不希望它有任何的输出，你可以使用 `callSilent` 方法，`callSilent` 方法具有 `call` 方法相同的调用方式：

```php
$this->callSilent('email:send', [
  'user' => 1, '--queue' => 'default'
]);
```

