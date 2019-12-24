---
title: Laravel 基础教程【4】 中间件、路由分组的用法
tags: Laravel
---

<div>{%- include extensions/bilibili.html id='44250366' -%}</div>

上期视频，通过实战，从头搭建一个博客网站，演示了路由、控制器、视图模板的用法，也包含注册、登录的表单操作，还有 `Cache`、`Session` 的基本使用方式。但是，由于时间关系，我们没有讲 Laravel 中的 中间件 和 路由分组，于是放到本期视频里进行讲解。

<!--more-->

## 视频教程
[https://www.bilibili.com/video/av44250366/](https://www.bilibili.com/video/av44250366/)

## Github 源码
[https://github.com/bananaplan/the-first-laravel-blog-demo](https://github.com/bananaplan/the-first-laravel-blog-demo)

升级日志：

- 新增 CheckLogin Middleware，用于登录验证，只有登录后，才有权访问 about  和 contact 页面
- 将 about 和 contact 路由以路由分组的形式，统一指定中间件

## 什么是中间件
当我们做一个网站的时候，会遇到这样的情况，某些页面需要登录后才能访问，像前台登录后，用户中心里的各种操作；后台登录后，管理员模块的各种操作，都需要先判断登录状态。那如果在每个需要验证的模块当中，都写上同样的代码做判断，肯定是代码冗余了。那么，有没有一种方法，能解决这个痛点呢，那就用 Laravel 的中间件。

其实，Laravel 的中间件，就是用户在页面上操作后，对发出的请求数据的内容，进行过滤，通过的则继续交给控制器处理，没通过的则重定向进行页面提示。如果你之前学过 J2EE 的话，里面有过滤器的概念，都是一个道理。

我们通过中间件写了验证规则后，实际上就将相同的规则，用类的形式定义，统一管理起来了，也就解决了代码冗余的痛点。

## 如何定义中间件
我们可以参考官方文档，中间件其实就是定义一个类，在这个类中，自己实现请求数据的过滤规则。而这个类，可以不用我们手动创建，通过 Laravel 框架的 `artisan` 命令，就能快速的构建出来。

```php
php artisan make:middleware CheckLogin
```

像这样，就创建了一个名为 CheckLogin 的中间件的类文件，这个类文件，会位于 `app/Http/Middleware` 目录下。你会发现，在这个目录下，还有其他的中间件类文件，这些都是 Laravel 内置的，由框架或用户自由使用。

我们通过上面的命令，定义出来的中间件类文件，长这样：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckLogin
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        return $next($request);
    }
}

```

我们需要在 `handle()` 方法中，写上我们的登录验证规则。这个方法，有2个参数，其中一个我们可以会用到：`$request`，我们可以通过这个参数，从用户发来的请求中，拿到数据，针对性的进行验证。但这里，我们只需要从 `Session` 中验证用户是否已经登录过了，因为我们上期的登录操作，是将用户信息，存到 `Session` 里的。

于是，我们在 `handle()` 方法中，加入了简单的验证代码，最终变成了这样：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckLogin
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if (!session('user')) {
            return redirect('/');
        }

        return $next($request);
    }
}

```

只要很简单的判断 `Session` 中，没有用户信息，表明还未登录，则重定向到首页。而验证通过的请求，则由 `return $next($request);` 这行代码，交给后面的控制器，做进一步处理。

## 如何使中间件生效
好了，此时我们定义好了中间件，也就是写好了请求过滤规则，但是我们还尚未指定哪些路由，要经过这个中间件的过滤处理。所以，为了让我们定义的中间件起真正的作用，我们还需要在 `web.php` 路由定义文件中，指定使用这个中间件的路由是哪些个。但在这之前，我们还需要把刚才定义的中间件，加入到 `app/Http/Kernel.php` 类文件的 `$routeMiddleware` 数组中，这个操作非常简单：

```php
protected $routeMiddleware = [
	'auth' => \App\Http\Middleware\Authenticate::class,
	'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
	'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
	'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
	'can' => \Illuminate\Auth\Middleware\Authorize::class,
	'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
	'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
	'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
	'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
	'custom.check_login' => \App\Http\Middleware\CheckLogin::class,
];
```

最后一行 `'custom.check_login' => \App\Http\Middleware\CheckLogin::class,`，就是我们刚添加进去的配置。这样我们新建的中间件，就有了一个方便引用的键名：`custom.check_login`，这个可以自定义。

我们的博客示例网站，有2个模块：About 和 Contact，假如我们希望这2个页面，都需要登录后，才能访问，我们可以给 `about` 和 `contact` 这2个路由定义，指定中间件。像下面这样修改之前的路由定义：

```php
Route::get('about', 'Front\UserController@about')->middleware('custom.check_login');
Route::get('contact', 'Front\UserController@contact')->middleware('custom.check_login');
```

至此，我们就完成了。我们之前写的2个路由：关于我们 `about` 和 联系我们 `contact`，现在都使用了我们自己写的 `CheckLogin` 中间件，进行请求过滤。于是，只有登录后，才能访问这两个页面，否则，将重定向到首页。

## 路由分组
好像完成任务了，不是吗？但是，我们现在又有了一个新的痛点。作为一个有经验的码农，在学习任何新技术的时候，都要考虑这个新技术，是否有相关的方案，能解决以前在业务中遇到的一些痛点，这样才能更有效的学习，带着疑问去思考和探索。那么现在，这个新痛点就是，我们不可能为每一个需要中间件的路由，都手动去修改路由代码，这样岂不是又会代码冗余了吗。

解决方案很简单，用路由分组。将功能类似，要用到同一个中间件的路由，定义在路由分组里，然后给这个分组，统一指定分配一个中间件就好了嘛。参考官方文档，像这样定义路由分组，并指定中间件：

```php
Route::middleware(['custom.check_login'])->group(function () {
    Route::get('about', 'Front\UserController@about');
    Route::get('contact', 'Front\UserController@contact');
});
```

你可以看到，`group()` 函数，是用来分组的，将 about 和 contact 两个路由分在了一个组里，而 `middleware()` 函数，则是指定用哪些中间件来进行数据过滤，你是可以在 `[]` 参数数组中，指定多个中间件的。

## 结束
这期教程就是讲中间件，使用的方式很简单，关键是要理解它的作用和使用场景，这在日后的实际开发中，会经常的用到。