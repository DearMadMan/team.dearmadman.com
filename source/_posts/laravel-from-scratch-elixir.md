title: laravel 基础教程 —— 炼金药
date: 2016-06-03 14:49:12
tags: [php, laravel]
---

# Laravel Elixir（炼金药）

## 简介

Laravel Elixir 为你的应用定义基础的 [Gulp](http://gulpjs.com/) 任务提供了简单流利的 API。Elixir 提供了几种常用的 CSS 和 JavaScript 预处理器和测试工具。Elixir 允许你通过链式调用来对你的资源管道进行流利的操作。比如：

```javascript
elixir(function (mix) {
  mix.sass('app.scss')
     .coffee('app.coffee');
})
```

如果你曾对如何使用 Gulp 和资源预编译有所疑惑，那么你肯定会爱上 Laravel Elixir。事实上，你也可以在开发应用的时候不使用它。你可以自由的使用任何的资源管道工具，或者一点也不用。

## 安装 & 起步

### 安装 Node

在触碰到 Elixir 之前，你首先需要确定你的机器中已经安装了 Node:

```php
node -v
```

默认的，Laravel Homestead 包含了所有你所需要的；事实上，如果你使用 Vagrant，你也是可以非常简单的通过 [这里](http://nodejs.org/download/) 来进行安装 Node。

### Gulp

接着，你需要通过 NPM 中安装 [Gulp](http://gulpjs.com/) 到全局:

```php
npm install --global gulp
```

如果你使用了版本控制器，你可能希望去运行 `npm shrinkwrap` 来锁住你的 NPM 依赖：

```php
npm shrinkwrap
```

一旦你运行了这条命令。你可以自由的提交 `npm-shrinkwrap.json` 文件到源码控制器中去。

### Laravel Elixir

最后剩下的就是安装 Elixir 了！在一个新的 laravel 应用中，你会在根目录中发现 `package.json` 文件。你可以把它想象成 `composer.json` 文件。它的不同之处就是它定义的是 Node 依赖，而不是 PHP。你可以通过下面的命令来安装这些依赖：

```php
npm install
```

如果你是在 Windows 系统中进行开发，或者运行在虚拟机中的 Windows 系统，你需要运行 `npm install` 命令的同时添加 `--no-bin-links` 选项：

```php
npm install --no-bin-links
```

## 运行 Elixir

Elixir 是建立于 Gulp 之上的，所以运行 Elixir 任务你只需要在终端运行 `gulp` 命令就行了。添加 `--production` 标识到命令会指导 Elixir 去压缩你的 CSS 和 JavaScript 文件：

```php
// Run all tasks...
gulp

// Run all tasks and minify all CSS and JavaScript...
```

**监控资源文件的变动**

为了不让你每次文件变动之后还要重新运行 `gulp` 命令，你应该使用 `gulp watch` 命令来监控资源文件的变动。这个命令会持续的在你的终端中运行。当检测到资源文件的变动，新的文件将自动编译完成：

```php
gulp watch
```

## 与样式文件合作

 在你项目的根目录中有一个 `gulpfile.js` 文件，该文件包含了所有的 Elixir 任务。Elixir 任务可以被链式的调用，会通过有序的传递来对你的资源文件进行编译。

### Less

你可以使用 `less` 方法来将 less 文件编译为 CSS。`less` 方法假设你的 less 文件存放在 `resources/assets/less` 目录中。默认的，在下面的示例中，任务运行的结果将编译的 CSS 文件存放在 `public/css/app.css`:

```php
elixir(function (mix) {
 mix.less('app.less');
});
```

你也可以合并多个 less 文件到一个单独的 CSS 文件中。默认的，他们将被编译为 `public/css/app.css` 文件：

```php
elixir(function (mix) {
mix.less([
  'app.less',
  'controllers.less'
]); 
});
```

如果你希望将编译的文件存放到自定义的位置，那么你可以传递第二个参数到 less 方法：

```php
elixir(function (mix) {
mix.less('app.less', 'public/stylesheets');
});

// Specifying a specific output filename...
elixir(function (mix) {
mix.less('app.less', 'public/stylesheets/style.css');
});
```

### sass

`sass` 方法允许你讲 sass 文件编译为 CSS。它假定你的 Sass 文件存放在 `resources/assets/sass`，你可以像下面的方式来使用该方法：

```php
elixir(function (mix) {
  mix.sass('app.scss'); 
});
```

就像 `less` 方法一样，你可以编译多个 Sass 文件到一个 CSS 文件中，还可以将编译的结果存放到指定的位置：

```php
elixir(function (mix) {
  mix.sass([
    'app.scss',
    'controllers.scss'
  ], 'public/assets/css'); 
});
```

### 原生 CSS

如果你希望合并多个 CSS 文件到一个文件中，你可以使用 `styles` 方法。你需要传递文件的路径是相对于 `resources/assets/css` 目录的，并且默认的合并的结果将会被存放到 `public/css/all.css`:

```php
elixir(function (mix) {
  mix.styles([
    'normalize.css',
    'main.css'
  ]);
});
```

当然，你也是可以自定义输出结果的路径：

```php
elixir(function (mix) {
  mix.styles([
    'normalize.css',
    'main.css'
  ], 'public/assets/css'); 
});
```

### 编译地图

编译地图是开箱即用的。所以，对于所有的被编译后的文件你都可以在相同的目录下发现 `*.css.map` 文件。这个地图文件可以使你在浏览器中追踪到编译前代码的位置，这样方便于调试。

如果你不希望生成地图，你可以在配置中进行关闭：

```php
elixir.config.sourcemaps = false;

elixir(function (mix) {
  mix.sass('app.scss');
});
```

## 与脚本合作

Elixr 也提供了多种方法来帮助你协同 JavaScript 的工作，例如 ECMAPScript 6，CoffeeScript 的编译，Browserify 模块的加载，脚本的压缩，或者是简单的原生 JavaScript 文件的合并，这都不是问题！

### CoffeeScript

`coffee` 方法可以用来编译 CoffeeScript 到原生 JavaScript。`coffee` 方法可以接收一个字符串或者一个 CoffeeScript 文件的数组来进行文件的编译，它假设你的 CoffeeScript 文件存放在 `resources/assets/coffee` 目录，并且合并生成的 JavaScript 文件到 `public/js/app.js`：

```php
elixir(function (mix) {
  mix.coffee(['app.coffee', 'controllers.coffee']);
});
```

### Browserify

Elixir 也附带了 `browserify` 方法，该方法可以在浏览器中给你提供你所需要模块的加载，并且允许你使用 ECMAScript 6 和 JSX。

这个任务假设你的脚本存放在 `resources/assets/js` 并且会将结果文件输出到 `public/js/main.js`。你也可以传递第二个参数来指定输出的位置：

```php
elixir(function (mix) {
  mix.browserify('main.js');
});

// Specifying a specific output filename...
elixir(function(mix) {
  mix.browserify('main.js', 'public/javascripts/main.js');
});
```

而 Browserify 附带了 Partialify 和 Babelify 转换器，你可以自由的安装你所希望的：

```php
npm install aliasify --save-dev
```

```php
elixir.config.js.browserify.transformers.push({
  name: 'aliasify',
  options: {} 
});

elixir(function (mix) {
  mix.browserify('main.js');
});
```

### Babel

`babel` 方法可以使你编译 `ECMAScript 6 和 7` 和 `JSX` 到原生的 JavaScript。该方法接收一个文件列表相对于 `resources/assets/js` 目录的数组，并且在 `public/js` 目录中生成 `all.js` 文件：

```php
elixir(function (mix) {
  mix.babel([
    'order.js',
    'product.js',
    'react-component.jsx'
  ]); 
});
```

你可以传递第二个参数来指定不同的输出路径。除了进行 Babel 编译，其方法的功能和 `mix.scripts` 一样。

### Scripts

如果你想要将多个 JavaScript 文件合并到一个文件中，你可以使用 `scripts` 方法。

`scripts` 方法假设你所有的文件都相对于 `resouces/assets/js` 目录，并且会默认的将结果编译到 `public/js/all.js` 文件中：

```php
elixir(function (mix) {
  mix.scripts([
    'jquery.js',
    'app.js'
  ]);
})
```
如果你需要合并多个文件到多个不同的路径，你可以通过多次链式调用并传递第二个参数作为指定输出的路径：

```php
elixir(function (mix) {
  mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
     .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
})
```

如果你需要合并指定目录下的所有脚本文件，你可以使用 `scriptIn` 方法。合并的结果将存放到 `public/js/all.js`:

```php
elixir(function (mix) {
  mix.scriptsIn('public/js/some/directory')
});
```

## 复制文件 & 目录

`copy` 方法可以用来复制文件和目录到一个新的位置。所有的操作都是相对于项目的根目录：

```php
elixir(function (mix) {
  mix.copy('vendor/foo/bar.css', 'public/css/bar.css')
})

elixir(function(mix) {
  mix.copy('vendor/package/views', 'resources/views')
})
```

## 版本 / 缓存移除

对于许多开发者比较痛苦的事就是手动的对资源文件增加时间戳或者唯一的 token 标识来强迫浏览器重新加载新的资源文件。Elixir 可以通过 `version` 方法来帮你自动的完成这些。

`version` 方法接收文件名称相对于 `public` 目录，并且它会自动的为文件名增加一个独特的 hash，这样就可以自动的进行缓存清除了。比如，新生成的文件名看上去像这样：`all-16d570a7.css`:

```php
elixir(function (mix) {
  mix.version('css/all.css') 
});
```

在生成版本化的文件之后，你可以使用 laravel 的全局帮助函数 `elixir` 在你的视图文件中进行加载适当的 hashed 资源。`elixir` 方法会自动的判断文件的名称：

```php
<link rel="stylesheet" href="{{ elixir('css/all.css') }}">
```

**对多个文件进行版本化**

你可以传递一个数组到 `version` 方法来进行多个文件的版本化：

```php
elixir(function (mix) {
  mix.version(['css/all.css', 'js/app.js']);
});
```

一旦文件本版本化，你就可以使用 laravel 的 `elixir` 方法去生成版本化的 link。记住，你只需要向 `elixir` 帮助方法中传递文件名的前缀就可以了，并不需要填写 hash 后的文件名。帮助方法会自动的识别 hash 后的文件名：

```php
<link rel="stylesheet" href="{{ elixir('css/all.css') }}">

<script src="{{ elixir('js/app.js') }}"></script>
```

## BrowserSync

BrowserSync 可以在你的前端资源文件变更之后自动的刷新的你浏览器。你可以使用 `browserSync` 方法来指导 Elixir 当你运行 `gulp watch` 命令时启动 BrowserSync 服务：

```php
elixir(function (mix) {
  mix.browserSync(); 
});
```

一旦你运行 `gulp watch`，你可以通过访问你的应用使用 3000 端口来启用 browsersyncing: `http://homestead.app:3000`。如果你使用了 `homestead.app` 之外的域名作为本地开发的支持，你可以传递一个 [options](http://www.browsersync.io/docs/options/) 数组到第一个参数到 `browserSync` 方法：

```php
elixir(function (mix) {
  mix.browserSync({
    proxy: 'project.app'
  }) 
});
```

## 调用已存在的 Gulp 任务

如果你需要从 Elixir 中调用已经存在的 Gulp 任务。你可以使用 `task` 方法。你可以想象一下你有一个 Gulp 任务是用来简单的发送一个文本：

```php
gulp.task('speak', function () {
  var message = 'Tea...Earl Grey...Hot';

  gulp.src('').pipe(shell('say ' + message))
})
```

如果你想通过 Elixir 来调用这个任务，你可以使用 `mix.task` 方法并且传递任务的名称到该方法：

```php
elixir(function (mix) {
  mix.task('speak')
})
```

**自定义监控**

如果你需要注册一个监控者在你的自定义任务中监控每次文件的变更。你可以传递一个正则表达式作为第二个参数到 `task` 方法：

```php
elixir(function (mix) {
  mix.task('speak', 'app/**/*.php') 
})
```

## 编写 Elixir 扩展

如果你需要比 Elixir `task` 方法所提供的更高的灵活性，你可以创建自己的 Elixir 扩展。Elixir 扩展允许你传递参数到你的自定义任务中。比如，你可以编写一个下面类似的扩展：

```php
// File: elixir-extensions.js

var gulp = require('gulp')
var shell = require('gulp-shell')
var Elixir = require('laravel-elixir')

var Task = Elixir.Task

Elixi.extends('speak', function (message) {
  new Task('speak', function () {
    return gulp.src('').pipe(shell('say ' + message))
  })  
})

// mix.speak('Hello World')
```

就是这么简单。你应该注意，任务的专有逻辑应该编写在方法中传递给 `Tast` 构造函数作为第二个参数。你也可以将其存放在 Gulpfile 的顶部或者提取到独立的扩展文件中。比如，你可以存放你的扩展到 `elixir-extendsions.js`，然后在你的 `Gulpfile` 文件中进行载入：

```php
// File: Gulpfile.js

var elixir = require('laravel-elixir')

require('./elixir-extendsions')

elixir(function (mix) {
  mix.speak('Tea, Earl Grey, Hot')
})
```

**自定义监控者**
如果你希望在运行 `gulp watch` 时，你的自定义任务可以在文件变更时自动的触发，你可以注册一个监控者：

```php
new Task('speak', function () {
  return gulp.src('').pipe(shell('say ' + mesage))
})
.watch('./app/**')
```