title: laravel 基础教程 —— SSH 任务
date: 2016-06-17 16:02:39
tags: [php, laravel]
---

# Envoy 任务运行器

## 简介

Laravel [Envoy](https://github.com/laravel/envoy) 为远端服务器常用任务的定义与执行提供了迷你简洁的语法。你可以通过使用 Blade 语法样式轻松的为部署，Artisan 命令等设置任务。目前，Envoy 只支持 Mac 和 Linux 操作系统。

### 安装

首先，你需要通过 Composer 的 `global` 命令来安装 Envoy：

```php
composer global require "laravel/envoy=~1.0"
```

你需要确保 `~/.composer/vendor/bin` 目录被加入到你的 PATH 中，这样才能使你在使用终端时可以直接使用 `envoy` 命令。

**更新 Envoy**

你可以使用 Composer 来维持 Envoy 的更新：

```php
composer global update
```

## 编写任务

所有的 Envoy 任务应该被定义在你项目的根目录下的 `Envoy.blade.php` 文件中。这里有一个简单的示例：

```php
@servers(['web' => 'user@192.168.1.1'])

@task('foo', ['on' => 'web'])
  ls -al
@endtask
```

就如你所看到的，`@servers` 指令被定义在文件的头部，并且包含一个数组，数组中包含服务器的列表。`@task` 指令用来定义任务，它包含一个任务名称，和一个数组参数，数组中包含一个 `on` 键，它的值就是任务所要执行的服务器，它应该是 `@servers` 指令列表中的一个或多个。你应该在 `@task` 指令的内部放置 Bash 代码，这些代码会在任务执行时传递给所要执行的远端服务器。

**本地任务**

你可以指定服务器为本地来执行本地的任务：

```php
@servers(['localhost' => '127.0.0.1'])
```

**引导**

有时候，你可能希望在执行 Envoy 任务之前先执行某些 PHP 操作。你可以使用 `@setup` 指令来声明变量，并且你可以在其内部使用 PHP 来工作：

```php
@setup
  $now = new DateTime();

  $environment = isset($env) ? $env : "testing";
@endsetup
```

你也可以使用 `@include` 指令来引入任意的外部 PHP 文件：

```php
@include('vendor/autoload.php')
```

**确认任务**

如果你希望在远端服务器执行所给定任务之前先进行提示，你可以在你的任务定义时添加 `confirm` 指令：

```php
@task('deploy', ['on' => 'web', 'confirm' => true])
  cd site
  git pull origin {{ $branch }}
  php artisan migrate
@endtask
```

### 任务变量

如果你需要的话，你可以使用命令行开关来传递变量到 Envoy 任务中，这允许你定制化你的任务：

```php
envoy run deploy --branch=master
```

你可以在你的任务中通过 Blade 的 echo 语法使用该选项:

```php
@servers(['web' => '192.168.1.1'])

@task('deploy', ['on' => 'web'])
  cd site
  git pull origin {{ $branch }}
  php artisan migrate
@endtask
```

### 多个服务器

你可以轻松的跨多个服务器执行任务。首先，你需要在 `@servers` 指令中添加额外的服务器。每个服务器应该被分配一个唯一的名字。当你添加完额外的服务器之后，你需要在待执行的任务指令中使用数组 `on` 键来列出待执行的服务器：

```php
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
  cd site
  git pull origin {{ $branch }}
  php artisan migrate
```

默认的，任务会在服务器间串行执行，这意味着只有在当前服务器执行任务完成之后才会执行下一个服务器的任务。

**平行执行**

如果你希望跨服务器平行执行任务。你可以在任务指令中添加 `parallel` 选项：

```php
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
  cd site
  git pull origin {{ $branch }}
  php artisan migrate
@endtask
```

### 任务宏

任务宏允许你定义一个命令来顺序的执行一组任务。举个实例，我们定义一个 `deploy` 宏来执行 `git` 和 `composer` 任务：

```php
@servers(['web' => '192.168.1.1'])

@macro('deploy')
  git
  composer
@endmacro

@task('git')
  git pull origin master
@endtask

@task('composer')
  composer install
@endtask
```

一旦你定义完成了宏，你就可以通过一条命令来运行多个任务：

```php
envoy run deploy
```

### 运行任务

你需要使用 Envoy 的 `run` 命令来执行 `Envoy.blade.php` 文件中所定义的任务。你可以传递一个任务的名称或者宏名称到命令中。Envoy 会执行任务并同步显示服务器执行的输出：

```php
envoy run task
```

## 通知

### HipChat

你可以使用 `@hipchat` 指令来在任务执行完成之后，发送一个消息通知到团队的 HipChat 房间中。这个指令接收一个 API token，房间的名称和消息中所显示的发送者的用户名:

```php
@servers(['web' => '192.168.1.1'])

@task('foo', ['on' => 'web'])
  ls -al
@endtask

@after
  @hipchat('token', 'room', 'Envoy')
@endafter
```

如果你需要，你也可以发送自定义的消息到 HipChat 房间。构建消息时，任务可用的变量在消息中也是可用的：

```php
@after
  @hipchat('token', 'room', 'Envoy', "$task ran in the $env environment.")
@endafter
```

### Slack

除了 HipChat 之外，Envoy 也支持向 [Slack](https://github.com/laravel/envoy) 中发送通知。`@slack` 指令接收一个 Slack hook URL，一个频道名称，和你需要发送的消息内容：

```php
@after
  @slack('hook', 'channel', 'message')
@endafter
```

你可以通过在 Slack 的网站上创建一个 `Incoming WebHooks` 来获取 webhook URL。`hook` 参数应该是一个完整的 webhook URL，比如：

```php
https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX
```

你可以提供以下作为频道的参数之一：
- `#channel` 发送通知到频道
- `@user` 发送通知到用户