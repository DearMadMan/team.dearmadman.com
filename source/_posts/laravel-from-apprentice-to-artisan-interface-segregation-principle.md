title: 从百草园到三味书屋 —— 接口隔离原则
date: 2016-07-10 21:37:57
tags: [php, laravel]
---

# Interface Segregation Principle 接口隔离原则

## Introduction 介绍

接口隔离原则规定在实现接口的时候，不能强迫去实现没有用处的方法。你是否曾被迫去实现一些接口里你用不到的方法？如果答案是肯定的，那你可能创建了一个空方法放在那里。被迫去实现用不到的函数，这就是一个违背了接口隔离原则的例子。

在实际操作中，该原则要求接口必须粒度很细，且专注于一个领域。听起来很耳熟？记住，所有五个“坚实”原则都是相关的，也就是说当打破一个原则时，你通常肯定打破了其他的原则。在这里当你违背了接口隔离原则后，肯定也违背了单一职责原则。

“臃肿”的接口，有着很多不是所有的实现类都需要的方法。与其写这样的接口，不如将其拆分成多个小巧的接口，里面的方法都是各自领域所需要的。这样将臃肿接口拆成小巧、功能集中的接口后，我们就可以使用小接口来编码，而不必为我们不需要的功能买单。

> **Interface Segregation Principle 接口隔离原则**
>
> 该原则规定，一个接口的一个实现类，不应该去实现那些自己用不到的方法。如果需要，那就是接口设计有问题，违背了接口隔离原则。

## In Action 实践

为了说明该原则，我们来思考一个关于会话处理的类库。实际上我们将要考察 PHP 自己的 `SessionHandlerInterface`。下面是该接口定义的方法，他们是从 PHP 5.4 版才开始有的：

```php
interface SessionHandlerInterface {
  public function close();
  public function destroy($sessionId);
  public function gc($maxLifetime);
  public function open($savePath, $name);
  public function read($sessionId);
  public function write($sessionId, $sessionData);
}
```

现在我们知道接口里面都是什么方法了，我们打算用 Memcached 来实现它。Memcached 需要实现这个接口里的所有方法么？不，里面一半的方法对于 Memcached 来说都是不需要实现的！

因为 Memcached 会自动清除存储的过期数据，我们不需要实现 `gc` 方法。我们也不需要实现 `open` 和 `close` 方法。所以我们被迫去写空方法来占着位子。为了解决这个问题，我们来定义一个小巧的专门用来垃圾回收的接口：

```php
interface GarbageCollectorInterface {
  public function gc($maxLifetime);
}
```

现在我们有了一个小巧的接口，功能单一而专注。需要垃圾清理的只用依赖这个接口即可，而不必去依赖整个会话处理。

为了更深入理解该原则，我们用另一个例子来强化理解。想象我们有一个名为 `Contact` 的 Eloquent 类，定义成这样：

```php
class Contract extends Eloquent {
  public function getNameAttribute()
  {
    return $this->attributes['name'];
  }
  public function getEmailAtribute()
  {
    return $this->attributes['email'];
  }
}
```

现在我们再假设我们应用里还有一个叫 `PasswordReminder` 的类来负责给用户发送密码找回邮件。下面是 `PasswordReminder` 的定义方式的一种：

```php
class PasswordReminder {
  public function remind(Contact $contact, $view)
  {
    // Send password reminder e-mail...
  }
}
```

你可能注意到了，`PasswordReminder` 依赖着 `Contract` 类，也就是依赖着 Eloquent ORM。对于一个密码找回系统来说，依赖着一个特定的 ORM 实在是没必要，也是不可取的。切断对该 ORM 的依赖，我们就可以自由的改变我们后台存储机制或 ORM，同时不会影响到我们的密码找回组件。重申一遍，违背了“坚实”原则的任何一条，就意味着有个类它知道的太多了。

要切断这种依赖，我们来创建一个 `RemindableInterface` 接口。事实上 Laravel 已经有了这个接口，并且默认由 `User` 模型实现了该接口：

```php
interface RemindableInterface {
  public function getReminderEmail();
}
```

一旦接口定义好了，我们就可以在模型上实现它：

```php
class Contact extends Eloquent implements RemindableInterface {
  public function getReminderEmail() 
  {
    return $this->email;
  }
}
```

最终我们可以在 `passwordReminder` 里面依赖这样一个小巧且专注的接口了：

```php
class PasswordReminder {
  public function remind(RemindableInterface $remindable, $view)
  {
    // Send password reminder e-mail...
  }
}
```

通过这小小的改动，我们已经移除了密码找回组件里不必要的依赖，并且使它足够灵活能使任何实现了 `RemindableInterface` 的类或 ORM。这其实正是 Laravel 的密码找回组件如何保持与数据库 ORM 无关的秘诀！

> **Knowledge Is Power 知识就是力量**
>
> 我们再次发现了一个使类知道太多东西的陷阱。通过小心留意是否让一个类知道了太多，我们就可以遵守所有的“坚实”原则。

译文地址：[http://my.oschina.net/zgldh/blog/389246](http://my.oschina.net/zgldh/blog/389246)