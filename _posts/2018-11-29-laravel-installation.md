---
title: Laravel 基础教程【1】 入门
tags: Laravel
---

<div>{%- include extensions/bilibili.html id='37036878' -%}</div>

Laravel 基础教程，希望用浅显易懂的方式，让我带你一起飞。

<!--more-->

## 修订

- 2018-11-30
	视频中，遗漏了很重要的一个步骤，那就是在通过 `composer global require "laravel/installer"`，安装好 Laravel 安装器 之后，你用 `laravel new blog` 会告诉你 `laravel` 命令不存在，这是因为，没有把 Composer 下的 `vendor/bin` 加入环境变量导致的。所以，在使用 `laravel` 命令 `new` 项目之前，一定要将下面的路径，加入环境变量：
	
	- MacOS：`$HOME/.composer/vendor/bin`
	- GNU / Linux 发行版：`$HOME/.config/composer/vendor/bin`
	- Windows 下的 Composer 安装好后，会自动配置好的

## Laravel 是什么
Laravel 是一个优雅的为 Web 应用开发者而生的 PHP 框架。

## Laravel 安装
要安装 Laravel，必须先安装 Composer。

#### Composer 又是什么
Composer 是 PHP 下的一个依赖管理器，可以很方便的安装、使用第三方开发的PHP程序，集成到自己的项目中，而如果你所依赖的软件包有升级，而此软件包又依赖的其他包也有升级，你不需要一一的升级它们，而只需 `composer update` 一行命令搞定。所以，现在很多 PHP 的框架，都用 Composer 来管理框架内众多的扩展软件包。

#### Composer 安装
官网 - [https://getcomposer.org](https://getcomposer.org)

**Linux、MacOS 下安装**

执行一下命令，即可安装完毕

```php
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '93b54496392c062774670ac18b134c3b3a95e5a5e5c8f1a9f115f203b75bf9a129d5daa8ba6a13e2cc8a1da0806388a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

使 composer 在全局环境中都可用

```php
mv composer.phar /usr/local/bin/composer
```

**Windows 下安装**

下载安装程序 [https://getcomposer.org/Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe)

**检验安装是否成功**

不管你用 Linux、Mac 还是 Windows，在 Composer 安装好后，在命令行下输入 `composer --version` 回车，如果显示出 Composer 的版本号信息，则说明安装成功。

#### 安装 Laravel Installer 和 创建 Laravel 项目
Composer 安装好后，即可通过 `composer global require laravel/installer` 来安装 Laravel Installer，你可以把这一步安装的 Laravel Installer 当成一个快捷命令，后面用它来创建 Laravel 项目。

最后，通过 `laravel new blog`，来创建一个名为 blog 的 Laravel 项目。

## Laravel 服务环境配置、运行
项目创建好了，如何运行，立刻看到网站呢？有一种可以立刻看到成果的方法，用 `php artisan serve` 开启一个临时的服务环境，然后在浏览器地址栏内访问 `http://localhost:8000`，就可以立刻看到刚才用 Laravel 框架创建出来的网站效果了。

但是，正常情况下，我们本地环境上，都是用 apache 或是 nginx 来跑自己的测试项目，所以，下面介绍下如何在这2种服务环境中，配置 Laravel 项目。

#### Apache Laravel 项目配置
1. 首先，找到你的 apache 配置文件 httpd.conf，确保你的 apache 开启了 mod_rewrite，并且启用了虚拟站点配置文件，类似 `Include /usr/local/etc/httpd/extra/httpd-vhosts.conf`，这行开头的 `#` 注释删掉。

2. 在本地的 hosts 文件中，配置虚拟域名，windows 下的 hosts 在 C:\Windows\System32\drivers\etc\hosts，linux 的在 /etc/hosts，mac 的在 /private/etc/hosts，在里面添加一行 `127.0.0.1    laravel.me`，域名可自定义，后面就用这个域名来访问本地刚刚创建的 Laravel 项目。

3. 在 httpd-vhosts.conf 中，增加一下配置，来指明 Laravel 项目的路径、绑定的域名 和 访问权限等

```php
<VirtualHost *:80>
    DocumentRoot "/Users/bananaplan/www/blog/public"
    ServerName laravel.me
</VirtualHost>
```

4. 最后，重启 apache，在浏览器中，通过 laravel.me 访问网站。

#### Nginx Laravel 项目配置
而 nginx 的配置主要牵扯2个配置文件：nginx.conf 和 xxx.xx.conf，前者是 nginx 服务器的基础配置，后者是某一个站点的独立配置。

1. 在 nginx.conf 中，基本都会开启 `include servers/*;` 这一行，也是类似 apache 中的开启虚拟站点的独立配置。

2. 在 nginx 的配置目录下，你应该能找到 servers 这个文件夹，windows 系统的话，`include servers/*;` 类似的路径，应该是全路径，所以你会很容易的找到；而 linux 或 mac 系统，如果默认配置中不是全路径，不出意外的话，应该会在 `/usr/local/etc/nginx/servers` 下，如果你实在找不到在哪里，没关系，可以自己修改 include 的路径为全路径，如：`include /usr/local/etc/nginx/servers/*;`。

3. 在 servers 文件夹下，新建 laravel.me.conf 文件，独立配置你的网站：

```php
server {
    listen 80;
    server_name laravel.me;
    root /Users/bananaplan/www/blog/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/usr/local/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

4. 重启 nginx，通过 laravel.me 访问网站。

## 结语
- 本节，我们讲了 Laravel 的安装，在此之前，会牵扯到 Composer 的安装和配置，最后通过命令 `laravel new 项目名称`，来成功创建出一个基于 Laravel 框架的项目。

- 最后，我们分别讲了在 apache 和 nginx 下的服务器环境配置，以使我们可以在本地，通过域名的形式，访问我们的网站。

- 任何事情，看起来容易，当亲自动手的时候，总是会发现，不是那么简单，总是会碰到各种问题。如果你在亲自操刀的时候，遇到了任何问题，欢迎在下面留言。

