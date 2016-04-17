title: laravel-boilerplate 源码学习 (一)
date: 2016-04-09 09:15:58
tags: [php, laravel]
---

# laravel-boilerplate

读laravel-boilerplate源码，分析各模块编码方式，分析代码整理方式，分析目录结构，以提高自我编码水平，提高代码整理能力。提高模块化架构能力，提高面向对象分析能力。

## 目录结构分析

### 路由

- 路由分组
- 结构化目录
- 独立路由以require的形式注入
- 保持组命名空间与目录一致
- 命名路由

#### 路由分组 

所有路由入口在 `app\Http\routes.php`文件中，在该文件中定义分组路由:
- web端路由组 （包含 前端路由组 及 语言选择路由组）
- 后端路由组

#### 结构化目录

建立`app\Http\Routes\`文件目录，用于存放各模块组路由并建立相应的文件目录：
- Backend
- Frontend
- Language

在模块组目录下细化模块路由

#### 独立路由以require的形式注入

相应的模块组目录下的细化模块路由文件 在`app\Http\routes.php` 文件中相应路由组中以require的形式注入:

``` php
/**
 * Backend Routes
 * Namespaces indicate folder structure
 * Admin middleware groups web, auth, and routeNeedsPermission
 */
Route::group(['namespace' => 'Backend', 'prefix' => 'admin', 'middleware' => 'admin'], function () {
    /**
     * These routes need view-backend permission
     * (good if you want to allow more than one group in the backend,
     * then limit the backend features by different roles or permissions)
     *
     * Note: Administrator has all permissions so you do not have to specify the administrator role everywhere.
     */
    require (__DIR__ . '/Routes/Backend/Dashboard.php');
    require (__DIR__ . '/Routes/Backend/Access.php');
    require (__DIR__ . '/Routes/Backend/LogViewer.php');
});

```

#### 保持组命名空间与目录一致

因为组模块下的相应路由文件中会使用到`控制器`来指定路由，并且在 `boilerplate`中控制器也是以模块组的目录结构来建立的, 
而该模块组中的命名空间是以`composer.json`文件中的 `autoload` `psr-4` 的规范来定义的, 
比如 `app/Http/Controllers/Backend/DashboardController.php` 的命名空间就是`App\Http\Controllers\Backend`,  
为了在子路由下不必重复的编写命名空间引入控制器，可以在路由组中加入`namespace`来指定当前组所在的命名空间.

``` php
  // 模块组路由中
  Route::group(['namespace' => 'Frontend'], function () {
    // 当前组的命名空间为 /App/Http/Controllers/Frontend
        require (__DIR__ . '/Routes/Frontend/Frontend.php');
        require (__DIR__ . '/Routes/Frontend/Access.php');
    });
```

``` php
  // Routes/frontend/frontend.php
  // 引入FrontendController控制器 该控制器对应的命名空间为 /App/Http/Controllers/Frontend
  // 如果模块组中未定义命名空间 则自动检索 /App/Http/Controllers 命名空间 会提示找不到，因为该文件并不在这个命名空间下
  Route::get('macros', 'FrontendController@macros')->name('frontend.macros');

```

#### 命名路由

对详细路由进行命名，方便在视图中进行跳转

``` php
// router
Route::get('/', 'FrontendController@index')->name('frontend.index');

// view
<a href="{!! route('frontend.index') !!}" class="logo">{!! app_name() !!}</a>
```

### 控制器

**控制器**的主要作用是便于管理路由，对相应路由的业务逻辑的实现集中在这里处理或传递。

`boilerplate`在控制器的目录结构上也进行了整理，在`app\Http\Controllers`目录下建立的三个目录对控制器进行了归类整理：
- Backend
- Frontend
- Language

在这些目录下根据业务需求建立另外一些归类目录，便于控制器的集中化管理。
例如目录`app\Http\Controllers\Frontend\User`目录集中管理前端的用户相关的路由的实现:
- DashboardController.php  用户中心仪表盘控制器
- ProfileController.php 用户文件控制器 （实现用户个人文档操作路由的相关业务逻辑）

在控制器中一般我们只根据路由做业务的接入和结果的输出，而不是仅仅将业务的处理逻辑全扔进控制器控制的函数中，

如果是这样的话，控制器会非常的臃肿，而且相关业务功能不能做到很好的复用。

如果需求变更的话，我们不得不删除原先的业务逻辑重新编写新的业务逻辑。

`boilerplate`中使用`contracts（契约接口）`的形式来管理这些逻辑的复用， 并且这种做法，提供了很好的可扩展性，如果业务变更，我们只需要提供一个实现了`contracts（契约接口）`的类文件就可以了，而不需要去修改之前业务的类文件。

让我们来看一下 `boilerplate`中的控制器：

``` php
<?php

