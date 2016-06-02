title: laravel 基础教程 —— 授权
date: 2016-05-25 16:33:12
tags: [php, laravel]
---

# 授权

## 简介

laravel 除了提供开箱即用的授权服务，还提供了许多简单的方式来管理授权逻辑和资源的访问控制。这些各式的方法和帮助函数便于你管理你的授权逻辑。我们将在本章中对其进行一一的解读。

## 定义能力

判断一个用户是否具有执行给定动作的能力的最简单的方式就是使用 `Illuminate\Auth\Access\Gate` 类去定义相应的能力。laravel 所提供的 `AuthServiceProvider` 类是定义这些能力的推荐位置。让我们来看个示例，我们定义一个 `update-post` 的能力，这个能力接收一个当前 `User` 和一个 `Post` 模型。在这个能力中，我们需要判断用户的 `id` 与 post 的 `user_id` 是否匹配：

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Auth\Access\Gate as GateContract;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
  /**
   * Register any application authentication / authorization services.
   *
   * @param \Illuminate\Contracts\Auth\Access\Gate $gate
   * @return void
   */
   public function boot(GateContract $gate)
   {
     $this->registerPolicies($gate);

     $gate->define('update-post', function ($user, $post) {
       return $user->id === $post->user_id;
     });
   }
}
```

注意上面的示例中我们并没有检查所给定的 `$user` 是否为 `NULL`。这是因为当给定的用户没有经过认证或者用户没有经过 `forUser` 方法指定，`Gate` 类会自动的为所有的能力返回 `false`。

**基于类的能力**

除了使用 `Closures` 的方式作为授权检查的回调来注册能力，你还可以通过传递类中的方法来进行能力的注册，当需要的时候，类会通过服务容器来解析：

```php
$gate->define('update-post', 'Class@method');
```

**授权检查拦截器**

有时候，你可能需要向某些特殊用户发放所有能力的通行证，这个时候，你可以使用 `before` 方法来定义一个回调，它会在所有的授权检查之前运行：

```php
$gate->before(function ($user, $ability) {
  if ($user->isSuperAdmin()) {
    return true;
  }
});
```

如果 `before` 的回调函数返回一个非空的结果，那么该结果将作为检查的结果。

你也可以使用 `after` 方法来定义一个在每个能力授权检查之后执行的回调函数，但是，你不能在这个回调函数内修改检查的结果：

```php
$gate->after(function ($user, $ability, $result, $arguments) {
  //
});
```

## 检查能力

### 通过 Gate 假面

一旦一个能力被定义完成，我们就可以通过多种方式来进行能力的检查。首先，我们可以使用 `Gate` 假面的 `check`，`allows`，或者 `denies` 方法。所有的这些方法都会接收能力的名称，并且会把额外的参数传递给相应能力的回调函数中。你并不需要传递当前的用户到这些方法中，`Gate` 会自动的前置当前用户到参数中并传递给能力的回调函数。所以当我们检查早前定义的 `update-post` 能力时，我们只需要传递 `Post` 的实例到 `denies` 方法中就可以了：

```php
<?php

namespace App\Http\Controllers;

use Gate;
use App\User;
use App\Post;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
  /**
   * Update the given post.
   *
   * @param int $id
   * @return Response
   */
   public function update($id)
   {
     $post = Post::findOrFail($id);

     if (Gate::denies('update-post', $post)) {
        abort(403);
     }

     // Update Post ...
   }
}

```
当然，`allows` 方法与 `denies` 相反，如果动作被授权通过则返回 `true`. `check` 方法就是 `allows` 方法的别名。

**对指定的用户检查能力**

如果你想要使用 `Gate` 假面来检查非当前经授权通过用户的其他用户是否具备相应的能力，你可以使用 `forUser` 方法：

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
  //
}
```

**传递多个参数**

当然，能力的回调函数可以接收多个参数：

```php
Gate:define('delete-comment', function ($user, $post, $comment) {
  // 
});
```

如果你的能力需要接收多个参数，你可以简单的通过 `Gate` 假面的方法进行传递一个经多个参数所组成的数组：

```php
if (Gate::allows('delete-comment', [$post, $comment])) {
  //
}
```

### 通过用户模型检查能力

