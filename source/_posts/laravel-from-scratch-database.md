title: laravel 基础教程 —— 数据库
date: 2016-06-23 13:00:10
tags: [php, laravel]
---

# 数据库入门

## 简介

Laravel 可以非常简单的使用原生的 SQL，流利的查询构建器，和 [Eloquent ORM](https://laravel.com/docs/5.2/eloquent) 来进行跨终端的后端数据库连接的建立与交互。
目前，Laravel 支持的 4 种数据库系统：
- MySQL
- Postgres
- SQLite
- SQL Serve


### 配置

Laravel 使建立数据库连接与其进行交互变的非常简单。而数据库的配置文件就存放在 `config/database.php`。在这个文件中你可以定义所有的数据库连接，以及指定何种连接作为应用的默认数据库连接。对于所有支持的数据库系统，该文件中都给出了相应的配置示例。

默认的，Laravel 的环境配置示例已经在 Laravel Homestead 中被设置使用，当然，你可以自由的根据需求来修改这个配置。

**SQLite 配置**

在创建 SQLite 数据库之后，比如你可以通过 `touch database/database.sqlite` 命令直接创建一个文件，然后你就可以在环境配置中添加新创建的数据库的绝对路径了：

```php
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

**SQL Server Configuration**

Laravel 支持 SQL Server 的开箱即用。但是，你需要为这个数据库添加连接配置: 

```php
'sqlsrv' => [
  'driver' => 'sqlsrv',
  'host' => env('DB_HOST', 'localhost'),
  'database' => env('DB_DATABASE', 'forge'),
  'username' => env('DB_USERNAME', 'forge'),
  'password' => env('DB_PASSWORD', ''),
  'charset' => 'utf8',
  'prefix' => '', 
],
```


**读写分离的连接**

有时候，你可能希望使用一个数据库连接来处理 SELECT 声明，而使用另外一个连接来处理 INSERT，UPDATE，和 DELETE 声明。Laravel 可以轻而易举的做到这些，并且不论你是使用的原生查询还是通过查询构造器，又或者是 Eloquent ORM，laravel 都会为你提供恰当的连接。

下面给出的示例来演示如何配置读写分离的数据库：

```php
'mysql' => [
  'read' => [
    'host' => '192.168.1.1',
  ],
  'write' => [
    'host' => '192.168.1.2'
  ],
  'driver' => 'mysql',
  'database' => 'database',
  'username' => 'root',
  'password' => '',
  'charset' => 'utf8',
  'collation' => 'utf8_unicode_ci',
  'prefix' => '',
],
```

你应该注意到了数组中被添加了两个键：`read` 和 `write`。它们两个都是一个数组且只包含了一个键： `host`。而对 `read` 和 `write` 连接的其余选项都会从 `mysql` 数组中合并过来。

所以，如果我们想要覆盖主数组中的值，我们只需要在 `read` 和 `write` 数组中添加相应的项就可以了。在这个场景下，`192.168.1.1` 会被用来作为读连接，而 `192.168.1.2` 将会被用来作为写连接。数据库凭证，数据表前缀，字符集，所有主 `mysql` 数组中的其它选项都会被分享到这两个连接中。

## 执行原生的 SQL 查询

当你配置完你的数据库连接之后，你可以通过使用 `DB` 假面来执行查询。`DB` 假面对每种类型的查询都提供了方法：`select`，`update`，`insert`，`delete` 和 `statement`。

**执行 select 查询**

我们可以使用 `DB` 假面的 `select` 方法来进行基础的查询：

```php
<?php

namespace App\Http\Controllers;

use DB;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Show a list of all of the application's users.
   *
   * @return Response
   */
   public function index()
   {
     $users = DB::select('select * from users where active = ?', [1]);

     return view('user.index', ['users' => $users]);
   }
}
```

传递到 `select` 方法的第一个参数是一个原生的 SQL 查询，而第二个参数则是传递的所有绑定到查询中的参数值。通常，这些值都是关于条件约束的。参数绑定提供了针对 SQL 注入的保护。

`select` 方法总是返回一个 `array` 作为结果。而数组中的每一项都应该是一个 `StdClass` PHP 对象，这允许你从结果中访问它的值：

```php
foreach($users as $user) {
  echo $user->name;
}
```

**使用命名的绑定**

你可以使用命名的绑定来取代使用 `?` 参数绑定：

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

**运行一个 Insert 声明**

你可以使用 `DB` 假面的 `insert` 方法来执行一个 `insert` 声明。就像 `select` 一样，该方法接收一个原生的 SQL 查询作为第一个参数，一个绑定值作为第二个参数：

```php
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
```

**运行一个 Update 声明**

`update` 方法会更新数据库中检索到的记录。这个方法会返回受影响的行数：

```php
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
```

**运行一个 Delete 声明**

`delete` 方法应该用来从数据库中删除记录。就像 `update` 一样，它返回删除的行数：

```php
$deleted = DB::delete('delete from users');
```

**运行普通的声明**

一些数据库声明不应该返回任意的值。对于这些特别的操作，你可以使用 `DB` 假面的 `statement` 方法：

```php
DB::statement('drop table users');
```

### 监听查询事件

如果你希望在每次应用中执行查询时都受到通知，那么你可以使用 `listen` 方法，该方法通常用来作为查询日志的调试。你可以在服务提供者中进行注册一个查询监听器：

```php
<?php

namespace App\Providers;

use DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
  /**
   * Bootstrap any application services.
   *
   * @return void
   */
   public function boot()
   {
     DB::listen(function ($query) {
       // $query->sql
       // $query->bindings
       // $query->time
     });
   }

   /**
    * Register the service provider.
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

## 数据库事务

你可以通过使用 `DB` 假面的 `transaction` 方法来在数据库事务中执行某些操作。如果在事务 `Closure` 中抛出了异常，那么事务会自动的执行回滚操作。如果 `Closure` 成功的执行，那么事务就会自动的进行提交操作。你并不需要在使用 `transaction` 方法时担心如何手动的执行回滚或者提交操作：

```php
DB::transaction(function () {
  DB::table('users')->update(['votes' => 1]);

  DB::table('posts')->delete(); 
});
```

**手动的使用事务**

如果你想要手动的执行事务，并且想要对回滚和提交具有完整的控制权，那么你可以使用 `DB` 假面的 `beginTransaction` 方法：

```php
DB::beginTransaction();
```

你可以通过 `rollBack` 方法来进行回滚操作：

```php
DB::rollBack();
```

最后，你可以使用 `commit` 方法来进行提交操作：

```php
DB::commit();
```

> 注意：使用 `DB` 假面的事务方法同样对查询构造器和 Eloquent ORM 有效。

## 使用多个数据库连接

当使用多个数据连接时，你可以通过 `DB` 假面的 `connection` 方法来访问这些连接。传递到 `connection` 方法中的 `name` 应该与你的 `config/database.php` 配置文件中的连接相匹配：

```php
$users = DB::connection('foo')->select(...);
```

你也可以在连接实例上调用 `getPdo` 方法来获取底层的 PDO 实例。