namespace App\Http\Controllers\Backend\Access\User;

use App\Http\Controllers\Controller;
use App\Http\Requests\Backend\Access\User\EditUserRequest;
use App\Http\Requests\Backend\Access\User\MarkUserRequest;
use App\Http\Requests\Backend\Access\User\StoreUserRequest;
use App\Http\Requests\Backend\Access\User\CreateUserRequest;
use App\Http\Requests\Backend\Access\User\UpdateUserRequest;
use App\Http\Requests\Backend\Access\User\DeleteUserRequest;
use App\Http\Requests\Backend\Access\User\RestoreUserRequest;
use App\Repositories\Backend\Access\User\UserRepositoryContract;
use App\Repositories\Backend\Access\Role\RoleRepositoryContract;
use App\Http\Requests\Backend\Access\User\ChangeUserPasswordRequest;
use App\Http\Requests\Backend\Access\User\UpdateUserPasswordRequest;
use App\Http\Requests\Backend\Access\User\PermanentlyDeleteUserRequest;
use App\Http\Requests\Backend\Access\User\ResendConfirmationEmailRequest;
use App\Repositories\Backend\Access\Permission\PermissionRepositoryContract;
use App\Repositories\Frontend\Access\User\UserRepositoryContract as FrontendUserRepositoryContract;

/**
 * Class UserController
 */
class UserController extends Controller
{
    /**
     * @var UserRepositoryContract
     */
    protected $users;

    /**
     * @var RoleRepositoryContract
     */
    protected $roles;

    /**
     * @var PermissionRepositoryContract
     */
    protected $permissions;

    /**
     * @param UserRepositoryContract       $users
     * @param RoleRepositoryContract       $roles
     * @param PermissionRepositoryContract $permissions
     */
    public function __construct(
        UserRepositoryContract $users,
        RoleRepositoryContract $roles,
        PermissionRepositoryContract $permissions
    )
    {
        $this->users       = $users;
        $this->roles       = $roles;
        $this->permissions = $permissions;
    }

    /**
     * @return mixed
     */
    public function index()
    {
        return view('backend.access.index')
            ->withUsers($this->users->getUsersPaginated(config('access.users.default_per_page'), 1));
    }

    /**
     * @param  CreateUserRequest $request
     * @return mixed
     */
    public function create(CreateUserRequest $request)
    {
        return view('backend.access.create')
            ->withRoles($this->roles->getAllRoles('sort', 'asc', true))
            ->withPermissions($this->permissions->getAllPermissions());
    }

    /**
     * @param  StoreUserRequest $request
     * @return mixed
     */
    public function store(StoreUserRequest $request)
    {
        $this->users->create(
            $request->except('assignees_roles', 'permission_user'),
            $request->only('assignees_roles'),
            $request->only('permission_user')
        );
        return redirect()->route('admin.access.users.index')->withFlashSuccess(trans('alerts.backend.users.created'));
    }

    /**
     * @param  $id
     * @param  EditUserRequest $request
     * @return mixed
     */
    public function edit($id, EditUserRequest $request)
    {
        $user = $this->users->findOrThrowException($id, true);
        return view('backend.access.edit')
            ->withUser($user)
            ->withUserRoles($user->roles->lists('id')->all())
            ->withRoles($this->roles->getAllRoles('sort', 'asc', true))
            ->withUserPermissions($user->permissions->lists('id')->all())
            ->withPermissions($this->permissions->getAllPermissions());
    }

