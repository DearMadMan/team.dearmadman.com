title: laravel 开启跨域功能
date: 2016-08-08 10:09:58
tags: [php, laravel]
---

# 跨域的请求

出于安全性的原因，浏览器会限制 Script 中的跨域请求。由于 XMLHttpRequest 遵循同源策略，所有使用 XMLHttpRequest 构造 HTTP 请求的应用只能访问自己的域名，如果需要构造跨域的请求，那么开发者需要配合浏览器做出一些允许跨域的配置。

W3C 应用工作组推荐了一种跨资源共享的机制，这种机制让 Web 应用服务器能支持跨站访问控制，从而使得安全的进行跨站数据传输成为可能，该机制通过几种方式来对原有模式进行了扩展：
* 响应的头部应该追加 `Access-Control-Allow-Orign`，用来表明哪些请求源被允许访问资源内容
* 浏览器会对请求源和响应中的值进行匹配验证
* 对于跨域的请求，浏览器会预发送一个非简单方式的请求，来判断给定资源是否准备接受跨域资源访问
* 服务端应用通过检查请求头部的 `Orign` 来判定请求是否跨域。

## 跨源资源共享标准

> 跨源资源共享标准通过新增一系列 HTTP 头，让服务器能声明哪些来源可以通过浏览器访问该服务器上的资源。另外，对哪些会对服务器数据造成破坏性响应的 HTTP 请求方法（特别是 GET 以外的 HTTP 方法，或者搭配某些 MIME 类型的 POST 请求），标准强烈要求浏览器必须先以 OPTIONS 请求方式发送一个预请求（preflight request），从而获取知服务器端对跨源请求所支持 HTTP 方法。在确认服务器允许跨源请求的情况下，以实际的 HTTP 请求方法发送那个真正的请求。服务器端也可以通知客户端，是不是需要随同请求一起发送信用信息（包括 Cookies 和 HTTP 认证相关数据）。

跨源共享标准需要浏览器和服务端共同配合才能完成，目前浏览器厂商已经可以将请求部分自动完成，所以跨源资源访问的重点还是在于服务器端。

下面列出一些标准中可用的响应头和请求头。

## Response Header

* Access-Control-Allow-Origin      : 指明哪些请求源被允许访问资源，值可以为 "*"，"null"，或者单个源地址。
* Access-Control-Allow-Credentials : 指明当请求中省略 creadentials 标识时响应是否暴露。对于预请求来说，它表明实际的请求中可以包含用户凭证。
* Access-Control-Expose-Headers    : 指明哪些头信息可以安全的暴露给 CORS API 规范的 API。
* Access-Control-Max-Age           : 指明预请求可以在预请求缓存中存放多久。
* Access-Control-Allow-Methods     : 对于预请求来说，哪些请求方式可以用于实际的请求。
* Access-Control-Allow-Headers     : 对于预请求来说，指明了哪些头信息可以用于实际的请求中。
* Origin                           : 指明预请求或者跨域请求的来源。
* Access-Control-Request-Method    : 对于预请求来说，指明哪些预请求中的请求方式可以被用在实际的请求中。
* Access-Control-Request-Headers   : 指明预请求中的哪些头信息可以用于实际的请求中。

## Request Header

* Origin                         : 表明发送请求或预请求的来源。
* Access-Control-Request-Method  : 在发送预请求时带该请求头，表明实际的请求将使用的请求方式。
* Access-Control-Request-Headers : 在发送预请求时带有该请求头，表明实际的请求将携带的请求头。

## 中间件

在 Laravel 中允许跨域请求，我们可以构建一个追加响应的中间件，用来添加专门处理跨域的请求的响应头:

```php
<?php namespace App\Http\Middleware;

use Closure;
use Response;
class EnableCrossRequestMiddleware {

  /**
   * Handle an incoming request.
   *
   * @param  \Illuminate\Http\Request  $request
   * @param  \Closure  $next
   * @return mixed
   */
  public function handle($request, Closure $next)
  {

    $response = $next($request);
        $response->header('Access-Control-Allow-Origin', config('app.allow'));
        $response->header('Access-Control-Allow-Headers', 'Origin, Content-Type, Cookie, Accept');
        $response->header('Access-Control-Allow-Methods', 'GET, POST, PATCH, PUT, OPTIONS');
        $response->header('Access-Control-Allow-Credentials', 'true');
        return $response;
  }

}
```

其中有以下需要注意的地方：
* 对于跨域访问并需要伴随认证信息的请求，需要在 XMLHttpRequest 实例中指定 `withCredentials` 为 `true`。
* 这个中间件你可以根据自己的需求进行构建，如果需要在请求中伴随认证信息（包含 cookie，session）那么你就需要指定 `Access-Control-Allow-Credentials` 为 `true`, 因为对于预请求来说如果你未指定该响应头，那么浏览器会直接忽略该响应。
* 在响应中指定 `Access-Control-Allow-Credentials` 为 `true` 时，`Access-Control-Allow-Origin` 不能指定为 `*` 
* 后置中间件只有在正常响应时才会被追加响应头，而如果出现异常，这时响应是不会经过中间件的。

PS: 欢迎关注简书 [Laravel](http://www.jianshu.com/collection/3dff40fa5135) 专题，也欢迎 Laravel 相关文章的投稿 :)，作者知识技能水平有限，如果你有更好的设计方案欢迎讨论交流，如果有错误的地方也请批评指正，在此表示感谢 :)
