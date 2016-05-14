title: laravel 基础教程 —— 配置
date: 2016-04-18 16:19:40
tags: [php, laravel]
---

# laravel基础教程 —— 配置

所有配置文件都被保存在`config`目录下，并且每个配置文件里的配置项都有文档标注。

## 访问配置值

`config`全局帮助方法被提供在`vendor/laravel/framework/src/Illuminate/Foundation/helpers.php` 文件中，该函数允许使用 `.` 语法来获取文件内的配置项值。

``` php
$value = config('app.timezone');
```

你也可以在config函数传递第二个参数作为默认值，当找不到该配置项时返回默认值.

``` php
$value = config('app.timezone', 'Asia/Shanghai');
```

设置配置项的值：

``` php
config(['app.timezone' => 'Asia/Shanghai']);
```

### 环境配置

我们经常希望开发环境和生产环境具有不同的配置。比如说你在本地开发环境使用不同的缓存驱动，而laravel基于环境的配置使之非常容易。

laravel使用了[DotEnv](https://github.com/vlucas/phpdotenv)类库来构建基于环境的配置，默认的基于环境的配置信息在根目录下的`.env`文件中，如果你是通过`composer`的方式安装的laravel，那么它会自动的将`.env.example`文件复制并重命名为`.env`, 如果不是你则需要手动做了。如果你每增加一个基于环境的配置项，你最好在`.env.example`中增加相同的配置项，这样在多人协作中别人可以根据`.env.example`理解你的配置信息.

每当程序接收到request请求时，应用程序会自动装载`.env`文件，并将配置信息封装在全局变量`$_ENV`中，当然你也可以通过全局辅助函数`env()`来进行获取环境配置项信息并将其设置在你的其它配置文件中，事实上，laravel已经在部分配置文件中使用这种这种方式进行配置。

```
'debug' => env('APP_DEBUG', false),
```

`env`函数中的第二个参数为配置项的默认值，当环境配置文件`.env`中没有该配置项时会自动使用默认值。

另外你的`.env`配置环境不应该提交到版本控制器中，因为其它服务器环境或者开发者环境可能需要引入不同的环境配置。比如生产环境不应该开启debug, 不同的开发者的本地数据库配置信息可能不同。

如果你是在一个团队中做开发，你应该在`.env.example`文件中引入你增加的环境配置信息，并提交给其它开发者知道，这样他们就能够理解使用你开发的部分应该引入哪些配置信息。

### 确定当前环境

当前环境是定义在`.env`文件中的`APP_ENV`变量里的，你可以通过`App` **facade** 的 `environment` 函数来获取：

```
$environment = App::environment();
```

当然你也可以通过 全局方法 `env` 或 `app` 来获取：

```
$environment = env('APP_ENV');
# or
$environment = app()->environment();
```

有时候我们需要特别识别一下当前环境是哪种环境，并根据不同的环境执行不同的业务逻辑，这时候就需要通过`environment`函数来进行判断匹配了，当然你可以在其中传递一个或多个环境参数，只要匹配到其中任何一个都会返回true:

```
if (App::environment('local')) {
  // if env('APP_ENV') === 'local'
}

if (App::environment('local', 'staging')) {
  // env('APP_ENV') === 'local' || env('APP_ENV') === 'staging'
}
```

### 缓存配置信息

在`config`目录下有很多配置文件，配置文件中有不同的配置信息，为了启动程序更为迅速，我们可以在开发环境中将这些配置信息集中到一个配置文件中，这样，程序在被访问时，不会每次都要加载N个文件了，我们可以通过`artisan`的 `config:cache`命令 来做这件事情。所有配置文件被整合在一个文件里并被程序自动加载。

当然，在开发环境并不建议这么做，因为开发环境我们可能会频繁的更改配置信息，这样为了使配置信息及时生效我们不得不频繁的运行 `php artisan config:cache`命令， 偶尔我们会忘记执行命令。生产环境缓存配置文件应该是常态,并且应该在版本发布时执行缓存配置文件命令重新生成缓存配置信息。你应该将其做为自动发布的一部分。


### 维护模式

laravel提供了维护模式，维护模式在开启时，所有的访问请求都会被返回某个视图，这个视图是可以自定义的。如果维护模式开启，则每个请求都会返回 503 状态码。
开启维护模式的方法:

```
php artisan down
```

关闭维护模式的方法：
```
php artisan up
```

#### 维护模式响应模板

维护模式响应的视图模板存放在`resources/views/errors/503.blade.php`, 你可以自由的修改。

#### 维护模式 & 队列

在维护模式开启时，队列工作将会暂停执行，当维护模式关闭时，队列将继续进行处理工作。

#### 备选方案到维护模式

由于开启维护模式需要关闭应用程序一段时间，所以你也许可以考虑像[Envoyer](https://envoyer.io/)这种不需要关闭应用程序的持续集成服务。