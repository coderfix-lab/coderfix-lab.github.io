---
layout: post
title:  "【Yii2】yii2-editable在GridView上修改数值并刷新页面"
date:   2020-05-08 12:55:06 +0800
categories: Yii2
---

本文为本人原创，首发地址 https://coderfix.blog.csdn.net/article/details/105954680 

# 需求
因为业务需求需要修改数据的排序值，但是为了单独修改排序值打开页面提交数据又对用户操作不好，所以我决定采用直接在列上对数据进行修改。

# 版本说明
## PHP 
> root@1bd5d900decc:/var/www/html# php -v                                                                
PHP 7.0.25 (cli) (built: Nov  4 2017 10:58:36) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.0.25, Copyright (c) 1999-2017, by Zend Technologies
    with blackfire v1.18.0~linux-x64-non_zts70, https://blackfire.io, by SensioLabs

## Yii2
> root@1bd5d900decc:/var/www/html# ./yii
This is Yii version 2.0.32.

# 安装
## 更换composer的阿里源
首先是更换源，原因是composer的国外源的速度非常慢，并且中国官方提供的源的速度也不快，所以我们采用阿里的源。

修改composer.json文件，在最后增加如下内容

```
    "repositories": {
        "packagist": {
            "type": "composer",
            "url": "https://mirrors.aliyun.com/composer"
        }
    }
```

## 安装kartik-v/yii2-grid
```
composer require kartik-v/yii2-grid "@dev"
```
如果提示因为你因为其他的第三方因为php版本太低报错，就忽略掉版本

```
composer require --ignore-platform-reqs kartik-v/yii2-grid "@dev"
```

## 安装kartik-v/yii2-editable
```
composer require kartik-v/yii2-editable "@dev"
```

# 修改config
对配置文件`config/main.php`进行修改
```php
    'modules' => [
        'gridview' =>  [
            'class' => '\kartik\grid\Module'
        ]
```

# 在视图文件引入
在我们原来的视图文件中引入`kartik\grid\GridView`和`kartik\editable\Editable`,并将原来的注释掉

```php
//use yii\grid\GridView;
use kartik\grid\GridView;
use kartik\editable\Editable;
```
# 在GridView中使用
对展示的列使用引入的可编辑列
```
[
                                    'attribute' => 'sort',
                                    'class'=>'kartik\grid\EditableColumn',
                                ],
```

然后舒心页面，看到的是这样的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506172054777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
这个时候我们修改提交，数值是不会改变，我们需要去编辑对应的控制器，求接收提交到到本页的ajax。

# 修改Controller
打开浏览器的F12，我们能看到这次提交的内容。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506172642356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
其中

hasEditable 表示有修改提交
editableIndex 表示对应GridView的第几行
editableKey 表示修改内容对应的id
editableAttribute 表示对应修改内容model的字段
PageCategoryIdles[3][sort] 表示对应model GridView的index 对应字段 修改的值

经过分析之后，我们在原来的控制器中这样写，亲测可用

```php
        if (isset($_POST['hasEditable'])) {
            \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
            $model = PageCategoryIdles::findOne($_POST['editableKey']);;
            $value = $_POST['PageCategoryIdles'][$_POST['editableIndex']][$_POST['editableAttribute']];
            $model->setAttribute($_POST['editableAttribute'], $value);
            if ($model->save()) {
                return ['output' => $value, 'message' => ''];
            } else {
                return ['output' => '', 'message' => ''];
            }
        }
```

PageCategoryIdles是我的model的名字，复制的时候需要替换成你使用的model的名字

然后再次提交，经过ajax提交，数据就会成功修改并直接返回到页面上。

# 成功后刷新页面
这样提交页面之后，页面的数据排序数值完成了，修改，但是数据没有根据排序变成排序之后的字段。

我找到官方文档，官方有个`pluginEvents`对事件进行回调，文档上是这么写的

```
pluginEvents = [
    "editableChange"=>"function(event, val) { log('Changed Value ' + val); }",
    "editableSubmit"=>"function(event, val, form) { log('Submitted Value ' + val); }",
    "editableBeforeSubmit"=>"function(event, jqXHR) { log('Before submit triggered'); }",
    "editableSubmit"=>"function(event, val, form, jqXHR) { log('Submitted Value ' + val); }",
    "editableReset"=>"function(event) { log('Reset editable form'); }",
    "editableSuccess"=>"function(event, val, form, data) { log('Successful submission of value ' + val); }",
    "editableError"=>"function(event, val, form, data) { log('Error while submission of value ' + val); }",
    "editableAjaxError"=>"function(event, jqXHR, status, message) { log(message); }",
];
```
但是我找不到结合GridView使用的例子。

然后我找到源码``

```php
class EditableColumn extends DataColumn
{
    /**
     * @var array|Closure the configuration options for the [[\kartik\editable\Editable]] widget. If not set as an
     * array, this can be passed as a callback function of the signature: `function ($model, $key, $index)`, where:
     * - `$model`: _\yii\base\Model_, is the data model.
     * - `$key`: _string|object_, is the primary key value associated with the data model.
     * - `$index`: _integer_, is the zero-based index of the data model among the model array returned by [[dataProvider]].
     * - `$column`: _EditableColumn_, is the column object instance.
     *
     * This property allows to configure these additional settings for configuring the widget options:
     * - `class`: _string_, the Editable widget class name. If not set this defaults to `kartik\editable\Editable`.
     */
    public $editableOptions = [];
```

结合上面的，最终在视图上的写法如下

```php
                                [
                                    'attribute' => 'sort',
                                    'class'=>'kartik\grid\EditableColumn',
                                    'editableOptions'=>[
                                            'pluginEvents'=>[
                                                "editableSuccess"=>"function(event, val, form, data) { console.log('Successful submission of value ' + val); history.go(0); }",
                                            ]
                                    ]
                                ],
```

到此，完成了页面修改内容并刷新排序的全部功能。

# 细节总结
- composer换源并忽略版本
- 引入第三方
- 通过回调刷新页面

**官方文档不一定都有，解决问题还需要查源码和动脑筋。**
# 参考资料
- https://github.com/kartik-v/yii2-editable
- https://demos.krajee.com/editable