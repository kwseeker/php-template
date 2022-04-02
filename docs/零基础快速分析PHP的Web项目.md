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

官网说请求入口是`index.php`，感觉这么说不太恰当，index.php是启动 http服务器的入口；

请求的入口应该还是Router。

基于Laravel开发主要就是针对ServiceProvider这部分。

> TODO

#### 基本组件

##### 调试配置&日志

为了更好的理解代码执行过程，需要调试或输出日志。

**调试配置**

PhpStorm并没有内置调试组件，需要安装 `Settings->PHP->Debug`; 这里选择 [XDebug](https://www.jetbrains.com/help/phpstorm/2021.3/configuring-xdebug.html)，按官方文档的配置即可。

注意需要选择和PHP适配的XDebug版本

将 `php -i`的结果粘贴到https://xdebug.org/wizard.php，获取安装建议，不建议通过apt命令行安装，经试验会下载旧版本，且会安装旧版本的php。

如果使用旧版本就满足需要的话，可以执行下面的命令快速安装XDebug

```shell
# 这个命令比较简单但是装的不是最新版本这里默认装的2.6.0，而且可能会自动下载php,可能把之前的安装给覆盖了
sudo apt-get install php-xdebug
# 获取拓展包路径，官网没说怎么找拓展包路径 zend_extension, 可以用下面命令定位
sudo updatedb
locate xdebug.so
# 修改php.ini, 在安装路径里如这里自动安装的php路径：/etc/php/7.2/cli/php.ini
[xdebug]
zend_extension="/usr/lib/php/20170718/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_host=127.0.0.1
xdebug.remote_port=9000
```

按照官方安装建议安装（比如针对我本地php(7.4.27)环境给出的建议）：

1. Download [xdebug-3.1.3.tgz](https://xdebug.org/files/xdebug-3.1.3.tgz)

2. Install the pre-requisites for compiling PHP extensions.
   On your Ubuntu system, install them with: `apt-get install php-dev autoconf automake`

3. Unpack the downloaded file with `tar -xvzf xdebug-3.1.3.tgz`

4. Run: `cd xdebug-3.1.3`

5. Run: `phpize` (See the [FAQ](https://xdebug.org/docs/faq#phpize) if you don't have `phpize`).

   As part of its output it should show:

   ```
   Configuring for:
   ...
   Zend Module Api No:      20190902
   Zend Extension Api No:   320190902
   ```

   If it does not, you are using the wrong `phpize`. Please follow [this FAQ entry](https://xdebug.org/docs/faq#custom-phpize) and skip the next step.

6. Run: `./configure`

7. Run: `make`

8. Run: `cp modules/xdebug.so /opt/lampp/lib/php/extensions/no-debug-non-zts-20190902`

9. Update `/opt/lampp/etc/php.ini` and add the line:
   `zend_extension = xdebug`

**日志输出**



#### 其他组件

### Laravel/tinker

### fideloper/proxy



## PHP Web项目架构

即PHP Web项目组件怎么集成进去的？



## 从某条业务线开始复刻

比如登录，入口URI: login/verify， 由前面分析[路由](https://laravelacademy.org/post/21970)一般都放在项目根目录的routes目录下，

```php
Route::post('/login/verify', 'LoginController@verify');
```

然后看下代码语法的含义，看上去和C++调用静态成员方法一样，去官网[类与对象](https://www.php.net/manual/zh/language.oop5.php)查了下含义一样的。

Route 是Laravel framework中定义的一个类，post() 方法用于响应Post请求。

```php
 @method static \Illuminate\Routing\Route post(string $uri, array|string|callable|null $action = null)
```

'LoginController@verify' 又是什么意思？

Laravel 官方文档貌似没有讲这种使用方式，在一个非官方的教程[Laravel 路由入门](https://laravelacademy.org/post/9611)中找到了答案：

将请求传递给了LoginCotroller的verify方法处理（这里@就只是传递了个字符串，并不是php的什么语法，具体对@怎么解析处理的还是得看Laravel源码），一方面可以简化路由文件，另一方面可以使用路由缓存，而传递闭包的方式是无法使用路由缓存的。





> 涉及的相关语法：
>
> php源码文件格式；
>
> 字符串定义，单引号不会对斜线转义，双引号会对斜线转义；字符串拼接是通过'.'; 
>
> 类定义(包括成员变量、成员方法)，对象实例化，成员方法调用；
>
> php是脚本语言，没有main() 方法入口，代码逻辑从上到下执行；
>
> php是弱类型语言，方法不需要指定返回值类型，方法传参参数可能传多种类型数据；

