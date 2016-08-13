title: 邮箱有效性验证模块的设计
date: 2016-07-20 10:17:42
tags: [php, laravel]
---

# 邮箱验证设计

在应用开发中，邮件验证是一个常用的功能，它要求用户对其邮箱地址的有效性进行验证，一般流程为：
- 用户在注册时填写邮箱地址
- 在注册成功时，应用自动发送一封确认邮箱有效性的邮件，在邮件中附上效验地址 
- 用户访问效验地址以验证邮箱有效性

## 功能需求

- 用户注册时对用户的邮箱地址发送效验邮件
- 需要对用户和邮箱以及校验码进行映射以验证用户邮箱的有效性

## 功能分析

- 应该具有邮件发送的功能
- 效验的邮件应该具有时效性
- 需要一张邮箱效验的表，用于存储用户和邮箱及校验码

## 数据库设计

```
`mail_verifies`: 
  - integer `id` increments
  - integer `user_id`
  - string 'mail'
  - string `code`
  - boolean `expired` default:false
  - datetime `created_at`
  - datetime `updated_at`
```

## 职责分析

我们分析整个流程应该不难发现，整个流程突出几个职责：
- 唯一效验码的生成
- 发邮件
- 对效验码进行验证

我们将生成校验码和对校验码进行验证的职责封装到 MailVerifier 类中，发邮件就直接使用 Laravel 中的邮件服务

## 编码设计

我们先设计用于效验邮件的路由：

```php
Route::get('mail-verification/{userId}/{code}', 'MailController@verify');
```

我们需要在用户对象中设计一个发送验证邮件的方法：

```php
Class User {

  public function sendEmailVerification (Mailer $mailer, MailVerifier $mailVerifier) {
    if ($this->eamil_verified) {
      return false;
    }

    $data = [
    'user' => $this, 
    'code' => $mailVerifier->create($this->id, $this->email)->code
    ];
    $mailer->send('mail-verifier', $data, function ($message) {
      // ...
    });
  }

  public function setMailVerified () {
    $this->email_verified = true;
    $this->save();
  }
}
```

我们需要一个验证器类来进行校验码的生成和对效验码进行验证:

```php
use Carbon\Carbon;

class MailVerifier {
  protected $mailVerify;

  public function __construct(MailVerify $mailVerify) {
    $this->$mailVerify = $mailVerify 
  }

  public function create ($userId, $mail) {
    $this->mailVerify->expiredAll($userId);
    return $this->mailVerify->create([
            'user_id' => $userId,
            'mail' => $mail,
            'code' => $this->generateCode($userId)
          ]);
  } 

  public function generateCode($userId) {
    return md5(time() . $userId . str_pod(rand(0, 999999), 6, '0'));
  }

  public function verify ($mail, $code) {
    $finder = $this->mailVerify->where([
      'mail' => $mail,
      'code' => $code,
      'expired' => false
      ])->where('created_at', '>', Carbon::now()->subMinute(10))->first();

    if ($finder) {
      $finder->setExpired();
    }

    return !!$finder;
  }
}
```

接着就是控制器了：

```php
class MailController extends Controller
{
  protected $mailVerifier;
  protected $user;

  pbulic function __construct(User $user, MailVerifier $mailVerifier) {
    $this->user = $user;
    $this->mailVerifier = $mailVerifier; 
  }

  public function verify ($userId, $code) {
    $user = $this->uesr->find($userId);

    if (!$user) {
      return redirect('404');
    }

    $result = $this->mailVerifier->verify($user->eamil, $code);
    if ($result) {
      $user->setMailVerified();
      return view('mail-verified');
    }
    return redirect('404');

  }
}
```