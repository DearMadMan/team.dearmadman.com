title: Laravel Socialite 详解
date: 2016-09-21 13:23:35
tags: [php, laravel]
---

不久之前 Dearmadman 曾写过一篇 [使用 Laravel Socialite 集成微信登录](http://www.jianshu.com/p/1e16beb46ea3) 的文章，但是似乎还是有些同学不太明白，询问着如何集成 QQ 登录，那么，本篇我们就来剖析一下 Laravel Socialite 的详细内容。让各位同学都获得 Laravel Socialite 所提供的驱动适配能力。

## Laravel Socialite

> Laravel Socialite 为第三方应用的 OAuth 认证提供了非常丰富友好的接口，我们使用它可以非常方便快捷的对类似微信、微博等第三方登录进行集成。

那么首先，你需要知道 Laravel Socialite 主要是用于那些 OAuth 授权登录的。这些场景下的第三方应用比如微信，QQ，微博等，都使用了 OAuth 协议为用户提供授权其它应用获取用户信息及权限的能力。

## Open Authorization

> OAuth（开放授权）是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。

> OAuth 允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站（例如，视频编辑网站)在特定的时段（例如，接下来的 2 小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth 让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容。

我们的应用与使用 OAuth 标准的第三方应用的交互流程一般是这样的：

* 展示跳转到第三方应用登录的链接
* 用户点击链接跳转到第三方应用登录并进行授权
* 在用户授权后第三方应用会跳转到我们所指定的应用回调资源地址并伴随用于交互 AccessToken 的 Code
* 我们的应用拿到 Code 后自主请求第三方应用并使用 Code 获取该用户的 AccessToken
* 获取 AccessToken 之后的应用即可自主的从第三方应用中获取用户的资源信息

## 详解

对于以上流程，我们来分析一下 Laravel Socialite 针对上面的每一步都做了些什么，我们将构建一个 `QQProvider` 来作为演示示例：

### 第三方应用的登录链接

显而易见的，第一步我们需要构建用于跳转到第三方登录的 URL，这里需要提供一些额外的参数来进行应用的识别，比如，当你为你的应用注册 QQ 登录网站接入应用时，QQ 互联会为你提供一组数据用来验证登录来源的合法性，通常这些关键数据为以下几项:

``` js
{
  appid: 'xxxxxxxxx',
  app_secret: 'xxxxxxxxx',
  redirect: 'xxxxxxxxxxx'
}
```

在 Laravel Socialite 中这些关键数据映射到了 `AbstractProvider` 顶级抽象中，你可以将这些配置项定义到 `services.php` 配置文件中:

``` php
'qq' => [
    'client_id' => 'your-qq-app-id',
    'client_secret' => 'your-qq-app-secret',
    'redirect' => 'http://your-callback-url',
],
```

我们将会在后续注册扩展驱动时使用这些配置。

如果需要为第三方应用登录构建自定义的驱动，那么你需要先继承 `AbstractProvider`，并且实现 `ProviderInterface` 接口，因为国内流行的应用都使用的 OAuth 2.0 协议，比如 QQ，就是使用的这种协议，所以我们需要继承 `Laravel\Socialite\Two\AbstractProvider` 抽象类，并且实现 `ProviderInterface` 接口。

`ProviderInterface` 接口限定了两个方法: `redirect` 和 `user`，其中 `redirect` 方法就是返回构建后的跳转响应，用于跳转到第三方应用进行授权登录，而 `user` 方法就是用来在用户同意授权登录之后，获取第三方应用中用户的信息数据的。这些方法已经在 `AbstractProvider` 抽象类中提供了实现。所以基本上你不需要再去实现它。

Socialite 通过三个函数的配合来构造跳转 URL，它们分别是 `getAuthUrl`，`buildAuthUrlFromBase` 和 `getCodeFields`：

- getAuthUrl($state): 返回构造完成后的跳转 URL，它需要调用 `BuildAuthUrlFromBase` 函数，我们只需要在这里指定第三方应用所规定的应用登录授权的地址即可。
- BuildAuthUrlFromBase($url, $state): 它主要是为授权地址构建相应的查询参数，将驱动配置中的关键数据以及第三方应用所规定的数据填充到 URL 中。在它的内部需要调用 `getCodeFields` 函数，我们只需要在这里返回第三方应用所规定的应用登录授权的地址与查询参数合并即可。
- getCodeFields($state = null): 将驱动配置信息映射到第三方应用登录验证所需的参数中。

## 使用 Code 换取 AccessToken

当用户同意登录授权之后，第三方应用会将地址跳转到 `redirect` 所配置的地址中，并携带 `code` 参数，你需要使用 `code` 来换取 `access_token`。然后使用 `access_token` 换取用户的信息数据。这就是集成登录的整个流程。

