title: 浅谈多个社交账号的绑定设计 
date: 2016-09-27 14:02:04
tags: [php, laravel]
---

Dearmadman 在 [Laravel Socialite 详解](http://www.jianshu.com/p/14c5e2faea70) 中使用 [larastarscn/socialite](https://github.com/larastarscn/socialite) 解决了第三方账号登录集成的问题，那么在获取到用户资料之后呢？集成多个社交账号，该如何绑定同一个账号？本篇就让我们来探讨一下集成登录的那点事。

## 起初

起初，当我们只需要集成单个社交登录时，我们可能会为了快速的完成任务简单粗暴的在用户模型中加入 open_id 或者 github_id 类似的属性，那么在数据库中，我们需要在表中添加相应的字段。这是可以快速有效的完成任务的做法。

但是，当更多的需求来临时，要求我们额外的集成一种或者多种社交登录，那该怎么办？难道我们还要任性的在表结构中添加相应的字段？

``` php
Schema::table('users', function ($table) {
  $table->string('github_id');
  $table->string('douban_id');
});
```

这很明显的违背了开放封闭原则，假如我们这么做，那么可以想象的当每多集成一种登录时，我们就需要对数据表结构做出一次修正，并且，在登录授权回调验证时，还要增加一道集成驱动与字段查询匹配的工序。

那应该怎么做？

## 设想

这么想来，User 表是否承担了过多的能力，它是否应该浪费自己的精力来管理这些社交标识？那不如我们安排 SocialiteUser 来专门管理用户与社交账号之间的关系？我们需要设计一种易扩展的方案来管理不同驱动的社交登录，那么我们很容易的设计出这种表结构：

``` php
- socialite_users
  - id
  - user_id
  - driver
  - open_id
```

SocialiteUser 应该具有哪些职责？很显然，它主要用来维护社交登录标识和用户模型之间的关系。那么它应该具有以下能力：
- 将社交登录账户绑定到用户模型上
- 获取匹配的用户模型

以下为简单的代码演示：

``` php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class SocialiteUser extends Model
{
    public $guarded = ['id'];

    /**
     * Get user instance by driver and openid.
     *
     * @param  $driver  string
     * @param  $openid  string
     * @return /App/User|null
     */
    public function getUser($driver, $openid)
    {
        $finder =  $this->where([
            'driver' => $driver,
            'open_id' => $openid
        ])->first();

        return $finder ? $finder->user : $finder;
    }

    /**
     * get related user model.
     *
     * @return /App/User||null
     */
    public function user()
    {
        return $this->belongsTo('App\User');
    }

    /**
     * Save a new record.
     *
     * @param  $userId  integer
     * @param  $driver  string
     * @param  $id  string
     * @return /App/SocialiteUser
     */
    public function saveOne($userId, $driver, $id)
    {
        return $this->create([
            'user_id' => $userId,
            'driver' => $driver,
            'open_id' => $id
       ]);
    }
}

```

## 使用 

在授权登录流程中，用户同意授权，第三方应用将重定向到回调路由，回调路由中 Socialite 会主动请求获取用户资料，并将用户的社交标识 ID 映射到 `User` 模型的 id 属性上。

那么我们就可以在回调路由中根据驱动标识和用户相应的社交标识 ID 来匹配查询库中是否已存在绑定的用户。如果存在那就直接使用匹配到的用户登录，如果不存在，那么就生成一个用户，并为这个用户附加社交账户信息。然后使用新生成的账户登录。

``` php
<?php

namespace App\Http\Controllers;

use App\SocialiteUser;
use App\User;
use Socialite;

class OAuthAuthorizationController extends Controller
{
    //
    public function redirectToProvider($driver)
    {
        return Socialite::driver($driver)->redirect();
    }

    public function handleProviderCallback($driver)
    {
        $user =  Socialite::driver($driver)->user();

        $model = new User();
        $socialiteUser = new SocialiteUser();
        $finder = $socialiteUser->getUser($driver, $user->id);
        if (! $finder) {
            $finder = $model->generateUserInstance();
            $finder->save();
            $socialiteUser->saveOne($finder->id, $driver, $user->id);
        }
        
        Auth::login($finder);

        return view('home');
    }
}

```

这样看来，如果需求一种新的社交登录的集成，那么完全不需要做出其它代码的改动，直接配置驱动就可以了。

PS: 欢迎关注简书 [Laravel](http://www.jianshu.com/collection/3dff40fa5135) 专题，也欢迎 Laravel 相关文章的投稿 :)，作者知识技能水平有限，如果你有更好的设计方案欢迎讨论交流，如果有错误的地方也请批评指正，在此表示感谢谢谢 :)