    /**
     * @param  $id
     * @param  UpdateUserRequest $request
     * @return mixed
     */
    public function update($id, UpdateUserRequest $request)
    {
        $this->users->update($id,
            $request->except('assignees_roles', 'permission_user'),
            $request->only('assignees_roles'),
            $request->only('permission_user')
        );
        return redirect()->route('admin.access.users.index')->withFlashSuccess(trans('alerts.backend.users.updated'));
    }

    /**
     * @param  $id
     * @param  DeleteUserRequest $request
     * @return mixed
     */
    public function destroy($id, DeleteUserRequest $request)
    {
        $this->users->destroy($id);
        return redirect()->back()->withFlashSuccess(trans('alerts.backend.users.deleted'));
    }

    /**
     * @param  $id
     * @param  PermanentlyDeleteUserRequest $request
     * @return mixed
     */
    public function delete($id, PermanentlyDeleteUserRequest $request)
    {
        $this->users->delete($id);
        return redirect()->back()->withFlashSuccess(trans('alerts.backend.users.deleted_permanently'));
    }

    /**
     * @param  $id
     * @param  RestoreUserRequest $request
     * @return mixed
     */
    public function restore($id, RestoreUserRequest $request)
    {
        $this->users->restore($id);
        return redirect()->back()->withFlashSuccess(trans('alerts.backend.users.restored'));
    }

    /**
     * @param  $id
     * @param  $status
     * @param  MarkUserRequest $request
     * @return mixed
     */
    public function mark($id, $status, MarkUserRequest $request)
    {
        $this->users->mark($id, $status);
        return redirect()->back()->withFlashSuccess(trans('alerts.backend.users.updated'));
    }

    /**
     * @return mixed
     */
    public function deactivated()
    {
        return view('backend.access.deactivated')
            ->withUsers($this->users->getUsersPaginated(25, 0));
    }

    /**
     * @return mixed
     */
    public function deleted()
    {
        return view('backend.access.deleted')
            ->withUsers($this->users->getDeletedUsersPaginated(25));
    }

    /**
     * @param  $id
     * @param  ChangeUserPasswordRequest $request
     * @return mixed
     */
    public function changePassword($id, ChangeUserPasswordRequest $request)
    {
        return view('backend.access.change-password')
            ->withUser($this->users->findOrThrowException($id));
    }

    /**
     * @param  $id
     * @param  UpdateUserPasswordRequest $request
     * @return mixed
     */
    public function updatePassword($id, UpdateUserPasswordRequest $request)
    {
        $this->users->updatePassword($id, $request->all());
        return redirect()->route('admin.access.users.index')->withFlashSuccess(trans('alerts.backend.users.updated_password'));
    }

