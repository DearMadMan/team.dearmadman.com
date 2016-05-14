title: laravel 基础教程 —— 安装
date: 2016-04-18 14:51:26
tags: [php, laravel]
---

# laravel基础教程  —— 安装


## 环境需求

- PHP >= 5.5.9
- OpenSSL 扩展
- PDO 扩展
- Mbstring 扩展
- Tokenizer 扩展

## 安装方式


### 通过Homestead安装

**初学者不建议此方式安装!  会明显增加上手难度**  安装方式参考文档： [Laravel Homestead](https://laravel.com/docs/5.2/homestead)

`ps: 如果是win用户，我劝你还是用此方式安装吧`

### 通过Composer安装

> Composer是PHP的软件包管理系统, 是 PHP 用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。

#### 安装Composer 

``` php
curl https://getcomposer.org/download/1.0.0/composer.phar -O && mv composer.phar /usr/local/bin/composer
```

#### 安装laravel


``` php
composer global require "laravel/installer"
```

#### 新建项目

```
laravel new my-project
```

### 配置

所有配置文件都在`my-project/config`目录下

### 权限 

`storage`和`bootstrap/cache`目录下需要有写权限，否则laravel没办法运行.

```
chmod 755 -R storage bootstrap/cache
```

### 运行

``` php
cd my-project && php artisan serve 
// laravel development server started on http://localhost:8000/
```

## 结尾

其实安装本身并不难，你有可能会卡壳在下载环节，这初学时这确实浪费了不少时间。

