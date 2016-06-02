title: laravel 基础教程 —— 计费
date: 2016-05-27 16:28:28
tags: [php, laravel]
---

# laravel 会计员（Cashier）

## 简介

Laravel Cashier 提供了一种表现流利的接口来支持 [Stripe](https://stripe.com/) 和 [Braintree](https://braintreepayments.com/) 的计费服务。它封装了许多你畏惧编写的认购计费样板。除了基本的认购管理，Cashier 还可以处理优惠券、服务升级、服务替换、认购份额、取消并保留期限内可用、甚至是生成 PDF 类型的发票。

## Stripe 配置

**Composer**

首先，添加 Strpe 的 Cashier 包到你的 `composer.json` 文件，然后运行 `composer update` 命令：

```php
"laravel/cashier": "~6.0"
```

**服务提供者**

然后在 `app` 配置文件中注册 `Laravel\Cashier\CashierServiceProvider` 服务提供者。

**数据库迁移**

在使用 Cashier 之前，我们也需要去准备一下数据库。我们需要在 `users` 表中添加几列，并且创建一个新的 `subscriptions` 表来保留客户的认购：

```php
Schema::table('users', function ($table) {
  $table->string('stripe_id')->nullable();
  $table->string('card_brand')->nullable();
  $table->string('card_lost_four')->nullable();
  $table->timestamp('trial_ends_at')->nullable();
});

Schema::create('subscriptions', function ($table) {
  $table->increments('id');
  $table->integer('user_id') ;
  $table->string('name');
  $table->string('stripe_id');
  $table->string('stripe_plan');
  $table->integer('quantity');
  $table->timestamp('trial_ends_at')->nullable();
  $table->timestamp('ends_at')->nullable();
  $table->timestamps();
});
```

一旦迁移表创建完毕，你可以通过 artisan 命令 `migrate` 进行迁移操作。

**模型起步**

接着，你需要在你的模型定义中添加 `Billable` trait:

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable 
{
  use Billable;
}
```

**提供秘钥**

然后你需要在 `services.php` 配置文件中对你的 Stripe 进行配置：

```php
'stripe' => [
  'model' => App\User::class,
  'secret' => env('SIRIPE_SECRET'),
];
```

## Braintree 配置

**注意事项**

对于大多数的操作，Stripe 和 Braintree 在 Cashier 中的方法实现是相同的，两种服务都提供了对使用信用卡进行认购的计费支持。但是 Braintree 也支持通过 PayPal 的支付方式。Braintree 也相应的缺乏一些 Stripe 所支持的特性。你需要根据自身的需求去对比选择使用 Stripe 还是 Braintree。
- Braintree 支持 PayPal 支付，而 Stripe 不支持。
- Braintree 在认购时不支持 `increment` 和 `decrement`，这是 Braintree 自身的限制，而不是 Cashier 的局限性。
- Braintree 不支持百分比的折扣。这是 Braintree 自身的限制，而不是 Cashier 所限制的。

**Composer**

首先，你需要在你的 `composer.json` 文件中进行添加 `Cashier` 的文件包并使用 `composer update` 命令进行安装：

```php
"laravel/cashier-braintree": "~1.0"
```

**服务提供者**

接着，注册 `laravel\Cashier\CashierServiceProvider` 服务提供者到你的 `app` 配置文件。

**安排信用优惠**

在你使用 Braintree 的 Cashier 之前，你需要先在你的 Braintree 控制面板中定义一个 `plan-credit` 折扣。这个折扣会被用于正确的按比例从年度改为按月计费或者从月到年计费。你可以在控制面板中定义这个折扣为任意你所希望的值，Cashier 会在我们使用优惠时进行正确的结算。

**数据库迁移**

在使用 Cashier 之前，我们也需要预先构建好数据库。我们需要在 `users` 表中添加几列并且创建一个 `subscriptions` 表来处理所有的用户认购：

```php
Schema::table('users', function ($table) {
  $table->string('braintree_id')->nullable();
  $table->string('paypal_email')->nullable();
  $table->string('card_brand')->nullable();
  $table->string('card_last_four')->nullable();
  $table->timestamp('trial_ends_at')->nullable(); 
});

Schema::create('subscriptions', function ($table) {
  $table->increments('id');
  $table->integer('user_id');
  $table->string('name');
  $table->string('braintree_id');
  $table->string('braintree_plan');
  $table->integer('quantity');
  $table->timestamp('trial_ends_at')->nullable();
  $table->timestamp('ends_at')->nullable();
  $table->timestamps();
});
```

一旦迁移的表结构构建完成，你就可以通过 artisan 命令 `migrate` 运行迁移操作。

**模型起步**

接着，在你的模型中添加 `Billable` trait:

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
  use Billable;
}
```

**提供配置**

然后，你需要在你的 `services.php` 配置文件中进行相关配置：

```php
'braintree' => [
  'model' => App\User::class,
  'environment' => env('BRAINTREE_ENV'),
  'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
  'public_key' => env('BRAINTREE_PUBLIC_KEY'),
  'private_key' => env('BRAINTREE_PRIVATE_KEY'),
];
```

你还需要在你的 `AppServiceProvider` 服务提供者的 `boot` 方法中运行下面的 Braintree SDK 的方法：

```php
Braintree_Configuration::environment(env('BRAINTREE_ENV'));
Braintree_Configuration::merchantId(env('BRAINTREE_MERCHANT_ID'));
Braintree_Configuration::publicKey(env('BRAINTREE_PUBLIC_KEY'));
Braintree_Configuration::privateKey(env('BRAINTREE_PRIVATE_KEY'));
```

## 订阅/认购

### 创建一个认购

为了创建一个认购，你首先需要先从 billable 模型中提取一个实例，通常情况下它是 `App\User` 的一个实例。一旦你拥有了一个模型的实例，你就可以使用 `newSubscription` 方法来创建一个该模型的认购:

```php
$user = User::find(1);

$user->newSubscription('main', 'monthly')->create($creditCardToken);
```

传递到 `newSubscription` 方法的第一个参数应该是认购的名称。如果你的应用仅仅只提供一种单一功能的订阅，你或许可以定义为 `main` 或者 `primary`。而第二个参数则是用户需要认购的 Stripe / Braintree 的付费计划。这个值应该和你的 Stripe 或者 Braintree 中的计划标识相对应。

`create` 方法会自动的创建一个认购，并且同客户的 ID 和 其他账单相关的信息更新到数据库。

**其他用户详细信息**

如果你需要指定添加一些额外的用户信息，你可以在 `create` 方法中传递第二个参数：

```php
$user->newSubscription('main', 'monthly')->create($creditCardToken, [
  'email' => $email,
]);
```

 你可以查看 [Stripe](https://stripe.com/docs/api#create_customer) 和 [Braintree](https://developers.braintreepayments.com/reference/request/customer/create/php) 的文档来获取其所支持的额外字段。

**优惠**

如果你需要在创建一个认购时使用优惠，你可以使用 `withCoupon` 方法：

```php
$user->newSubscription('main', 'monthly')
     ->withCoupon('code')
     ->create($creditCardToken);
```

### 检查认购状态

一旦订阅了你的应用，你可以通过各种简单方便的方法来检查他们的认购状态。首先，你可以通过 `subscribed` 方法来检查用户是否激活了认购，即使用户是在试用期中，该方法也会返回 `true`:

```php
if ($user->subscribed('main')) {
  //
}
```

通过 `subscribed` 方法，我们可以创建一个极好的路由中间件，用来过滤只有处于订阅状态的用户可以访问路由或者控制器：

```php
public function handle($request, Closure $next) 
{
  if ($request->user() && ! $request->user()->subscribed('main')) {
    // This user is not a paying customer...
    return redirect('billing');
  }

  return $next($request);
}
```

如果你需要判断用户是不是还在试用期之中，你可以使用 `onTrial` 方法，该方法通常是用来向用户提醒其还在试用期之中：

```php
if ($user->subscription('main')->onTrial()) {
  //
}
```

`subscribedToPlan` 方法用来判断用户是否订阅了所给定的 Stripe / Braintree 计划。比如，我们需要判断用户的 `main` 认购是不是通过 `monthly` 计划来进行付费的：

```php
if ($user->subscribedToPlan('monthly', 'main')) {
  //
}
```

**取消认购状态**

你可以通过 `cancelled` 方法来判断用户是否曾经进行过认购，但是却在使用的过程中取消了继续付费订阅:

```php
if ($user->subscription('main')->cancelled()) {
  //
}
```

或许，你需要判断一个用户取消了持续订阅，但是他的订阅期还未过期。比如，一个用户在 3 月 5 号取消了持续订阅，但是他的上次付费服务还可以继续使用到 3 月 10 号。他还在其可用期内。你应该注意只要是在可用期内，不论是否取消了持续订阅或者还是在试用期内，`subscribed` 方法都会返回 `true` :

```php
if ($user->subscription('main')->onGracePeriod()) {
  //
}
```

### 改变付费计划

在用户认购了你的应用之后，有时候或许他们想要变更付费的计划。你可以使用 `swap` 方法来变更到一个新的认购计划。比如，我们可以非常简单的切换当前用户到 `premium` 认购：

```php
$user = App\User::find(1);

$user->subscription('main')->swap('provider-plan-id');
```

如果用户还处于试用期之中，那么即使切换付费计划，他们的试用期也是会被保持。还有，如果之前认购的份数存在，相应的认购的飞鼠也会被保持：
```php
$user->subscription('main')->swap('provider-plan-id');
```

如果你想要在切换付费方案的同时取消之前的试用期，你可以使用 `skipTrial` 方法：

```php
$user->subscription('main')
     ->skipTrial()
     ->swap('provider-plan-id');
```

### 认购份额

> 注意：认购份额只支持 Stripe 版本的 Cashier。Braintree 并没有 Stripe 份额的概念。

有时候认购时受份额影响的。比如，你的应用可能是按照每用户每月 $10 来进行计费的。要轻松的递增或递减认购的数量，你可以使用 `incrementQuantity` 和 `decrementQuantity` 方法：

```php
$user = User::find(1);

$user->subscription('main')->incrementQuantity();

// Add five to the subscription's current quantity...
$user->subscription('main')->incrementQuantity(5);

$user->subscription('main')->decrementQuantity();

// Subtract five to the subscription's current quantity...

$user->subscription('main')->decrementQuantity(5)
```

另外，你也可以通过使用 `updateQuantity` 放法来设置一个指定的份额：

```php
$user->subscription('main')->updateQuantity(10);
```

关于更多的认购份额信息，请查看 [Strpe documentation](https://stripe.com/docs/guides/subscriptions#setting-quantities)。

### 认购税率

在 Cashier 中，可以非常简单的提供 `tax_percent` 值到 Stripe / Braintree。为了指定用户支付认购时的税率百分比，你需要实现 `taxPercentage` 方法到你的 billable 模型中，并且在该方法中返回一个 0 - 100 的值，该值不应该多于 2 个小数:

```php
public function taxPercentage() {
  return 20;
}
```
这使你可以应用在多种模型间计算不同的税率,这意味着你可以跨越多个国家的用户群来设置结算税率。

### 取消订阅

你可以简单的调用 `cancel` 方法来取消用户订阅：

```php
$user->subscription('main')->cancel();
```

当认购被取消时，Cashier 会自动的设置 `ends_at` 列到数据库中。该列会被用来了解当调用 `subscribed` 方法时何时应该返回 `false`。比如，如果用户在 3 月 1 号取消了认购，但是上一次认购期应该是在 3 月 5 号结束，所以， `subscribed` 方法会持续返回 `true` 直到 3 月 5 号。

你可以通过 `onGracePeriod` 方法来判断一个已经取消订阅的用户是否还在其可用期内：

```php
if ($user->subscription('main')->onGracePeriod()) {
  //
}
```

### 恢复订阅

如果用户取消了其订阅，并且想要恢复订阅，那么你可以使用 `resume` 方法。用户必须是在其可用期内才可以恢复订阅：

```php
$user->subscription('main')->resume();
```

如果用户取消了订阅，并在其可用期内恢复了订阅，那么认购不会立即进行计费支付。订阅只是被重新激活，其付费周期还是基于原先的计费周期。


## 试用

### 提供试用并记录信用卡信息

如果你想要在提供试用期的同时还收集支付方式的信息。你应该在你创建用户认购时使用 `trialDays` 方法:

```php
$user = User::find(1);

$user->newSubscription('main', 'monthly')
     ->trialDays(10)
     ->create($creditCardToken);
```

该方法会设置一个试用结束日期到数据库的认购记录中。并且会通知 Stripe / Braintree 暂不计费，直到超过了该日期才开始进行计费。

> 注意：如果你的客户并没有在试用期结束之前取消订阅，那么他们会在试用期结束时自动扣费，所以你应该在试用期结束前那天进行用户通知。

你可以通过用户实例的 `onTrial` 方法或者认购实例的 `onTrial` 方法来判断用户是否在试用期：

```php
if ($user->onTrial('main')) {
  //
}

if ($user->subscription('main')->onTrial()) {
  //
}
```

### 提供试用的同时不记录支付信息

如果你想要在提供试用期的同时而不收集用户的支付方法信息，你可以简单的设置 `trial_ends_at` 列到你的用户记录中。你应该在该列中设置你所期望的试用结束的日期。比如，这一般是用户注册过程中的典型做法：

```php
$user = User::create([
  // Populate other user properties...
  'trial_ends_at' => Carbon::now()->addDays(10),
]);
```

Cashier 会认为这中类型的试用是一种通用的试用期，因为这个日期并不会被关联到任何已有的订阅中。如果当前日期并没有超过 `trial_ends_at` 所设置的值，在 `User` 实例中的 `onTrial` 方法将会返回为 `true`:

```php
if ($user->onTrial()) {
  // User is within their trial period...
}
```

你也可以通过 `onGenericTrial` 方法来判断用户是否还没有进行任何认购，并且还在通用期内：

```php
if ($user->onGenericTrial()) {
  // User is within their 'generic' trial period...
}
```

一旦你准备好去创建一个真实的认购到用户，你通常应该使用 `newSubscription` 方法：

```php
$user = User::find(1);

$user->newSubscription('main', 'monthly')->create($creditCardToken);
```

## 处理 Stripe Webhooks

### 失败的认购

如果用户的信用卡过期了怎么办？不用担心，Cashier 为你提供了便捷的取消用户订阅的 Webhook 控制器。你只需要将控制器添加到路由就可以了：

```php
Route::post(
  'stripe/webhook',
  '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
);
```

就那么简单！失败的支付会在这个控制器中捕获和处理。这个控制器会在 Stripe 确定用户的订阅支付失败时（通常是在三次支付失败）取消用户的订阅。不要忘记在你的 Stripe 控制面板中配置 webhook 的 URI。

因为 Stripe webhooks 需要穿过 laravel 的 CSRF 中间件，所以你应该将你的路由抽离出 `web` 中间件组，或者在你的 `VerifyCsrfToken` 中间件中添加排除的 URI：

```php
protected $except = [
  'stripe/*',
];
```

### 其他 Webhooks

如果你想要处理额外的 Stripe webhook 事件，你可以简单的继承 Webhook 控制器。你所继承的控制器名中的方法名称应该与 Cashier 的预期约定想对应。特别的，方法名称应该有 `handle` 前缀，并且以大写驼峰的方式命名你所期望捕获的 Stripe webhook 事件名称。比如，你想要处理 `invoice.payment_succeeded` webhook, 那么你应该在控制器中添加 `handleInvociePaymentSucceeded` 方法：

```php
<?php

namespace App\Http\Controllers;

use Laravel\Cashier\Http\Controllers\WebhookController as BaseController;

class WebhookController extends BaseController
{
  /**
   * Handle a Stripe webhook.
   *
   * @param array $payload
   * @return Response
   */
   public function handleInvoicePaymentSucceeded($payload)
   {
     // Handle The Event
   }
}
````

## 处理 Braintree Webhooks

### 失败的订阅

如果用户的信用卡过期了怎么办？不用担心，Cashier 为你提供了便捷的取消用户订阅的 Webhook 控制器。你只需要将控制器添加到路由就可以了：

```php
Route::post(
  'braintree/webhook',
  '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
);
```

就那么简单！失败的支付会在这个控制器中捕获和处理。这个控制器会在 Braintree 确定用户的订阅支付失败时（通常是在三次支付失败）取消用户的订阅。不要忘记在你的 Braintree 控制面板中配置 webhook 的 URI。

因为 Stripe webhooks 需要穿过 laravel 的 CSRF 中间件，所以你应该将你的路由抽离出 `web` 中间件组，或者在你的 `VerifyCsrfToken` 中间件中添加排除的 URI：

```php
protected $except = [
  'braintree/*',
];
```

### 其他 Webhooks

如果你想要处理额外的 Braintree webhook 事件，你可以简单的继承 Webhook 控制器。你所继承的控制器名中的方法名称应该与 Cashier 的预期约定想对应。特别的，方法名称应该有 `handle` 前缀，并且以大写驼峰的方式命名你所期望捕获的 Stripe webhook 事件名称。比如，你想要处理 `dispute_opened` webhook, 那么你应该在控制器中添加 `handleDisputeOpened` 方法：

```php
<?php

namespace App\Http\Controllers;

use Braintree\WebhookNotification;
use Laravel\Cashier\Http\Controllers\WebhookController as BaseController;

class WebhookController extends BaseController
{
  /**
   * Handle a Braintree webhook.
   *
   * @param WebhookNotification $webhook
   * @return Response
   */
   public function handleDisputeOpened(WebhookNotification $notification)
   {
     // Handle The Event
   }
}
````

## 一次性付费

### 简单的付费

> 注意：在使用 Stripe 时，`charge` 方法接收的金额可以是相应货币的最小单位。而 Braintree，你应该传递美元到 `charge` 方法：

如果你想要对订阅的用户施行一次性付费，你可以使用 `charge` 方法：

```php
// Stripe Accepts Charges In Cents...
$user->charge(100);

// Braintree Accepts Charges In Dollars...
$user->charge(1);
```

`charge` 方法也可以接收一个数组作为其第二个参数，这个参数将传递给底层的 Stripe / Braintree 用于支付账单的创建：

```php
$user->charge(100, [
  'custom_option' => $value,
]);
```

当支付失败时，`charge` 方法将会抛出一个异常，如果支付成功则完整的 Stripe / Braintree 响应将会被返回：

```php
try {
  $response = $user->charge(100);
} catch (Exception $e) {
  //
}
```

### 支付并提供发票

有时候，你可能需要做一次性的支付，并且也要生成一个发票给用户。你可以使用 `invoiceFor` 方法来生成发票并提供一个 PDF 的收据给用户：

```php
// Stripe Accepts Charges In Cents...
$user->invoiceFor('One Time Fee', 500);

// Braintree Accepts Charges In Dollars...
$user->invoiceFor('One Time Fee', 5);
```

开具发票会立即向用户的信用卡进行收费。`invoiceFor` 方法也接收一个数组作为第三个参数，用于传递给底层的 Stripe / Braintree 支付账单创建时使用：

```php
$user->invoiceFor('One Time Fee', 500, [
  'custom-option' => $value,
]);
```

> 注意：`invoiceFor` 方法会创建 Stripe 发票，这将重新尝试失败的计费。如果你不希望开具发票的同时尝试失败的支付，你需要使用 Stripe API 在他们首次失败时就关闭这个支付。

## 发票

你可以通过使用 `invoices` 方法来检索一个可计费模型的发票数组：

```php
$invoices = $user->invoices();
```

当你为客户列出其发票时，你可以使用 invoice 帮助方法来显示相应的发票详情。比如，你想要在表格中列出所有的发票，从而允许用户方便的下载：

```php
<table>
  @foreach ($invoices as $invoice) 
    <tr>
      <td>{{ $invoice->date()->toFormattedDateString() }}</td>
      <td>{{ $invoice->total() }}</td>
      <td><a href=""/user/invoice/{{ $invoice->id }}">Download</a></td>
    </tr>
  @endforeach
</table>
```

**生成 PDF 类型的发票**

你需要安装 `dompdf` PHP 类库来生成 PDF:

```php
composer require dompdf/dompdf
```

在路由或者控制器中，使用 `downloadInvoice` 方法来生成一个供下载的 PDF 类型的发票。该方法会自动的生成正确的下载响应到浏览器中:

```php
Route::get('user/invoice/{invoice}', function ($invoiceId) {
  return Auth::user()->downloadInvoice($invoiceId, [
    'vendor' => 'Your Company',
    'product' => 'Your Product',
  ]); 
});
```

