title: laravel 基础教程 —— 哈希
date: 2016-06-07 23:13:18
tags: [php, laravel]
---

# 哈希

## 简介

Laravel 的 `Hash` 假面对存储用户的密码提供了安全的 Bcrypt 加密工具哈希化。如果你使用了 laravel 自带的 `AuthController` 控制器，那么你就已经使用过了它。它会在注册和认证时自动的使用 Bcrypt 加密工具。

Bcrypt 对于密码哈希化是一种很好的选择，因为它的‘工作因子’是可以调整的，这就意味着随着硬件功率的提升，生成哈希密码的时间也会提升。

## 基础用法

你可以使用 `Hash` 假面的 `make` 方法来进行密码的哈希化：

```php
<?php

namespace App\Http\Controllers;

use Hash;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Update the password for the user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
  public function updatePassword(Request $request, $id)
  {
    $user = User::findOrFail($id);

    // validate the new password length...

    $user->fill([
        'password' => Hash::make($request->newPassword)
      ])->save();
  }
}
```

另外，你可以使用全局帮助方法 `bcrypt` 来做同样的事情：

```php
bcrypt('plain-text');
```

**针对 Hash 密码的验证**

`check` 方法允许你验证所给定的哈希化的密码是否与所给定的原始字符串相匹配。如果你使用了 laravel 所提供的 `AuthController` 控制器，那么你并不需要直接的去使用这个方法，因为在认证控制器中系统会自动的调用这个方法：

```php
if (Hash::check('plain-text', $hashedPassword)) {
  // The passwords match...
}
```

**检查密码是否需要刷新**

`needsRehash` 方法可以用来判断密码是否需要重新生成，当工作因子变更时而密码没有使用新的工作因子生成时，其将返回 `true`:

```php
if (Hash::needsRehash($hashed)) {
  $hashed = Hash::make('plain-text');
}
```