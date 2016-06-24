title: laravel 基础教程 —— 迁移
date: 2016-06-23 22:18:46
tags: [php, laravel]
---

# 数据库：迁移

## 简介

迁移就像是为数据库提供的版本控制，这允许你的团队可以轻易的修改和共享应用数据库结构。迁移通常配合 laravel 的结构生成器来轻松的构建应用中的数据库结构。

Laravel 的 `Schema` 假面提供了创建和操作数据库的相关支持。它为所有 Laravel 支持的数据库系统提供了丰富流畅的相同的 API 接口。

## 生成迁移

你可以使用 `make:migration` Artisan 命令来生成迁移：

```php
php artisan make:migration create_users_table
```

新生成的迁移将会被存放在 `database/migrations` 目录下，每个迁移文件都包含了创建时的时间戳，这样就可以指导 Laravel 按序的进行迁移操作。

你也可以使用 `--table` 和 `--create` 选项来指导迁移所创建的表的名称。这些选项只是简单的填充生成的迁移文件为指定的表：

```php
php artisan make:migration add_votes_to_users_table --table=users

php artisan make:migration create_users_table --create=users
```

如果你想要自定义生成迁移的路径，你可以使用 `--path` 选项。所提供的路径应该是基于应用根目录的相对路径。

## 迁移结构

迁移类中包含了两个方法：`up` 和 `down`。`up` 方法用来添加新的表，列或者数据库中的索引。而 `down` 方法应该简单的执行 `up` 方法的逆操作。

