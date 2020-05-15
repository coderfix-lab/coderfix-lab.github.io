---
layout: post
title:  "【Yii2】Yii2使用Mongodb配置"
date:   2020-05-15 14:47:00 +0800
categories: Yii2  Mongodb
---
本文为本人原创，首发地址 https://coderfix.blog.csdn.net/article/details/106141326

# 环境

```sh
lixiaoyu@localhost basic % php -v
PHP 7.1.32 (cli) (built: Feb 17 2020 12:26:26) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2018 Zend Technologies
```

```sh
lixiaoyu@localhost basic % ./yii 

This is Yii version 2.0.35.

```

# 安装

## mongodb扩展

```sh
 brew install autoconf
sudo pecl install mongodb
```

```sh
lixiaoyu@localhost ~ % php -m                         
[PHP Modules]
bcmath
bz2
calendar
Core
ctype
curl
date
dom
exif
fileinfo
filter
ftp
gd
gettext
hash
iconv
imap
intl
json
ldap
libxml
mbstring
mcrypt
mongodb
mysqli
mysqlnd
openssl
pcre
PDO
pdo_mysql
pdo_pgsql
pdo_sqlite
pgsql
Phar
posix
readline
Reflection
session
SimpleXML
soap
sockets
SPL
sqlite3
standard
tokenizer
wddx
xml
xmlreader
xmlrpc
xmlwriter
xsl
zip
zlib

[Zend Modules]
```

## 安装"yiisoft/yii2-mongodb": "~2.1.0"
添加`"yiisoft/yii2-mongodb": "~2.1.0"`到`composer.json`.

执行`composer update`

经过安装之后，我得到的版本是`2.1.9`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515143245331.png)
# 配置

## 数据库配置
对于新增的数据库，需要增加对应的权限，比如我新增demo数据库，那么不管是admin用户还是其他的新增用户，都需要在admin和demo数据库中新增对应的用户，让他拥有对应的读写权限。

如果不做这一步配置，就会导致报错`Authentication failed`.

```sql
use admin
db.createUser({
    user:'xiaoyu',
    pwd:'123456',
    roles:[{role:'readWrite',db:'admin'}]
})
```
```sql
use demo
db.createUser({
    user:'xiaoyu',
    pwd:'123456',
    roles:[{role:'readWrite',db:'demo'}]
})
```

这里注意，网上一些资料在roles里面的array里没有加{},这样会导致报错`SyntaxError: missing ] after element list`,说明看资料要看最新的，旧版本可能会出问题。

## 配置文件配置
修改`config/web.conf`,在`components`中新增

```php
        'mongodb' => [
            'class' => '\yii\mongodb\Connection',
            'dsn' => 'mongodb://localhost:27017/demo',
            'options' => [
                "username" => "xiaoyu",
                "password" => "123456"
            ]
        ],
```

# 执行操作
```php
                //写入数据库
                $collection = Yii::$app->mongodb->getCollection ( 'dom' );
                $res = $collection->insert ($data);
                var_dump($res);
```

返回结果如下

```
object(MongoDB\BSON\ObjectId)#168 (1) { ["oid"]=> string(24) "5ebe35608b78250af03a9dc6" }
```
到此，引入完成。

# 参考资料
- https://www.yiiframework.com/extension/yiisoft/yii2-mongodb
- https://www.php.net/manual/en/set.mongodb.php
- https://stackoverflow.com/questions/36515921/mongodb-syntaxerror-missing-after-element-list
- https://segmentfault.com/q/1010000007993711