事实上，你可以通过 `User` 模型的实例来检查用户的能力。默认的 laravel 的 `App\User` 模型使用了 `Authorizable` trait，这个性状包含两个方法：`can` 和 `cannot`。这两个方法和 `Gate` 假面的 `allows` 和 `denies` 方法的用法相同。我们还使用上面曾使用过的例子，修改成如下：

```php
<?php

namespace App\Http\Controllers;

use App\Post;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
  /**
   * Update the given post.
   *
   * @param \Illuminate\Http\Request $request
   * @param int $id
   * @return Response
   */
   public function update(Request $request, $id)
   {
     $post = Post::findOrFail($id);

     if ($request->user()->cannot('update-post', $post)) {
       abort(403);
     }

     // Update Post...
   }
}
```

当然，`can` 方法与 `cannot` 的相反：

```php
if ($request->user()->can('update-post', $post)) {
  // Update Post...
}
```

### 在 Blade 模板中检查能力

为了方便，laravel 提供了 `@can` Blade 指令来快速的检查当前授权的用户是否具有指定的能力。比如：

```php
<a href="/post/{{ $post->id }}">View Post</a>

@can('update-post', $post)
  <a href="/post/{{ $post->id }}/edit">Edit Post</a>
@endcan
```

你也可以通过 `@else` 指令来配合 `@can` 指令：

```php
@can('update-post', $post)
  <!-- The Current User Can Update The Post -->
@else
  <!-- The Current User Can't Update The Post -->
@endcan
```

### 在表单请求中检查能力

你也可以通过使用表单请求（继承自 Request，用于表单验证的请求类）中自定义的 `authoriza` 方法来验证 `Gate` 假面中定义的能力：

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
 public function authorize()
 {
   $postId = $this->route('post');

   return Cate::allows('update', Post::findOrFail($postId));
 }
```

## 策略（Policies）

### 创建策略

为了不让你把所有的授权逻辑全部放进 `AuthServiceProvider` 从而使应用增长为一个庞大而笨重的应用。 laravel 允许你通过 `Policy` 类来分离你的授权逻辑。策略类其实就是一个包含授权逻辑组的原生 PHP 类。

首先，让我们来生成一个策略来管理我们的 `Post` 的授权。你可以通过 `make:policy` 命令来生成一个策略。所生成的策略会存放在 `app/Policies` 目录：

```php
php artisan make:policy PostPolicy
```

**注册策略**

一旦策略存在，我们还需要在 `Gate` 类中进行注册。在 `AuthServiceProvider` 中包含了一个 `policies` 属性，该属性存放所有实体与策略间的映射。所以，我们需要将 `Post` 模型的策略指定到 `PostPolice` 类：

```php
<?php

namespace App\Providers;

use App\Post;
use App\Policies\PostPolicy;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
  /**
   * The policy mappings for the application.
   *
   * @var array
   */
   protected $policies = [
     Post::class => PostPolicy::class,
   ];

   /**
    * Register any application authentication / authorization services.
    * 
    * @param \Illuminate\Contracts\Auth\Access\Gate $gate
    * @return void
    */
    public function boot(GateContract $gate)
    {
      $this->registerPolicies($gate); 
    }
}
```

### 编写策略

一旦策略被生成和注册后，我们就可以为所有能力的授权添加验证方法。例如，让我们在 `PostPolicy` 类中定义一个 `update` 方法，用来验证所给定的用户是否具有 `update` `Post` 的能力：

```php
<?php

namespace App\Policies;

use App\Uesr;
use App\Post;

