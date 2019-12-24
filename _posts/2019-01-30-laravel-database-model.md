---
title: Laravel 基础教程【5】Eloquent Model 数据库操作基本用法
tags: Laravel
---

现在，我们已经学会了如何定义路由、控制器，编写简单的页面模版，还有使用中间件来过滤请求数据，并且把使用了相同中间件的路由，归为一个路由分组里。

好了，下面我们开始要操作数据库了，来完成我们 blog 项目示例里的注册和登录，学习如何用 Laravel 的 Eloquent Model 来 插入 和 查询 数据库，学习 Model 的基本应用。

<!--more-->

## 视频教程
正在录制【然而我太监了 :ghost:】

## Laravel 中如何操作数据库
#### 先配置 .env
修改 .env 文件中，关于数据库连接的配置内容，如：

```php
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=blog
DB_USERNAME=root
DB_PASSWORD=root
```

这样，才能正确连接上数据库，来进行数据库的读写操作。

#### Laravel 操作数据库的三种方式
Laravel 中，有三种方式来操作数据库，在项目中，我们可以灵活的、自由运用，但大多数情况下，我们都是用 Eloquent ORM 的形式来操作数据库。

1. raw SQL
	用原始 SQL 语句的方式，来读写数据库。
	
2. QueryBuilder
	用 Laravel 提供的 QueryBuilder 接口，通过链式操作的方式，来读写数据库。
	
3. Eloquent ORM
	自定义 Model 类，封装读写操作，在控制器中调用。

## Model 是干嘛的
不论你用什么编程语言、什么框架来构建一个网站，都需要操作数据库。而编写数据库增、删、改、查的逻辑实现代码，一般不会凌乱的写在控制器的里，而是先以 Model 类的形式，编写针对某个表的数据库操作逻辑，再在控制器里调用 Model 类的实例，去操作数据。这实际上就是封装的体现，使得我们的代码更加简洁、可读和可维护性更高。

所以，Laravel 里的 Eloquent Model，与其他框架中的 Model 概念是一致的，用于封装数据库的逻辑操作代码。

## 如何定义 Model
定义 Model，就是定义一个类，这个类关联某张数据表，在其中写上数据库的操作逻辑代码。但我们无需从头手动创建这个类文件，可以用 `artisan` 命令来快捷创建：

```php
php artisan make:model UserModel
```

这会在 `app/` 目录下，创建一个名为 UserModel 的类文件。

下面，我们需要将此 Model 关联到某张表，很简单，添加 `protected $table` 成员变量即可：

```php
protected $table = 'blog_users';
```

哎呀，我们的数据库中还没有一个叫 blog_users 的表，下面我们来建表：

```sql
CREATE TABLE blog_users (
  id         INT AUTO_INCREMENT
    PRIMARY KEY,
  username   VARCHAR(20)  NOT NULL,
  password   VARCHAR(32)  NOT NULL,
  nickname   VARCHAR(20)  NULL,
  email      VARCHAR(200) NULL,
  tel        VARCHAR(20)  NULL,
  created_at DATETIME     NULL,
  updated_at DATETIME     NULL
);
```

如果你用 Laravel 的 Eloquent Model 来操作数据表的话，默认情况下，你需要保证表中要有这3个字段：`id` INT 类型的自增id，`created_at` 和 `updated_at` DATETIME 类型的日期字段。Laravel 会自动维护这3个字段，不论是 insert 还是 update 操作时。

如果，你需要 `created_at` 和 `updated_at` 用 INT 类型 时间戳保存数据的话，你需要在 Model 类中，添加 `protected $dateFormat = 'U';`。而如果你压根就不想要这2个字段，在数据表中把它们删掉后，在 Model 中添加 `public $timestamps = false;`即可。

## 如何使用 Model
现在，我们要在注册和登录的控制器中，调用 `UserModel` 来保存和查询用户数据了。修改 `UserController` 控制器的 `doRegister()` 方法，调整如下：

```php
public function doRegister(Request $request)
{
	$model = new UserModel();
	$model->add($request);

	return redirect('login');
}
```

很明显，`doRegister()` 注册方法调用的是 `UserModel` 类中的 `add()` 方法，来添加用户。那么，我们需要在 `UserModel` 类中，加入 `add()` 方法，来完成添加用户的操作，如下：

```php
use Illuminate\Database\Eloquent\Model;

class UserModel extends Model
{
    protected $table = 'blog_users';

    public function add($request) {
        $this->username = $request->username;
        $this->password = md5($request->password);
        $this->nickname = $request->nickname;
        $this->email = $request->email;
        $this->tel = $request->tel;

        return $this->save();
    }
}
```

运行原理很简单，在控制器的 `doRegister()` 方法中，实例化了 `UserModel`，然后调用它的实例对象的 `add()`，那么 `add()` 方法里面的 `$this`，就代表前面的 `UserModel` 实例对象，把要保存的数据统统赋给对象的数据表对应属性，最后 `$this->save()` 调用实例对象的 `save()` 方法，即可保存成功到 `blog_users` 表。

最后，我们修改登录的 `doLogin()` 方法，在其中去查询用户，因为查询的代码比较简单，我们可以直接将查询代码写到控制器里，而像保存这样的有很多行代码的逻辑，我们还是要封装到 `UserModel` 里的。下面是登录控制器方法的调整：

```php
public function doLogin(Request $request)
{
	$user = UserModel::where('username', $request->username)->first();

	if ($request->username == $user['username'] && md5($request->password) == $user['password']) {
		session(['user' => $user]);
		return redirect('about');
	}

	return '用户名或密码错误';
}
```

## 结束
本期，我们比较完整的讲解了，Laravel 框架的 Eloquent Model 操作数据库的流程，从为什么要用 Model，到如何创建、编写数据处理逻辑代码，最后如何在控制器中调用，都做了详细的演示，希望我能用浅显的语言，讲明白每个要点。
