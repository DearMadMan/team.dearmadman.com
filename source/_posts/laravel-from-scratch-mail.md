title: laravel 基础教程 —— 邮件
date: 2016-06-08 18:24:27
tags: [php, laravel]
---

# 邮件

## 简介

laravel 基于流行的 [SwiftMailer](http://swiftmailer.org/) 类库构建了一种干净简洁的邮件 API。laravel 提供了驱动以支持 SMTP，Mandrill，SparkPost，Amazon SES，PHP 的 `mail` 方法和 `sendmail` 方法，这允许你快速的开始构建通过本地或云服务发送邮件的功能。

### 驱动先决条件

基于 API 的驱动程序如 Mailgun 和 Mandrill 通常要比 SMTP 服务简单快速。所有的 API 驱动都必须引入 Guzzle HTTP 类库，你可以通过在 `composer.json` 文件中添加下面的内容来进行安装：

```php
"guzzlehttp/guzzle": "~5.3|~6.0"
```

**Mailgun 驱动**

如果使用 Mailgun 驱动，你首先需要安装 Guzzle，然后设置 `config/mail.php` 配置文件的 `driver` 选项为 `mailgun`。然后，你需要确认你的 `config/services.php` 配置文件中包含以下选项：

```php
'mailgun' => [
  'domain' => 'your-mailgun-domain',
  'secret' => 'your-mailgun-key',
],
```

**Mandrill 驱动**

如果使用 Mandrill 驱动，你首先需要安装 Guzzle，然后设置 `config/mail.php` 配置文件的 `driver` 选项为 `mandrill`。接着，你需要确认你的 `config/services.php` 配置文件中包含以下选项：

```php
'mandrill' => [
  'secret' => 'your-mandrill-key',
],
```

**SparkPost 驱动**

如果使用 SparkPost 驱动，你首先需要安装 Guzzle，然后设置你的 `config/mail.php` 配置文件中的 `driver` 选项为 `sparkpost`。接着，你需要确认你的 `config/services.php` 配置文件中包含以下选项：

```php
'sparkpost' => [
  'secret' => 'your-sparkpost-key',
],
```

**SES 驱动**

如果使用 Amazon SES 驱动，你需要在 PHP 中引入 Amazon AWS SDK。你可以通过在 `composer.json` 文件中进行引入安装：

```php
"aws/aws-sdk-php": "~3.0"
```

然后设置你的 `config/mail.php` 配置文件的 `driver` 选项为 `ses`。接着，你需要确认你的 `config/services.php` 配置文件中包含以下选项：

```php
'ses' => [
  'key' => 'your-ses-key',
  'secret' => 'your-ses-secret',
  'region' => 'ses-region', //e.g. us-east-1
],
```

## 发送邮件

laravel 允许你使用视图来存储你的邮件信息。比如，你可以在 `resources/views` 目录下创建一个 `emails` 目录来管理你的邮件视图：

你需要使用 `Mail` 假面的 `send` 方法来发送一个邮件。`send` 方法接收三个参数，第一个参数应该是视图的名称。第二个参数应该是传递到视图中的数据所组成的数组。最后，是一个 `Closure` 回调函数，这个回调函数会接收一个 message 实例，这允许你自由的配置收件人，主题，和其他方面的邮件信息：

```php
<?php

namespace App\Http\Controller;

use Mail;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Send an e-mail reminder to the user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
   public function sendEmailReminder(Request $request, $id)
   {
     $user = User::findOrFail($id);

     Mail::send('emails.reminder', ['user' => $user], function ($m) use ($user) {
       $m->from('hello@app.com', 'Your Application');

       $m->to($user->email, $user->name)->subject('Your Reminder!');
     });
   }
}
```

因为上面的示例中我们传递了 `user` 键到数组中，所以我们可以在我们的邮件视图中使用用户的名称：

```php
<?php echo $user->name; ?>
```

> 注意： 一个 `$message` 变量会自动的传递到邮件视图，并且允许内联附件。所以，你应该避免手动的传递到视图 `message` 变量。

**构建一个 Message**

就如我们上面谈到的，传递到 `send` 方法的第三个参数是一个 `Closure`，这个闭包允许你设置邮件信息的各种选项。使用闭包，你可以指定信息的其它属性，比如复写，密抄等：

```php
Mail::send('emails.welcome', $data, function ($message) {
  $message->from('us@example.com', 'Laravel');

  $message->to('foo@example.com')->cc('bar@example.com');
});
```
这里列出了 `$message` 构建实例所有可用的方法：

```php
$message->from($address, $name = null);
$message->sender($address, $name = null);
$message->to($address, $name = null);
$message->cc($address, $name = null);
$message->bcc($address, $name = null);
$message->replyTo($address, $name = null);
$message->subject($subject);
$message->priority($level);
$message->attach($pathToFile, array $options = []);

// Attach a file from a raw $data string...
$message->attachData($data, $name, array $options = []);

// Get the underlying SwiftMailer message instance...
$message->getSwiftMessage();
```

> 注意：message 实例继承自 SwiftMailer 信息类并会被传递到 `Mail::send` 闭包中。这允许你在访问所有 SwiftMailer 信息类的方法。

**纯文本邮件**

默认的，`send` 方法会假设传递到其中的视图包含了 HTML。但是，你可以传递一个数组到 `send` 方法的第一个参数，你指定一个纯文本视图来合并 HTML 视图：

```php
Mail::send(['html.view', 'text.view'], $data, $callback);
```

或者你可以只传递一个纯文本的邮件，你需要指定数组的键为 `text`:

```php
Mail::send(['text' => 'view'], $data, $callback);
```

**原始字符串邮件**

你可以使用 `raw` 方法来直接发送原始字符串邮件：

```php
Mail::raw('Text to e-mail', function ($message){
  //
});
```

### 附件

你可以使用传递到闭包中的 `$message` 对象的 `attach` 方法来在邮件中添加附件。`attach` 方法接收一个完整的文件路径作为第一个参数：

```php
Mail::send('emails.welcome', $data, function ($message) {
  //

  $message->attach($pathToFile); 
});
```

当你添加一个邮件附件时，你可能也需要指定附件显示的名称或者 MIME type。你可以向 `attach` 方法传递一个数组作为第二个参数：

```php
$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);
```

`attachData` 方法可以用来添加一个原始字节字符串作为一个附件。比如，你可能需要使用这个方法来将内存中生成的 PDF 直接添加到附件中，而不需要先存储到磁盘：

```php
$message->attachData($pdf, 'invoice.pdf');

$message->attachData($pdf, 'invoice.pdf', ['mime' => $mime]);
```

### 内联附件

**在邮件视图中嵌入一个图片**

在邮件中内联一个图片通常是比较笨重的。但是，laravel 提供了一种便捷的方式在你的邮件中附加一个图片并且取回适当的 CID。你需要在邮件视图中使用 `$message` 变量的 `embed` 方法。你应该记得，Laravel 会自动的创建一个 `$message` 变量并将其传递到你的邮件视图中：

```php
<body>
  Here is an image:

  <img src="<?php echo $message->embed($pathToFile); ?>"
</body>
```

**嵌入原始数据到视图**

如果你希望嵌入一个原始数据到邮件信息中，你可以使用 `$message` 变量的 `embedData` 方法：

```php
<body>
  Here is an image from raw data:

  <img src="<?php echo $message->embedData($data, $name); ?>"
</body>
```

### 邮件队列

**队列化邮件信息**

由于发送邮件是非常消耗资源的一件事，这样会影响到应用的响应时间。所以很多开发者都选择使用队列来完成异步的邮件发送。Laravel 使用统一的队列接口来使这些非常简单。你需要使用 `Mail` 假面的 `queue` 方法来使邮件队列化：

```php
Mail::queue('emails.welcome', $data, function ($message) {
  // 
});
```

该方法会自动的在队列添加发送邮件的任务，该任务会在后台自动的执行。当然，你需要先配置好队列。

**延迟消息队列**

如果你希望延迟执行邮件的发送队列。你需要使用 `later` 方法。你需要在方法的第一个参数传送一个你希望延迟执行的秒数：

```php
Mail::later(5, 'emails.welcome', $data, function ($message) {
  //
});
```

**添加到指定的队列**

如果你希望添加邮件任务到指定的队列，你需要使用 `queueOn` 和 `laterOn` 方法：

```php
Mail::queueOn('queue-name', 'emails.welcome', $data, function ($message) {
  // 
});

Mail::laterOn('queue-name', 5, 'emails.welcome', $data, function ($message) {
  // 
});
```

## 邮件 & 本地开发

当开发一个发送邮件的服务时，你可能并不希望真实的去发送一个邮件。只需要模拟测试成功就可以了。laravel 提供了多种方式来禁用真实的邮件发送。

**log 驱动**

其中一个解决方案就是在本地开发时使用 `log` 邮件驱动。这个驱动会将所有的邮件信息写入到日志文件中。

**通用的邮件**

另外一个解决方案就是设置一个通用的邮件来接收所有的应用发出的邮件。这种方式，会使应用生成的所有邮件都发送到这个地址。你可以通过配置 `config/mail.php` 文件的 `to` 选项来进行设置：

```php
'to' => [
  'address' => 'dev@domain.com',
  'name' => 'Dev Example'
],
```

**mailtrap**

最后，你也可以使用一些像 [Mailtrap](https://mailtrap.io/) 和 `smtp` 驱动的服务来发送你的邮件到一个虚拟邮箱，而且你也可以通过真实的邮件客户端进行查看。这可以使你看到真实的邮件在 Mailtrap 中的显示效果。

## 事件

laravel 会在发送邮件之前触发事件。你应该注意的是，它是在邮件发送之前触发事件，而不是添加到队列时。你可以在 `EventServiceProvider` 中注册一个事件监听者：

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
 protected $listen = [
  'Illuminate\Mail\Events\MessageSending' => [
    'App\Listeners\LogSentMessage',
  ],
 ];
```