`AbstractProvider` 中已经为 `user` 方法提供了基本的实现，并且它将经常变化的部分提取到了其它的方法中，这样我们在开发新驱动的时候，基本不需要重写 `user` 方法。我们来看一些为了完成这些操作，我们需要用到哪些方法。

首先，在登录回调的路由中，我们会获得请求中携带的 `code` 参数，然后需要使用 `code` 来换取 `access_token`，我们先来看一下底层 `AbstractProvider` 中 `user` 的实现：


``` php
/**
 * {@inheritdoc}
 */
public function user()
{
    if ($this->hasInvalidState()) {
        throw new InvalidStateException;
    }

    $response = $this->getAccessTokenResponse($this->getCode());

    $user = $this->mapUserToObject($this->getUserByToken(
        $token = Arr::get($response, 'access_token')
    ));

    return $user->setToken($token)
                ->setRefreshToken(Arr::get($response, 'refresh_token'))
                ->setExpiresIn(Arr::get($response, 'expires_in'));
}
```

其中用到的关键方法有：

- getAccessTokenResponse($code): 它需要使用 code 来换取 access_token 响应的主体，它应该返回一个包含了 `access_token` 的数组。根据不同应用所规定的响应格式，你可能会需要在这里做出一些适配，那么，在集成自定义驱动时，重写这个方法就可以了。
- getUserByToken($token): 使用 access_token 向第三方应用获取用户的数据。它应该返回一个数组。
- mapUserToObject(array $user): 它应该在使用 `access_token` 换取用户数据之后，将用户数据数组存储到 `Laravel\Socialite\Two\User` 实例中的 user 属性中，并将相应的数据映射到实例的属性。

所以流程上也就很清晰了，那么扩展自定义的驱动也就很简单了，调用 `getAccessTokenResponse` 方法时，需要指定换取 token 的 URL，所以我们需要重写 `getTokenUrl` 方法：

``` php
/**
 * {@inheritdoc}.
 */
public function getTokenUrl()
{
    return 'https://graph.qq.com/oauth2.0/token';
}
```

在构建换取 token 的 URL 时，还要组合相应的参数，所以我们需要重写 `getTokenFields` 方法：

``` php
/**
* {@inheritdoc}.
*/
protected function getTokenFields($code)
{
    return [
        'client_id' => $this->clientId, 'client_secret' => $this->clientSecret,
        'code' => $code, 'grant_type' => 'authorization_code',
        'redirect_uri' => $this->redirectUrl,
    ];
}
```

然后，在 `getUserByToken` 方法中完成使用 `token` 换取用户数据的业务逻辑就可以了，它应该返回一个数组:

``` php
/**
 * {@inheritdoc}.
 */
public function getUserByToken($token)
{
    $response = $this->getHttpClient()->get('https://graph.qq.com/oauth2.0/me', [
        'query' => [
            'access_token' => $token,
        ],
    ]);

    $openId = json_decode($this->jsonpToJson($response->getBody()->getContents()), true)['openid'];
    $this->setOpenId($openId);

    $response = $this->getHttpClient()->get('https://graph.qq.com/user/get_user_info', [
        'query' => [
            'oauth_consumer_key' => $this->clientId, 'access_token' => $token,
            'openid' => $openId, 'format' => 'json'
        ]
    ]);


    return json_decode($response->getBody(), true);

}
```

最后就是注册了，在 [使用 Laravel Socialite 集成微信登录](http://www.jianshu.com/p/1e16beb46ea3) 中我们已经谈到过 Socialite 自定义驱动的实现机制：

``` php

<?php

namespace Crowdfunding\Providers\Socialite;

use Illuminate\Support\ServiceProvider;

class WechatServiceProvider extends ServiceProvider
{
    public function boot()
    {
       $this->app->make('Laravel\Socialite\Contracts\Factory')->extend('wechat', function ($app) {
            $config = $app['config']['services.wechat'];
            return new WechatProvider(
                $app['request'], $config['client_id'],
                $config['client_secret'], $config['redirect']
            );
       });
    }
    public function register()
    {

    }
}
```

Laravel Socialite 对 OAuth 协议的流程进行了高度的抽象，并将相应的功能解耦，所以在自定义扩展时你并不需要做出太多，如果你想要了解的更多，你可以参考 [larastarscn/socialite](https://github.com/larastarscn/socialite) 的源码，它只是简单的为微信，QQ，微博的登录提供了扩展驱动。

PS: 欢迎关注简书 [Laravel](http://www.jianshu.com/collection/3dff40fa5135) 专题，也欢迎 Laravel 相关文章的投稿 :)，作者知识技能水平有限，如果你有更好的设计方案欢迎讨论交流，如果有错误的地方也请批评指正，在此表示感谢谢谢 :)