    /**
     * @param  $user_id
     * @param  FrontendUserRepositoryContract $user
     * @param  ResendConfirmationEmailRequest $request
     * @return mixed
     */
    public function resendConfirmationEmail($user_id, FrontendUserRepositoryContract $user, ResendConfirmationEmailRequest $request)
    {
        $user->sendConfirmationEmail($user_id);
        return redirect()->back()->withFlashSuccess(trans('alerts.backend.users.confirmation_email'));
    }
}
```

我们会发现`UserController`中的路由控制函数都非常的精简，每个控制函数都只有仅仅数行代码。
我们以`store`新增用户函数来追踪一下代码到底是怎么运行的。
该方法会在路由提交新增用户请求时进行处理，先经过依赖注入`StoreUserRequest`（文件地址`app\Http\Requests\Backend\Access\User\StoreUserRequest.php`）实例来判断Post过来的数据是不是经得起验证，如果数据是合法的，就执行该控制函数下的代码。
我们看下`store`函数会发现主要实现的业务逻辑都集中在`$this->user`实例的`create`方法中，
而`$this->user`是`Usercontroller`初始化时直接通过依赖注入构造的`UserRepositoryContract`类的实例，
该实例在 `AccessServiceProvider`中的`registerBindings`方法注册，实际上是从容器中提取的`EloquentUserRepository`的实例。

```php
  // app/Providers/AccessServiceProvider.php
  /**
     * Register service provider bindings
     */
    public function registerBindings()
    {
        $this->app->bind(
            \App\Repositories\Frontend\Access\User\UserRepositoryContract::class,
            \App\Repositories\Frontend\Access\User\EloquentUserRepository::class
        );

        $this->app->bind(
            \App\Repositories\Backend\Access\User\UserRepositoryContract::class,
            \App\Repositories\Backend\Access\User\EloquentUserRepository::class
        );

        $this->app->bind(
            \App\Repositories\Backend\Access\Role\RoleRepositoryContract::class,
            \App\Repositories\Backend\Access\Role\EloquentRoleRepository::class
        );

        $this->app->bind(
            \App\Repositories\Backend\Access\Permission\PermissionRepositoryContract::class,
            \App\Repositories\Backend\Access\Permission\EloquentPermissionRepository::class
        );

        $this->app->bind(
            \App\Repositories\Backend\Access\Permission\Group\PermissionGroupRepositoryContract::class,
            \App\Repositories\Backend\Access\Permission\Group\EloquentPermissionGroupRepository::class
        );

        $this->app->bind(
            \App\Repositories\Backend\Access\Permission\Dependency\PermissionDependencyRepositoryContract::class,
            \App\Repositories\Backend\Access\Permission\Dependency\EloquentPermissionDependencyRepository::class
        );
    }
```
我们看下`EloquentUserRepository`这个类，这个类才是真正的实现业务逻辑的类，它实现了`create`业务方法生成一个新的用户。
如果我们业务变更，需求用另一种实现来生成另外一种用户,这时候我们只需建立一个`SomeOtherUserRepository`并实现`UserRepositoryContracts`，然后再修改在容器中的注册就可以了，而完全不需要去修改`EloquentUserRepository`中的实现了。

### 模型
**boilerplate**将模型都集中在了`app\Models`目录下进行统一管理。

这里我们主要分析一下用户模型，而用户相关的文件存放在 `app\Models\Access\User\`目录下，在该目录下`User.php`就是用户模型，该模型继承了`Illuminate\Database\Eloquent\Model`。

我们发现`User.php`文件下内容非常的清爽，而`User`目录下建立了`Traits`目录用来存放, 而`User.php`里也使用了trait来引用不同的文件。 

**trait**是自php 5.4.0 起实现代码复用的一个方法，它和类相似，但自身不能实例化，类似于前端语言里的`[mixin]`方式复用方法。

**boilerplate**将用户模型中不同职责的方法分别独立出了一个trait文件，方便了功能方法的复用，文件的职责更加明确.

### helpers

`app/helpers.php`文件中封装了一些全局方法，该文件在`composer.json`文件中定义了`autoload`, 在由composer解析生成依赖文件后，会在程序启动中进行自动加载. 一些常用的方法可以在这里定义简便形式。


## 技能get

### Seed
``` php
    Model::unguard();

        if (env('DB_CONNECTION') == 'mysql') {
          // 取消外键约束
            DB::statement('SET FOREIGN_KEY_CHECKS=0;'); 
        }

        $this->call(AccessTableSeeder::class);

        if (env('DB_CONNECTION') == 'mysql') {
          // 使外键约束生效
            DB::statement('SET FOREIGN_KEY_CHECKS=1;');
        }

     Model::reguard();

```

## 总结

### 目录结构

优秀的目录结构使代码结构更清晰，模块职责明确，这样在未来新增功能或维护功能时，可以非常迅速的定位到相应的的位置，提高了扩展的效率，提高了可维护性。

### 业务实现

考虑需求变更的可能性，提高业务的复用性，可扩展性。使用 `contracts`的形式去规划业务，可能刚开始编写代码的时候会多做很多事情，但是后期的维护上却会节省很大的效率。

### 研究开源项目源码的好处

记得以前玩游戏的时候，玩不过了怎么办？我们经常做的就是研究攻略找秘籍，编程也是一样的，读一些优秀的开源项目的源码，可以让我们学习到非常多的东西，提高我们的编程视角，往往能够收获颇多。