class PostPolicy
{
  /**
   * Determine if the given post can be updated by the user.
   *
   * @param \App\User $user
   * @param \App\Post $post
   * @return bool
   */
   public function update(User $user, Post $post)
   {
     return $user->id === $post->user_id;
   }
}
```

你可以继续在策略中添加其他所需进行授权验证的方法。比如，你可以继续为验证 `Post` 的各种动作而定义 `show`，`destroy`，或者 `addComment` 方法。

> 注意：所有的策略类都是通过服务容器解析而来。这意味着你可以使用类型提示来在策略类的构造函数中进行依赖注入所需要的依赖。

**拦截所有检查**

有时候，你可能需要发放给指定用户具有所有能力的通行证，这时候，你可以在策略类中定义 `before` 方法。该方法会在策略中其他所有方法被执行前运行：

```php
public function before($user, $ability)
{
  if ($user->isSuperAdmin()) {
    return true;
  }
}
```

如果 `before` 方法返回一个非空值，那么其结果将会用来作为授权验证的结果的判断依据。

### 检查策略

策略类的方法和基于授权回调的方法一样通过相同的方式作为 `Closure` 被调用。你可以使用 `Gate` 假面，`User` 模型，`@can` Blade 指令或者 `policy` helper 来进行授权的检查。

**通过 Gate 假面**

`Gate` 会通过检查传递给方法中参数的类型来确定应该使用哪一种策略。所以，如果我们传递 `Post` 实例到 `denies` 方法，`Gate` 将会自动的使用相对应的 `PostPolicy` 来进行授权验证：

```php
<?php

namespace App\Http\Controllers;

use Gate;
use App\User;
use App\Post;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
  /**
   * Update the given post.
   * 
   * @param int $id
   * @return Response
   */
   public function update($id)
   {
     $post = Post::findOrFail($id);

     if (Gate::denies('update', $post)) {
       abort(403);
     }

     // Update Post...
   }
}
```

**通过用户的模型**

`User` 模型中的 `can` 和 `cannot` 方法也会在所给定参数可用时自动匹配相应的策略。这些方法提供了一种便利的方式去验证任意用户实例是否具有所给定的能力:

```php
if ($user->can('update', $post)) {
  //
}

if ($user->cannot('update', $post)) {
  //
}
```

**通过 Blade 模板**

就像我们所期望的，`@can` Blade 指令会在所给定参数可用时自动匹配相应的策略：

````php
@can('update', $post)
  <!-- The Current User Can update The Post -->
@endcan
```

**通过策略 Helper**

全局帮助方法 `policy` 可以通过所给定的类解析相应的 `Policy` 类。例如，我们可以传递一个 `Post` 实例到 `policy` 帮助方法，该方法会返回相应的 `PostPolicy` 类：

```php
if (policy($post)->update($user, $post)) {
  //
}
```

## 控制器授权

默认的，在 laravel 中基于 `Ap\Http\Controllers\Controller` 的类都引入了 `AuthorizesRequests` trait（性状）。该性状提供了 `authorize` 方法来快速的验证所给定的动作是否有执行的能力，如果不具备相应的能力会抛出一个 `HttpException` 。

`authorize` 方法共享了其它授权方法的签证方式，如 `Gate::allows` 和 `$user->can()`。那么，让我们来使用 `authorize` 方法快速的鉴别一个请求是否具有更新 `Post` 的能力：

```php
<?php

namespace App\Http\Controllers;

use App\Post;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
  /**
   * Update the given post.
   *
   * @param int $id
   * @return Response
   */
   public function update($id)
   {
     $post = Post::findOrFail($id);

     $this->authorize('update', $post);

     // Update Post...
   }
}
```

如果这个动作通过了授权，控制器将继续执行下面的逻辑。否则会自动的抛出一个 `HttpException` 错误，这个错误会生成一个 `403 Not Authorized` 的 Http 响应。就如你所看到的， `authorize` 方法是一个非常方便的方法，它的方便之处就在于只使用一条语句执行了授权的验证或抛出异常。

`AuthorizeRequests` trait 也提供了 `authorizeForUser` 方法来进行非当前用户的用户与给定能力的鉴权：

```php
$this->authorizeForUser($user, 'update', $post);
```

**自动的确定策略方法**

通常的，策略类中的方法是与控制器中的方法相对应的。比如在上面的 `update` 方法中，控制器的方法和策略的方法使用了相同的命名： `update`。

由于这个原因，laravel 允许你通过简单的传递一个实例参数到 `authorize` 方法，在能力的鉴定中，laravel 会根据当前方法的命名自动的确定策略方法的调用。在上面的例子中，由于 `authorize` 方法是在控制器中的 `update` 方法中调用的，所以 `PostPolicy` 中的 `update` 将会被调用：

```php
/**
 * Update the given post.
 * 
 * @param int $id
 * @return Response
 */
 public function update($id)
 {
   $post = Post::findOrFail($id);

   $this->authorize($post);

   // Update Post...
 }
```





















