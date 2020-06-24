---
layout: post
title:  "Laravel-admin后台框架-1安装"
date:   2020-06-24 12:00:00 +0800
categories: laravel 
---

本文原链接 https://coderfix.blog.csdn.net/article/details/106860589 

# 前言
Laravel是众所周知的优雅的PHP框架。

Laravel-admin可以快速实现后台的搭建，并且可以帮助不熟悉laravel的人快速熟悉。

但是再好用的工具也有学习的过程，下面开始我们由浅入深的学习吧~

# 安装
## 环境
### macOS
10.15.5

### PHP
7.4.2
```sh
coderfix.blog.csdn.net@localhost blog % php -v 
PHP 7.4.2 (cli) (built: Feb 17 2020 12:56:02) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
```
### laravel安装的版本
7.12.0
## 安装laravel

```sh
coderfix.blog.csdn.net@localhost ~ % composer global require laravel/installer
Changed current directory to /Users/lixiaoyu/.composer
    1/6:	https://mirrors.aliyun.com/composer/p/provider-2019-07$d52ebeb698ba9dbc61696d8b26e680771ccf276269f9e5ac8b617868b390a03e.json
    2/6:	https://mirrors.aliyun.com/composer/p/provider-latest$e85d549a2a54bf65464a47e8957ed64c8b38174ee59c934eec2984a014387c6a.json
    3/6:	https://mirrors.aliyun.com/composer/p/provider-2020-01$91379672ad2f40bfd93762449f18b3b810bde04edf8fc5ddd08a24d6f1dad71c.json
    4/6:	https://mirrors.aliyun.com/composer/p/provider-2019$c41b5c15a3c6c15ca03a55489f53a78c5213dbb2a2c3e1d5400ee15a0bf67371.json
    5/6:	https://mirrors.aliyun.com/composer/p/provider-2019-10$c51da90d7b29c4e3cea025bea11a82fae6f42ef39486f33f51d097eb34cd4928.json
    6/6:	https://mirrors.aliyun.com/composer/p/provider-2020-04$d8487a83ccc5a1fe37fe635e4df4730643d84bc90a0a5b9ac3f43f970d81e2c2.json
    Finished: success: 6, skipped: 0, failure: 0, total: 6
Using version ^3.1 for laravel/installer
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Nothing to install or update
Writing lock file
Generating autoload files
9 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```
## 修改环境变量
由于laravel的命令是通过composer安装的，所以我们需要修改环境变量让composer安装的命令全局可用。

```sh
coderfix.blog.csdn.net@localhost ~ % vim .zshrc
```
添加下面内容到文件尾部
```
export PATH="$HOME/.composer/vendor/bin:$PATH"
```
让文件生效
```sh
coderfix.blog.csdn.net@localhost ~ % source .zshrc
```

## 创建laravel应用
Laravel-admin是通过扩展的形式进行安装的，所以需要我们自己创建一个应用。

```sh
lixiaoyu@localhost php % composer create-project --prefer-dist laravel/laravel fit
```

## 配置nginx和数据库连接

ngixn配置如下，注意根目录指向public。

```
server {
    listen 80;
    server_name fit.local;
    index index.php
    error_log /home/xiaoyu/code/logs/example.error.log;
    access_log /home/xiaoyu/code/logs/example.access.log;
    root /home/xiaoyu/code/fit/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
}
```





打开浏览器，安装laravel成功。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619180700194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)



## 修改数据库连接
这一步是给安装laravel-admin做准备。

修改config/database.php,修改用户名密码，如果你用的不是本地数据库，记得修改host和port.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619180921804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
## 创建数据库
新建一个数据库，名称是 laravel 编码是utf8mb4.
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061918142478.png)
如果你想用别的数据库名称，需要修改根目录下的`.env`文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619181542114.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
## 安装laravel-admin
在项目根目录执行

```sh
coderfix.blog.csdn.net@localhost fit % composer require encore/laravel-admin
    1/2:	https://mirrors.aliyun.com/composer/p/provider-latest$ce8f7dd69721c361392a2c5e43283a288d5da30a7b9dd22c1b5332d3aebb3e3d.json
    2/2:	https://mirrors.aliyun.com/composer/p/provider-2020-04$2b2e8b3f1aad56cd7cc11620c8a16d8ae0b7ff4cc36e432e0de701874a0fb212.json
    Finished: success: 2, skipped: 0, failure: 0, total: 2
Using version ^1.8 for encore/laravel-admin
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 5 installs, 0 updates, 0 removals
  - Installing doctrine/event-manager (1.1.0): Loading from cache
  - Installing doctrine/cache (1.10.1): Loading from cache
  - Installing doctrine/dbal (2.10.2): Loading from cache
  - Installing symfony/dom-crawler (v5.1.2): Loading from cache
  - Installing encore/laravel-admin (v1.8.1): Loading from cache
doctrine/cache suggests installing alcaeus/mongo-php-adapter (Required to use legacy MongoDB driver)
encore/laravel-admin suggests installing intervention/image (Required to handling and manipulation upload images (~2.3).)
encore/laravel-admin suggests installing spatie/eloquent-sortable (Required to built orderable gird.)
Writing lock file
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: encore/laravel-admin
Discovered Package: facade/ignition
Discovered Package: fideloper/proxy
Discovered Package: fruitcake/laravel-cors
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
46 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
lixiaoyu@localhost fit % php artisan vendor:publish --provider="Encore\Admin\AdminServiceProvider"
Copied Directory [/vendor/encore/laravel-admin/config] To [/config]
Copied Directory [/vendor/encore/laravel-admin/resources/lang] To [/resources/lang]
Copied Directory [/vendor/encore/laravel-admin/database/migrations] To [/database/migrations]
Copied Directory [/vendor/encore/laravel-admin/resources/assets] To [/public/vendor/laravel-admin]
Publishing complete
coderfix.blog.csdn.net@localhost fit % php artisan admin:install
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.07 seconds)
Migrating: 2016_01_04_173148_create_admin_tables
Migrated:  2016_01_04_173148_create_admin_tables (0.81 seconds)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.04 seconds)
Database seeding completed successfully.
Admin directory was created: /app/Admin
HomeController file was created: /app/Admin/Controllers/HomeController.php
AuthController file was created: /app/Admin/Controllers/AuthController.php
ExampleController file was created: /app/Admin/Controllers/ExampleController.php
Bootstrap file was created: /app/Admin/bootstrap.php
Routes file was created: /app/Admin/routes.php
```

打开浏览器，安装成功！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619181919487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

输入默认的用户名密码 admin admin，进入主界面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619182021366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
# 总结
到此完成了完整的安装流程，其中的坑还是有的。

- 安装laravel的时候不能使用laravel new fit 会报错
- 一定要先修改数据库连接
# 参考资料

- https://learnku.com/docs/laravel/7.x
- https://laravel-admin.org/docs/zh












