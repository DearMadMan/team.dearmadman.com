title: laravel 基础教程 —— 文件系统
date: 2016-06-07 21:22:06
tags: [php, laravel]
---

# 文件系统/云存储

## 简介

laravel 提供了一个强大的文件系统的抽象，这得益于 Frank de Jonge 所开发的 [Flyststem](https://github.com/thephpleague/flysystem) PHP 包。laravel 的文件系统提供了对一些存储驱动的支持，它们包括本地文件系统，Amazon S3，Rackspace 云存储。更为奇妙的是，它可以通过存储配置选项来切换这些存储系统，因为 laravel 对它们提供了统一的 API 接口。

## 配置

文件系统的配置选项存储在 `config/filesystems.php` 文件中。在这个文件中你可以对所有的磁盘进行配置。每个磁盘选项包含着其所使用的存储驱动及其存储位置。对于 laravel 默认支持的存储驱动，在这个文件中都有相应的配置示例。所以，你可以简单的在这个文件中修改配置选项就可以使用其强大的功能。

当然，你可以配置多个磁盘，甚至可以使多个磁盘使用相同的驱动。

**公共磁盘**

`public` 磁盘意味着其可以被公开的访问。默认的 `public` 磁盘使用的是 `local` 驱动并且其存储的文件位置是在 `storage/app/public` 目录。如果你想要这个目录下的文件可以在 web 中进行访问，你需要创建一个 `public/storage` 到 `storage/app/public` 的符号链接。这个约定可以使公开访问的文件保持存放在同一个目录中并且可以在使用像 [Envoyer](https://envoyer.io/) 这种无痛持续部署系统时可以方便的共享整个部署过程。

当然，一旦文件被存储并且建立了符号链接。你就可以通过 `asset` 帮助方法来生成文件的 URL：

```php
echo asset('storage/file.txt')
```

**本地驱动**

当使用 `local` 驱动时，你需要知道的是所有的文件操作都是相对于配置文件中的 `root` 选项所定义的目录。默认的这个值设置的是 `storage/app` 目录。因此，下面的方法将文件存储到 `storage/app/file.txt`:

```php
Storage::disk('local')->put('file.txt', 'Contents');
```

**其他驱动先决条件**

在使用 S3 或者 Rackspace 驱动之前，你需要先通过 Composer 来安装适当的包文件：
- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

**FTP 驱动配置**

laravel 的文件系统可以很好的支持 FTP 的集成，但是在默认的配置文件中并没有给出示例。如果你需要配置 FTP 的文件系统，你可以使用下面的配置示例：

```php
'ftp' => [
  'dirver' => 'ftp',
  'host' => 'ftp.example.com',
  'username' => 'your-username',
  'password' => 'your-password',

  // Optional FTP Settings...
  // 'port' => 21,
  // 'root' => '',
  // 'passive' => true,
  // 'ssl' => true,
  // 'timeout' => 30,
],
```

**Rackspace 驱动配置**

laravel 的文件系统可以很好的支持 Rackspace 的集成，但是在默认的配置文件中并没有给出示例。如果你需要配置 Rackspace 文件系统，你可以使用下面的示例：

```php
'rackspace' => [
  'driver' => 'rackspace',
  'username' => 'your-username',
  'key' => 'your-key',
  'container' => 'your-container',
  'endpoint' => 'https://identity.api.rackspacecloud.com/v2.0/',
  'region' => 'IAD',
  'url_type' => 'publicURL',
],
```

## 基础用法

### 获取磁盘实例

`Storage` 假面可以用来和你所配置的磁盘进行交互。比如，你可以使用它的 `put` 方法来将用户的头像图片存储到默认的磁盘。如果你在调用该方法的时候没有在其之前使用 `disk` 方法的话，那么该方法会自动的将头像传递到默认的磁盘：

```php
<?php

namespace App\Http\Controllers;

use Storage;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
  /**
   * Update teh avatar for the given user.
   *
   * @param Request $request
   * @param int $id
   * @return Response
   */
   public function updateAvatar(Request $request, $id)
   {
     $user = User::findOrFail($id);

     Storage::put(
        'avatars/'.$user->id,
        file_get_contents($request->file('avatar')->getRealPath())
      );
   }
}
```

当你使用多个磁盘时，你可以使用 `Storage` 假面的 `disk` 方法来指定访问的磁盘。当然，你可以使用链式的方法来进行持续的操作：

```php
$disk = Storage::disk('s3');

$contents = Storage::disk('local')->get('file.jpg');
```

### 检索文件

`get` 方法可以用来从所给定的文件中检索出其内容。该方法会返回文件的原始字符串内容：

```php
$contents = Storage::get('file.jpg');
```

`exists` 方法可以用来判断所给定的文件是否存在于磁盘中：

```php
$exists = Storage::disk('s3')->exists('file.jpg');
```

### 文件 URLs

当使用 `local` 或者 `s3` 驱动时，你可以使用 `url` 方法来获得给定文件的 URL。如果你使用的是 `local` 驱动，那它只会简单的在给定的路径前增加 `/storage` 前缀以返回相对的文件路径。如果你使用的是 `s3` 驱动，将会返回完整的远端 RUL：

```php
$url = Storage::url('file1.jpg');
```

> 注意：当使用的是 `local` 驱动时，一定要确保创建了 `public/storage` 到 `storage/app/public` 的符号链接。

**文件元信息**

`size` 方法可以用来获取给定文件的字节大小：

```php
$size = Storage::size('file1.jpg');
```

`lastModified` 方法会返回给定文件的最后修改时间，它使用的 是 UNIX 时间戳：

```php
$time = Storage::lastModified('file1.jpg');
```

### 存储文件

`put` 方法可以用来将文件存储到磁盘。你可以传递一个 PHP 的 `resource` 到 `put` 方法，那么它会使用文件系统的底层流支持。在与大型文件交互时，推荐使用文件流：

```php
Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

`copy` 方法可以用来复制磁盘中已存在的文件到新的位置：

```php
Storage::copy('old/file1.jpg', 'new/file1.jpg');
```

`move` 方法可以被用对磁盘中已存在的文件进行名称的修改或移动到新的位置：

```php
Storage:move('old,file1.jpg', 'new/file1.jpg');
```

**前置或追加内容到文件**

`prepend` 和 `append` 方法允许你轻松的往文件的起始或结束位置加入内容：

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

### 文件可见性

你可以通过使用 `getVisibility` 和 `setVisibility` 方法来进行文件可见性的检索和设置。可见性是跨平台的文件权限的抽象：

```php
Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public');
```

另外，你可以在使用 `put` 方法的同时设置文件的可见性。有效的可见性值是 `public` 和 `private`:

```php
Storage::put('file.jpg', $contents, 'public');
```

### 删除文件

`delete` 方法可以接受一个文件名或者文件名所组成的数组，它将从磁盘中删除相应的文件：

```php
Storage::delete('file.jpg');

Storage::delete(['file1.jpg', 'file2.jpg']);
```

### 目录

**从目录中获取所有文件**

`files` 方法会返回所给定目录中所有的文件所组成的数组。如果你想要在检索到的文件中包含所给定目录的子目录。那么你需要使用 `allFiles` 方法：

```php
$file = Storage::files($directory);

$files = Storage::allFiles($directory);
```

**从给定的目录中获取所有的目录**

`directories` 方法可以返回所给定目录下的所有子目录所组成的数组。另外你可以使用 `allDirectories` 方法递归检索子目录：

```php
$directories = Storage::directories($directory);

// Recursive...

$directories = Storage::allDirectories($directory);
```

**创建目录**

`makeDirectory` 方法将创建给定的目录，包括其所需要的子目录：

```php
Storage::makeDirectory($directory);
```

**删除一个目录**

最后，`deleteDirectory` 方法可以用来删除一个目录，并且其删除磁盘中该目录下所有的文件：

```php
Storage::deleteDirectory($directory);
```

## 自定义文件系统

laravel 的文件系统对几种常见的存储系统提供了开箱即用的支持。事实上，文件系统并没有限制你只使用所提供的这些。你可以自己创建一个适配器来构建一个自定义的驱动去支持你所期望使用的文件存储系统。

你需要创建一个服务提供者来进行自定义文件存储系统的构建。比如 `DropboxServiceProvider`。在提供者的 `boot` 方法中，你需要使用 `Storage` 假面的 `extend` 方法来定义你自己的驱动：

```php
<?php

namespace App\Providers;

use Storage;
use League\Flysystem\Filesystem;
use Dropbox\Client as DropboxClient;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Dropbox\DropboxAdapter;

class DropboxServiceProvider extends ServiceProvider
{
  /**
   * Perform post-registration booting of services.
   *
   * @return void
   */
   public function boot()
   {
     Storage::extend('dropbox', function ($app, $config) {
       $client = new DropboxClient(
        $config['accessToken'], $config['clientIdentifier']
       );      

       return new Filesystem(new DropboxAdapter($client));
     });
   }

   /**
    * Register bindings in the container.
    *
    * @return void
    */
    public function register()
    {
      //
    }
}
```

`extend` 方法中的第一个参数应该是驱动的名称，第二个参数是一个闭包，闭包接受 `$app` 和 `$config` 变量。被解析的闭包必须返回一个 `League\Flysystem\Filesystem` 的实例。`$config` 变量包含了 `config/filesystems.php` 文件中指定磁盘的值。

一旦你创建了服务提供者并且注册了这个扩展，你就可以在 `config/filesystem.php` 配置文件中使用 `dropbox` 驱动了。