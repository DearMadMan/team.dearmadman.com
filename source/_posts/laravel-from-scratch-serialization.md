title: laravel 基础教程 —— 序列化
date: 2016-06-26 20:49:47
tags: [php, laravel]
---

# Eloquent: 序列化

## 简介

当构建 JSON APIs 时，你需要经常的将你的模型和关联转换为数组或者 JSON 格式。Eloquent 包含了便捷的方法来进行这些转换，以及控制哪些属性应该包含在序列化之中。

## 基础用法

**转换模型到数组**

为了将一个模型和其关联加载的数据转换为数组，你需要使用 `toArray` 方法。这个方法是一个递归的方法，所以所有的属性和所有的关联（包括关联的关联）都会被转换为数组：

```php
$user = App\User::with('roles')->first();

return $user->toArray();
```

你也可以将集合转换为数组：

```php
$users = App\User::all();

return $users->toArray();
```

**转换模型到 JSON**

你可以使用 `toJson` 方法来将一个模型转换为 JSON。就像 `toArray` 方法，`toJson` 方法也是递归方法，所以所有的属性和其关联将会被转换为 JSON：

```php
$user = App\User::find(1);

return $user->toJson();
```

另外，你可以转换模型或者集合到一个字符串中，它将会自动的调用 `toJson` 方法：

```php
$user = App\User::find(1);

return (string) $user;
```

由于模型和集合都会在转换为字符串时被自动的转换为 JSON。所以你可以直接在应用的路由或者控制器中返回 Eloquent 对象：

```php
Route::get('users', function () {
  return App\User::all(); 
});
```

## 从 JSON 中隐藏属性

有时候你可能会选择限制一些属性的展示，比如，密码，这些本应包含在模型中的数值或者 JSON 中的属性。你可以在模型中定义 `$hidden` 属性来选择隐藏它们：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
  /**
   * The attributes that should be hidden for arrays.
   *
   * @var array
   */
  protected $hidden = ['password'];
}
```

> 注意：当隐藏一些关联时，你应该使用关联模型的名字，而不是动态属性的名字。

另外，你也可以使用 `visible` 属性来定义一个白名单属性集来指明这些属性应该被包含在模型的数组或者 JSON 里：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
  /**
   * The attributes that should be visible in arrays.
   *
   * @var array
   */
  protected $visible = ['first_name', 'last_name'];
}
```

**临时的修改属性的可见性**

如果你希望在给定的模型实例中来设置一些通常隐藏的属性为可见，你可以使用 `makeVisible` 方法。`makeVisible` 方法将返回模型实例，这样有利于链式调用：

```php
return $user->makeVisible('attribute')->toArray();
```

同样，如果你希望在给定的模型实例中设置通常可见的属性为不可见，那么你可以使用 `makeHidden` 方法：

```php
return $user->makeHidden('attribute')->toArray();
```

## 追加值到 JSON

偶尔，你可能需要在模型中添加一些数据库中不存在的字段的属性。那么，你需要先为这个值定义一个访问器：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
  /**
   * Get the administrator flag for the user.
   *
   * @return bool
   */
  public function getIsAdminAttribute()
  {
    return $this->attributes['admin'] == 'yes';
  }
}
```

当你创建完访问器之后，将其属性的名称添加到模型的 `appends` 属性中:

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
  /**
   * The accessors to append to the model's array form.
   *
   * @var array
   */
  protected $appends = ['is_admin'];
}
```

当你在 `appends` 列表中加入属性之后，它就会被加入到模型的数组和 JSON 形式中。在 `appends` 数组中的属性也受到模型中的 `visible` 和 `hidden` 设置的约束。