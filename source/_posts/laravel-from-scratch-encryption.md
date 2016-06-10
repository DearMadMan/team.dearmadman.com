title: laravel 基础教程 —— 加密
date: 2016-06-05 21:06:58
tags: [php, laravel]
---

# 加密

## 配置

在你使用 laravel 的加密之前，你应该先在你的 `config/app.php` 文件中设置 `key` 选项，该选项的值应该是一个 32 位的随机字符串。如果这个值没有设置正确，那么 laravel 中所有的加密都是不安全的。

## 基础用法

**对值进行加密**

你可以使用 `Crypt` 假面来加密一个值。所有的加密都是使用的 OpenSSL 和 `AES-256-CBC` 算法。除此之外，所有加密后的值都会伴随一个消息认证码（MAC）用来检测任何的消息变动。
比如，我们可以使用 `encrypt` 方法来加密一个秘钥然后将其存储到 Eloquent 模型中：

```php
<?php

namespace App\Http\Controllers;

use Crypt;
use App\User;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Store a secret message for the user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
   public function storeSecret(Request $request, $id)
   {
     $user = User::findOrFail($id);

     $user->fill([
        'secret' => Crypt::encrypt($request->secret)
      ])->save();
   }
}
```

> 注意：加密后的值在加密期间是通过 `serialize` 来传递的。这意味着你可以对对象和数组进行加密。从而，非 PHP 客户端在接收加密后的值之后还需要对数据进行反序列化操作。

**对值进行解密**

当然，你可以使用 `Crypt` 假面的 `decrypt` 方法来对值进行解密。如果值不能被解密，比如 MAC 验证失败，那么会抛出一个 `Illuminate\Contracts\Encryption\DecryptException` 错误：

```php
use Illuminate\Contracts\Encryption\DecryptException;

try {
  $decrypted = Crypt::decrypt($encryptedValue);
} catch (DecryptException $e) {
  //
}
```