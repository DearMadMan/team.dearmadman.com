title: laravel 基础教程 —— 本土化
date: 2016-06-08 10:23:17
tags: [php, laravel]
---

# 本土化

## 简介

laravel 的本土化特性提供了一种便捷的方式来获取各种语言的字符串。它允许你在应用中可以简单的支持多语言。

语言字符串被存储在 `resources/lang` 目录下的文件里。在这个目录下，应该划分出多个支持的语言子目录：

```php
/resources
    /lang
        /en
            messages.php
        /es
            messages.php
```

所有的语言文件简单的返回一个使用字符串键化的数组，比如：

```php
<?php

return [
  'welcome' => 'Welcome to our application'
];
```

**配置语言环境**

应用使用的默认语言被存储在 `config/app.php` 配置文件中。当然，你可以根据需求自由的修改当前设置。你也可以使用 `App` 假面的 `setLocale` 方法在运行时切换语言：

```php
Route::get('welcome/{locale}', function ($locale) {
  App::setLocale($locale);

  // 
});
```
你也可以设置一个备用语言，它会在激活的语言环境中未找到所给定的语言键时使用。备用语言也是在 `config/app.php` 配置文件中进行设置：

```php
'fallback_local' => 'en',
```

你可以使用 `App` 假面的 `isLocale` 方法来判断当前语言环境是否给定的值：

```php
if (App::isLocale('en')) {
  //
}
```

你可以使用 `App` 假面的 `getLocale` 方法来获取当前的语言环境：

```php
return App::getLocale();
```

## 基础用法

你可以使用 `trans` 帮助方法来从语言文件中提取内容。`trans` 方法接收文件名和键值作为其第一个参数。比如，让我们检索 `resources/lang/messages.php` 语言文件中的 `welcome` 键：

```php
echo trans('messages.welcome');
```

如果你使用 Blade 模板引擎，那么你可以在视图文件中使用双括号语法或者使用 `@lang` 指令来提取内容：

```php
{{ trans('message.welcome') }}

@lang('messages.welcome')
```

如果指定的语言文件中的键不存在，`trans` 方法就会简单的返回这个键名。所以，如果上述示例中的键不存在，那么会返回 `messages.welcome`。

**语言内容中的参数替换**

如果你需要，你可以定义一个占位到你的语言内容中。所有的语言占位都是使用的 `:` 来标识。比如，你希望定义一个欢迎某某的一个占位：

```php
'welcome' => 'Welcome, :name',
```

你可以在 `trans` 方法中传递一个数组作为第二个参数，它会将数组的值替换到语言内容的占位符中：

```php
echo trans('message.welcome', ['name' => 'dayle']);
```

如果你的占位符中包含了首字母大写或者全体大写，那么替换后的字符串也是相应的方式进行处理：

```php
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
```

### 多元化

语言的多元化是个复杂的问题。不同的语言拥有各种复杂的规则来定义多元化。你可以通过使用 `|` 字符串来从一个完整的字符串中区分单数或者复数的形式：

```php
'apples' => 'There is one apple|There are many apples',
```

然后你可以使用 `trans_choice` 方法来根据给定的数目获取语言内容。在这个例子中，因为数目大于 1，所以语言会提取复数形式：

```php
echo trans_choice('messages.apples', 10);
```

因为 laravel 的翻译者是基于 Symfony 翻译组件的，所以你可以创建更为复杂的复杂复数规则：

```php
'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',
```

## 覆盖供应商语言文件

很多包都会附带自己的语言文件。如果你需要替换其中某些语言内容，你可以通过存放自己的语言文件到 `resources/lang/vendor/{package}/{locale}` 目录。

所以，比如，如果你需要覆盖一个包名为 `skyrim/hearthfire` 的 `messages.php` 文件，你需要存放你自己的语言文件到 `resources/lang/vendor/hearthfire/en/message.php` 。在这个文件中你应该只添加想要覆盖的语言内容，任何未覆盖的内容还将使用包自带的原始语言文件。

