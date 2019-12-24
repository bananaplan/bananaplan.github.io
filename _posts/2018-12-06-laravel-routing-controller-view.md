---
title: Laravel 基础教程【2】 路由、控制器和视图的基本使用
tags: Laravel
---

<div>{%- include extensions/bilibili.html id='37571896' -%}</div>

## 路由
#### 什么是路由
路由其实就是网站所有操作的入口配置文件。比如你的网站支持登录、注册、发表文章、留言、点赞等功能，那么这些功能的通过哪些 URL 能访问到，分别由哪些程序处理，这些配置，都是写在路由里面的。<!--more-->通过一个基于 Laravel 项目的路由文件，就会一目了然这个项目所有功能的入口网址和对应的程序处理模块，然后就可以一步一步的梳理出最终的业务处理流程。

#### 路由的最基本用法
那么，如何定义一个最简单的路由呢？很简单，把路由的配置写在 `routes/web.php` 文件里即可，在这个文件里，定义的都是跟网站功能模块相关的路由，而在 `routes` 目录下，你也会发现还有其他的路由配置文件，这是给其他的场景准备的，比如，你用 Laravel 写的不是网站项目，而只是为 App 提供 `API` 接口，那么，可以把路由配置，写在 `api.php` 这个配置文件中。

下面定义了一个最简单的路由：

```php
Route::get('hello', function () {
    return 'Hello World';
});
```

这个路由，分3部分组成：路由可接收的请求方式、请求地址 和 处理该请求的程序代码。

- `Route::get()` 方法，定义了这个路由只接收 **GET** 方式的请求
- `hello` 定义了，通过 `http://xxx.xx/hello` 可以访问这个程序模块
- `function () {}` 而这个匿名函数，则定义了，当你通过 `http://xxx.xx/hello` 访问时，由函数内的代码去处理请求，响应结果

所以，上面我们定义的路由，如果你定义的虚拟域名是 `blog.me`，当你通过 `http://blog.me/hello` 访问时，会在网页上显示 `Hello World`。

#### 路由支持的请求方式
如上所示的小例子，定义的路由支持 `GET` 请求方式，那么还能支持哪些请求方式呢？

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

你能想到的 HTTP 的请求方式，都是支持的。那还有一个小问题，如果一个路由，既想支持 get 请求，也想支持 post 请求，或者，要支持以上所有的请求方式，哪该怎么定义呢？文字写起来费劲啰嗦，请看视频的讲解演示。

#### 路由参数
好了，我们知道了一个基本路由的组成部分和定义方法，下面我们来看一看如何在路由中定义参数。比如，有这样一个路由：

```php
Route::get('user', function () {
	$id = 获得 Get 请求参数中的 id 的值;
	return 'Hello World, your user id is ' . $id;
});
```

那么，当我们通过 `http://blog.me/user?id=101` 访问时，会在网页上显示：Hello World, your user id is 101，这是因为我们通过请求参数 `id` 把 `101` 传给了路由，而在路由内部的匿名函数中取出 Get 请求参数，再显示出来了。

但这不是我们要说的路由参数！！！我们看看这两个网址的优雅程度：
`http://blog.me/user?id=101`
`http://blog.me/user/101`

很显然，是第二个优雅简洁，那么要达到这种效果，就要给我们的路由定义参数：

```php
Route::get('user/{id}', function ($id) {
	return 'Hello World, your user id is ' . $id;
});
```

你瞧，为路由定义参数非常简单，用 `{xxx}` 就可以，参数会作为函数参数，自动传给处理函数，进行下面的处理，这种方式不但使得 `URL` 看起来更优雅，也使得代码可读性更高。

当然，在官方文档上，会详细讲路由参数还能有哪些变化，这些，我们放在日后更加熟练之后，再做详解。

## 控制器
#### 控制器是干嘛滴
有了路由，把逻辑代码写在其中，就可以实现功能，那控制器还要来干嘛？

你想，一个网站功能如此庞大，怎么可能把所有的实现代码，都写在路由里！！！路由只是一个地图，按图索骥，能找出脉络、线索，但地图里没有真正的高楼大厦、马路大桥，所以，真正的功能实现代码，它们通常还是要放到控制器里的。

**MVC** 的模式，我就不再啰嗦了，我们通过 路由 -> 控制器 -> 模型（处理数据库读写）-> 视图，最终展现结果的方式，来实现网站逻辑流程，而控制器一般就是放逻辑控制代码的地方。

#### 如何定义控制器
在 Laravel 中定义控制器的方式，与其他框架并无太大区别，写个类，继承框架的父控制器类，实现自己的控制器方法，比如，我们把上面路由中的逻辑代码，挪到控制器中来：

```php
namespace App\Http\Controllers;

class UserController extends Controller {
	public function index ($id) {
		return 'Hello World, your user id is ' . $id;
	}
}
```

而路由的定义，也要修改为：
```php
Route::get('user/{id}', 'UserController@index');
```

原来路由里的匿名函数，由 `UserController@index` 替代了，表明该路由将交给 `UserController` 控制器的 `index` 方法来处理，响应结果。你看，我们移交了代码逻辑的处理权，路由就真的变成了一张地图，而地图上标示的高楼大厦，就交给控制器去构建吧。

我们不但可以手动创建控制器类，还可以用 Laravel 内置的 `artisan` 命令，来自动创建，具体操作，请参见视频。

#### 控制器参数
很显然，与路由参数一样，以此类推、举一反三，我就不啰嗦的。

## 视图
好了，至此，我们讲了路由 和 控制器，先把 模型层 搁下，下面我们来看看视图页面，这样就可以基本构建出一个最基础的网页出来。

视图就是页面，在视图文件里，写 `HTML`，通过漂亮的网页，把从控制器端传过来的数据，显示在网页上。

#### 如何写一个最简单的视图页面
很简单，在 Laravel 中，视图要以 `*.blade.php` 结尾，放置在 `resources\views` 目录下，所以，我们可以在该目录或子目录下，新建视图文件，写我们的页面，比如在 `resources\views` 下，新建 `hello.blade.php`

{% raw %}
```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello World</title>
</head>
<body>
    <p>Hello World!</p>
    <p>Your user id is {{ $id }}</p>
</body>
</html>
```

而原来的控制器要修改为：
```php
namespace App\Http\Controllers;

class UserController extends Controller {
	public function index ($id) {
		return view('hello', ['id' => $id]);
	}
}
```

在修改后的控制器里，我们不再 `return` 返回字符串了，而是 `return view();` 返回了一个 `view()` 函数结果，这就会输出你视图模板里的 `HTML` 内容。你还会发现，我们把控制器里的 `$id` 参数值，也传给了视图模板，在模板里，以 `{{ $id }}` 的形式读取变量值，显示出来用户id。

{% endraw %}

#### 如何提交一个表单给控制器处理
此部分文字不可描述，请见视频。

## 结语
本节，我们实现了一个最基本的由 **路由** -> **控制器** -> **视图** 的基本模式，虽然中间忽略了 **模型**，但我们初步掌握了用 Laravel 写一个网页的基本流程了，一种掌控感油然而生，以后只是逐步深入的问题了。

下期，准备写个小 demo，做个注册、登录，会牵扯到 中间件 和 命名路由 等稍微细一些的东西吧，这是初步构想。

## 修订 2018-12-07
本期视频遗漏了同时支持多种请求方式的路由规则写法，并且由于时间关系，没有演示表单提交，所以，预计后面会录一期一个小的拾遗视频，完成后会贴在本文中。
