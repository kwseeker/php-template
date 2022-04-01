# 零基础快速分析 PHP Web项目方法的思考

个人认为比较快速的流程：

1. 了解PHP项目依赖管理
2. 分析PHP项目依赖配置文件，获取核心依赖模块(比如Web框架)
3. 了解核心依赖模块或框架的基本功能和使用方法，以及架构
4. 梳理PHP Web项目的架构，从Web路由开始
5. 以某个业务线开始，分析请求的业务流程，涉及哪些依赖模块，可以在一个新的项目中复刻此业务线进行修改测试
6. 后面就是业务细节分析



下面以公司一个老PHP项目为例，分析。



## 了解PHP项目依赖管理

首先在[awesome-php](https://github.com/ziadoz/awesome-php)（中文仓库[awesome-php-cn](https://github.com/jobbole/awesome-php-cn)同步相对滞后）查PHP项目的依赖管理方法。

显示包和依赖管理器是[Composer](https://getcomposer.org/)，依赖仓库[Packagist](http://packagist.org/)，其他都是一些拓展工具 [Composer Installers](https://github.com/composer/installers)、[Phive](https://phar.io/)、[Pickle](https://github.com/FriendsOfPHP/pickle)。

了解Composer的基本使用方法，创建一个基于Composer的项目工程。

1. 安装Composer

   ```shell
   # 按官网文档下载 installer (php文件)，执行下载 composer.phar 
   php installer
   # 然后选择全局安装，这里放到 /opt/composer/bin/
   mv composer.phar /opt/composer/bin/composer
   # 加入到PATH
   # export PATH=/opt/lampp/bin:$PATH  # 这个是PHP安装目录
   export PATH=/opt/composer/bin:$PATH
   # 或者放到 ~/.local/bin/ 也可以
   # 查看composer.phar版本，可以直接命令行执行
   composer --version
   # Composer 2.3.2 2022-03-30 20:45:25
   ```

   有人推荐用国内镜像仓库，但是测试发现新版本的包找不到，估计同步频率太低或者停止同步了。还是翻墙靠谱。

   ```json
   "repositories": {
       "packagist": {
           "type": "composer",
           "url": "https://packagist.phpcomposer.com"
       }
   }
   ```

2. 使用Composer命令创建项目

   本来想用PhpStorm创建，但是必须指定的Package，这时还没有确定具体要添加什么依赖，作罢。

   ```shell
   composer init
   ```

   生成的文件结构，关于Web项目目录结构可以参考官网测试Demo: example-app。

   ```text
   .
   ├── composer.json
   ├── composer.lock	在安装依赖后，Composer 将把安装时确切的版本号列表写入 composer.lock 文件。这将锁定该项目的特定版本。
   ├── src
   └── vendor
       ├── autoload.php
       └── composer
           ├── autoload_classmap.php
           ├── autoload_namespaces.php
           ├── autoload_psr4.php
           ├── autoload_real.php
           ├── autoload_static.php
           ├── ClassLoader.php
           ├── installed.json
           ├── installed.php
           ├── InstalledVersions.php
           └── LICENSE
   ```

   > TODO List:
   >
   > 项目组织结构和命名规范。
   >
   > PHP执行原理。

   

## 获取主要项目依赖

项目依赖在 require 中指定, 查看项目原始提交。

查看项目原始提交, 有如下依赖

```json
{
  "require": {
    "php": "^7.1.3",
    "fideloper/proxy": "^4.0",
    "laravel/framework": "5.6.*",
    "laravel/tinker": "^1.0"
  }
}
```

+ fideloper/proxy

+ [laravel/framework](https://github.com/laravel/framework)

  Laravel是一个具有表达性和优雅语法的web应用程序框架。当前是最流行的PHP Web框架。

+ [laravel/tinker](https://github.com/laravel/tinker)

  是一个 [REPL (read-eval-print-loop)](https://en.wikipedia.org/wiki/Read–eval–print_loop)，REPL 是指交互式命令行界面，它可以让你输入一段代码去执行，并把执行结果直接打印到命令行界面里；用于代码调试。

  

## 核心依赖功能和基本使用

### laravel/framework

[Laravel 8 中文文档](https://laravelacademy.org/books/laravel-docs-8)

#### 在项目中引入依赖

可以在创建项目的时候指定(可以直接引入laravel/laravel ，就类似spring-initializer一样，会自动创建规范的目录结构，以及基本的代码文件)。

```shell
composer create-project --prefer-dist laravel/laravel webdemo
```

也可以在composer.json手动添加，关于通过 Composer 引入依赖参考：[Composer Basic usage](https://getcomposer.org/doc/01-basic-usage.md)。

`composer create-project`命令初始化的项目[目录结构](https://laravelacademy.org/post/21960)

```text
.
├── app								  //应用代码
│   ├── Console
│   ├── Exceptions
│   ├── Http
│   ├── Models
│   └── Providers
├── artisan							//控制台脚本
├── bootstrap					//启动目录
│   ├── app.php
│   └── cache
├── composer.json
├── composer.lock
├── config							//配置文件
│   ├── app.php
│   ├── auth.php
│   ├── broadcasting.php
│   ├── cache.php
│   ├── cors.php
│   ├── database.php
│   ├── filesystems.php
│   ├── hashing.php
│   ├── logging.php
│   ├── mail.php
│   ├── queue.php
│   ├── sanctum.php
│   ├── services.php
│   ├── session.php
│   └── view.php
├── database					//数据库目录
│   ├── factories
│   ├── .gitignore
│   ├── migrations
│   └── seeders
├── .editorconfig
├── .env							//环境配置文件
├── .env.example
├── .gitattributes
├── .gitignore
├── .idea
│   ├── commandlinetools
│   ├── .gitignore
│   ├── misc.xml
│   └── workspace.xml
├── package.json
├── phpunit.xml
├── public							//对外公开目录
│   ├── favicon.ico
│   ├── .htaccess
│   ├── index.php
│   └── robots.txt
├── README.md
├── resources				  //资源目录
│   ├── css
│   ├── js
│   ├── lang
│   └── views
├── routes						//路由目录
│   ├── api.php
│   ├── channels.php
│   ├── console.php
│   └── web.php
├── server.php
├── storage						//文件存储目录
│   ├── app
│   ├── framework
│   └── logs
├── .styleci.yml
├── tests							//tests
│   ├── CreatesApplication.php
│   ├── Feature
│   ├── TestCase.php
│   └── Unit
├── vendor
└── webpack.mix.js
```

如果不知道应该使用哪个版本，可以去 http://packagist.org/ 查询。

> TODO List
>
> Composer版本约束为何要使用不确切的版本号？比如范围、通配符、赋值运算符。

#### Laravel 架构与原理

##### 服务器启动流程

启动脚本`php artisan serve`；

> TODO

##### 请求的生命周期

![](https://laravel.gstatics.cn/storage/uploads/images/gallery/2020-09/scaled-1680-/image-6-1024x579.png)

请求入口`index.php`；

基于Laravel开发主要就是针对ServiceProvider这部分。

> TODO

#### 基本组件

#### 其他组件

### Laravel/tinker

### fideloper/proxy