在这个两个方法里你可以使用 Laravel 的结构生成器来进行表的创建和修改。对于所有的 `Schema` 生成器中可用的方法，请参考 [文档](https://laravel.com/docs/5.2/migrations#creating-tables)。比如，让我们来看一个简单的创建一个 `flights` 表的迁移：

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateFlightsTable extends Migration
{
  /**
   * Run the migrations.
   *
   * @return void
   */
   public function up()
   {
     Schema::create('flights', function (Blueprint $table) {
       $table->increments('id');
       $table->string('name');
       $table->string('airline');
       $table->timestamps();
     });
   }

   /**
    * Reverse the migrations.
    *
    * @return void
    */
    public function down()
    {
      Schema::drop('flights');
    }
}
```

## 执行迁移

你可以使用 `migrate` Artisan 命令来执行迁移操作。如果你使用 [Homestead virtual machine](https://laravel.com/docs/5.2/homestead)，你应该在你的虚拟机中运行如下命令：

```php
php artisan migrate
```

如果你在执行迁移的过程中出现 “class not found” 错误，你可以尝试运行 `composer dump-autoload` 命令然后重新运行迁移命令。

**在生成环境中强制迁移操作**

有时候迁移的操作是有害的，这意味着你可能会丢失数据。为了保护你在生产环境中不随意的进行迁移操作，Laravel 会提示你确认执行这些操作。你可以使用 `--force` 标识来取消这些提示：

```php
php artisan migrate --force
```

### 回滚迁移

你可以使用 `rollback` 命令来将迁移回滚到上一次的迁移操作。你需要注意的是，这个回滚是对最后一次迁移的批量操作的回滚，它可能会包含多个迁移文件：

```php
php artisan migrate:rollback
```

`migrate:reset` 命令将回滚所有的迁移操作：

```php
php artisan migrate:reset
```

**在同一个命令里执行回滚 / 迁移操作**

`migrate:refresh` 命令会一次回滚所有的迁移操作，然后接着执行 `migrate` 命令。该命令可以有效的重新创建你的整个数据库：

```php
php artisan migrate:refresh

php artisan migrate:refresh --seed
```

## 编写迁移

## 创建表

你可以使用 `Schema` 假面的 `create` 方法来创建一个新的数据库表。`create` 方法接收两个参数，第一个参数为表的名称，第二个参数是一个 `Closure`，该闭包接收一个 `Blueprint` 对象用来对表进行定义：

```php
Schema::create('users', function (Blueprint $table) {
  $table->increments('id'); 
});
```

当然，当创建表时，你也可以使用任意的结构生成器的 [列方法](https://laravel.com/docs/5.2/migrations#creating-columns) 来定义表的列。

**检查表 / 列是否已经存在**

你可以通过使用 `hasTable` 和 `hasColumn` 方法来简单的检查所给定的表或者列是否存在：

```php
if (Schema::hasTable('users')) {
  //
}

if (Schema::hasColumn('users', 'email')) {
  //
}
```

**连接 & 存储引擎**

如果你想在不是默认的数据库连接中执行 schema 的操作，你可以使用 `connection` 方法来选择数据库连接：

```php
Schema::connection('foo')->create('users', function ($table) {
  $table->increments('id'); 
});
```

你也可以在结构生成器中设置 `engine` 属性来指定表的存储引擎：

```php
Schema::create('users', function ($table) {
  $table->engine = 'InnoDB';

  $table->increments('id'); 
});
```

## 重命名 / 删除表

你可以使用 `rename` 方法来重命名已经存在的表：

```php
Schema::rename($from, $to);
```

你可以使用 `drop` 或者 `dropifExists` 方法来删除存在的表：

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

**重命名含有外键的表**

在对表进行重命名之前，你应该先确认表中所有的外键约束在迁移文件中都有一个明确的名称，而不是让 Laravel 分配基于惯例的名称。另外，外键约束名称将引用旧表名。

### 创建列

你可以使用 `Schema` 假面的 `table` 方法来进行已经存在表的更新操作。像 `create` 方法一样，`table` 方法接收两个参数：表的名称和一个接收 `Blueprint` 实例的闭包，我们可以使用 `Blueprint` 实例来在表中添加列：

```php
Schema::table('users', function ($table) {
  $table->string('email'); 
});
```

**可用的列类型**

当然，结构生成器包含了各种的列类型，你可以在构建表时使用它们：

命令                                       | 描述
---                                        | ---
`$table->bigIncrements('id');`             | 使用 "UNSIGNED BIG INTEGER" 类型来设置自增的 ID（主键）
`$table->bigInteger('votes');`              | 使用 BIGINT 类型
`$table->binary('data');`                  | 使用 BLOB 类型
`$table->boolean('confirmed');`            | 使用 BOOLEAN 类型
`$table->char('name', 4);`                 | 使用 CHAR 类型
`$table->date('created_at');`              | 使用 DATE 类型
`$table->dateTime('created_at');`          | 使用 DATETIME 类型
`$table->dateTimeTz('created_at');`        | 使用 DATETIME (和时区) 类型
`$table->decimal('amount', 5, 2);`         | 使用 DECIMAL 类型并伴随一个精度和尺度
`$table->double('column', 15, 8);`         | 使用 DOUBLE 类型并伴随一个精度，总共 15 个有效数组，并且其中 8 个是小数
`$table->enum('choices', ['foo', 'bar']);` | 使用 ENUM 类型
`$table->float('amount');`                 | 使用 FLOAT 类型
`$table->increments('id');`                | 使用 UNSIGNED INTEGER 类型设置自增的 ID（主键）
`$table->integer('votes');`                | 使用 INTEGER 类型
`$table->ipAddress('visitor');`            | 使用 IP 地址类型
`$table->json('options');`                 | 使用 JSON 类型
`$table->jsonb('options');`                | 使用 JSONB 类型
`$table->longText('description');`         | 使用 LONGTEXT 类型
`$table->macAddress('device');`            | 使用 MAC 地址类型
`$table->mediumInteger('numbers');`        | 使用 MEDIUMINT 类型
`$table->mediumText('description');`       | 使用 MEDIUMTEXT 类型
`$table->morphs('taggable');`              | 添加一个 INTEGER 类型的 `taggable_id` 和一个 STRING 类型的 `taggable_type`
`$table->nullableTimestamps();`            | 和 `timestamps()` 一样，除了允许 NULLs
`$table->rememberToken();`                 | 添加 VARCHAR(100) NULL `remember_token`
`$table->smallInteger('votes');`            | 使用 SMALLINT 类型
`$table->softDeletes();`                   | 为软删除添加 `delete_at` 列
`$table->string('email');`                 | 使用 VARCHAR 类型
`$table->string('name', 100);`             | 使用 VARCHAR 类型并伴随一个长度。
`$table->text('description');`             | 使用 TEXT 类型
`$table->time('sunrise');`                 | 使用 TIME 类型
`$table->timeTz('sunrise');`               | 使用 TIME （伴随时区）类型
`$table->tinyInteger('numbers');`          | 使用 TINYINT 类型
`$table->timestamp('added_on');`           | 使用 TIMESTAMP 类型
`$table->timestampTz('added_on');`         | 使用 TIMESTAMP 类型
`$table->timestamps();`                    | 添加 `created_at` 和 `updated_at` 列
`$table->uuid('id');`                      | 使用 UUID 类型

**修改列**

除了上面所列出的列类型，这里也提供了一些其他在添加列的同时增加一些修正的功能。比如，你可以使用 `nullable` 方法来让列可以为 `NULL`:

```php
Schema::table('users', function ($table) {
  $table->string('email')->nullable(); 
});
```

下面列出了一些可用的列的修正方法，这里并没有包含 [index modifiers](https://laravel.com/docs/5.2/migrations#creating-indexes):

变化                      | 描述
---                       | ---
`->first()`               | 将列放在表的首列（仅支持 MySQL)
`->after('column')`       | 将列放置在其他列之后（仅支持 MySQL)
`->nullable()`            | 允许列中存在 NULL 值
`->default($value)`       | 为列指定默认值
`->unsigned()`            | 设置 `integer` 列为 `UNSIGNED`
`->comment('my comment')` | 为列添加注释

### 进行列的修改

**先决条件**

在进行列的修改之前，你需要先在 `composer.json` 文件中添加 `doctrine/dbal` 用来。Doctrine DBAL 类库用来确定列的当前状态，并创建指定列所需调整的 SQL 查询。

**更新列的属性**

`change` 方法允许你修改已存在的列的类型，或者修改列的属性。比如，你可能希望增加字符串类型的列的长度。你可以使用 `change` 方法来让 `name` 列的长度从 25 到 50：

```php
Schema::table('users', function ($table) {
  $table->string('name', 50)->change(); 
});
```

我们也想要将列修改为允许 NULL：

```php
Schema::table('users', function ($table) {
  $table->string('name', 50)->nullable()->chage(); 
});
```

> 注意: 修改表中含有 `enum` 类型的列的表中的任意列，目前都没有得到支持。

**列的重命名**

你可以使用结构生成器的 `renameColumn` 方法来对列进行重命名,在进行重命名之前，你需要确保 `composer.json` 文件中引入了 `doctrine/dbal` 依赖：

```php
Schema::table('users', function ($table) {
  $table->renameColumn('from', 'to'); 
});
```
> 注意: 重命名表中含有 `enum` 类型的列的表，目前都没有得到支持。

### 删除列

你可以使用结构生成器中的 `dropColumn` 方法来删除一列：

```php
Schema::table('users', function ($table) {
  $table->dropColumn('votes'); 
});
```

你也可以传递一个包含了列名称的数组到 `dropColumn` 方法：

```php
Schema::table('users', function ($table) {
  $table->dropColumn(['votes', 'avatar', 'location']); 
});
```

> 注意：如果你删除的是 SQLite 数据库里的列，你需要先在 `composer.json` 文件里引入 `doctrine/dbal` 依赖，并且执行 `composer update` 命令来安装这个类库。
> 注意： 在使用 SQLite 数据库时，使用单个迁移操作来删除或者修改多个列是行不通的。

### 建立索引

结构生成器支持多种类型的索引。首先，让我们看一个简单的例子，这个例子指定列的值应该是唯一的。为了创建索引，我们可以简单的在列的定义中链式调用 `unique` 方法：

```php
$table->string('email')->unique();
```

另外，你也可以在定义完列之后添加索引：

```php
$table->unique('email');
```

你也可以通过传递一个列名的数组到 `index` 方法来创建一个复合的索引：

```php
$table->index(['account_id', 'created_at']);
```

Laravel 会自动的生成一个合理的索引名称，但是你也可以传递第二个参数来指定索引名称：

```php
$table->index('email', 'my_index_name');
```

**可用的索引类型**

命令                                        | 描述
---                                         | ---
`$table->primary('id');`                    | 添加主键
`$table->primary(['first', 'last']);`       | 添加复合键
`$table->unique('email');`                  | 添加唯一的索引
`$table->unique('state', 'my_index_name');` | 添加自定义的索引名称
`$table->index('state');`                   | 添加基础的索引

### 删除索引

你需要指定一个索引的名称才能删除索引，默认的 Laravel 会自动的分配一个合理的名称到索引。它只是简单的连接表名，索引的列名，索引的类型。这里有一些示例：

命令                                        | 描述
---                                         | ---
`$table->dropPrimary('users_id_primary');`  | 从用户表中删除主键
`$table->dropUnique('users_email_unique');` | 从用户表中删除唯一的索引
`$table->dropIndex('geo_stage_index');`     | 从 geo 表中删除基础的索引

如果你传递一个列名所组成的数组到删除索引的方法中，常规的索引的名称将根据表名，列名和键的类型来生成：

```php
Schema::table('geo', function ($table) {
  $table->dropIndex(['state']); // Drops index 'geo_state_index' 
});
```

### 外键约束

Laravel 也对创建外键约束提供了支持，外键约束用于强制参照表中数据的完整性。比如，让我们在 `posts` 表中定义一个 `user_id` 列，并且关联 `users` 表中的 `id` 列：

```php
Schema::table('posts', function ($table) {
  $table->integer('user_id')->unsigned();

  $table->foreign('user_id')->references('id')->on('users'); 
});
```

你也可以指定约束的“关于删除”和“更新”属性所需要的操作：

```php
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
```

你可以使用 `dropForegin` 方法来移除外键。外键约束使用了和索引相同的命名方式。所以，我们会串联表名，列名，和 "_foreign" 后缀：

```php
$table->dropForeign('posts_user_id_foreign');
```

或者你也可以通过传递一个包含列名的数组，它会自动的使用惯例命名来进行外键的移除：

```php
$table->dropForeign(['user_id']);
```

你也可以在你的迁移中使用下面的方法来开启或者关闭外键约束：

```php
Schema::enableForeignKeyConstraints();

Schema::diableForeignKeyConstraints();
```
