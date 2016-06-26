title: laravel 基础教程 —— Eloquent 集合
date: 2016-06-26 19:23:09
tags: [php, laravel]
---

# Eloquent: 集合

## 简介

通过 Eloquent 来返回的多结果集是一个 `Illuminate\Database\Eloquent\Collection` 对象，这包括通过使用 `get` 方法或者通过访问关联模型所获得的结果。Eloquent 的集合对象继承自 Laravel 的 [基础集合](https://laravel.com/docs/5.2/collections)，所以它自然地继承了几十种流畅执行的方法来与 Eloquent 模型的底层数组进行交互。

当然，所有的集合都可作为迭代器使用，这允许你像普通数组一样对他们进行循环操作：

```php
$users = App\User::where('active', 1)->get();

foreach ($users as $user) {
  echo $user->name;
}
```

事实上，集合要比数组强大的多，它提供了多种强大的接口方法可以直观的链式调用。比如，让我们删除所有未激活的用户，并且收集所有保留用户的首个名字：

```php
$users = App\User::where('active', 1)->get();

$names = $users->regect(function ($user) {
  return $user->active === false; 
})
->map(function ($user) {
  return $user->name;
});
```

> 注意：大多数的 Eloquent 集合方法都会返回一个 Eloquent 集合的实例，而 `pluck`，`keys`，`zip`，`collapse`，`flatten` 和 `flip` 方法返回基础集合的实例。

## 可用的方法

所有的 Eloquent 集合都继承自 [Laravel Collection](https://laravel.com/docs/5.2/collections) 对象。因此，所有继承自基础集合类的强大方法都是可用的。

## 自定义集合

如果你需要使用自定义的 `Collection` 对象和自己的扩展方法，你需要在你的模型中复写 `newCollection` 方法：

```php
<?php

namespace App;

use App\CustomCollection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
  /**
   * Create a new Eloquent Collection instance.
   *
   * @param array $models
   * @return \Illuminate\Database\Eloquent\Collection
   */
  public function newCollection(array $models = [])
  {
    return new CustomCollection($models);
  }
}
```

一旦你定义完成 `newCollection` 方法，那么无论什么时候，Eloquent 返回的模型的 `Collection` 实例都会获取都你所自定义的集合。如果你想要应用中所有的模型都使用自定义的集合，那么你需要在所有模型所继承的基类中复写 `newCollection` 方法。