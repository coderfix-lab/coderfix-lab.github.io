---
layout: post
title:  "【Yii2】模型Model中使用rules规则定义场景setScenario限制规则"
date:   2020-05-13 18:30:00 +0800
categories: yii2 
---

本文为本人原创，首发地址 https://coderfix.blog.csdn.net/article/details/106103771

# 业务需求
在我们的日常需求中，会有这种

数据表中存在图片字段，为必填，需要在操作过程中做到

- 创建数据时，图片字段必须
- 修改数据时，图片字段如果不传，就不修改

# 解决方案

## 业务数据操作
如果要保持原来的图片数据，那只要修改的时候带着原来的参数即可

1. 加载表单时带原来的数据
2. 图片数据放在隐藏的文本域中
3. 提交表单时如果图片上传的字段没有值，就将原来的图片数据从隐藏文本域中拿出来

这样的操作当然能满足需求，但是太复杂了

## 定义场景setScenario
在Yii的数据操作中，一般的逻辑如下

1. 判断是否有提交数据`Yii::$app->request->post()`
2. 表单数据加载到模型model中 `$model->load(Yii::$app->request->post())`
3. 保存model的数据 `$model->save()`

重点时第三步，在`save`的操作中，默认执行验证操作，判断提交的数据是否如何model中的规则

```php
    /**
     * Saves the current record.
     *
     * This method will call [[insert()]] when [[isNewRecord]] is `true`, or [[update()]]
     * when [[isNewRecord]] is `false`.
     *
     * For example, to save a customer record:
     *
     * ```php
     * $customer = new Customer; // or $customer = Customer::findOne($id);
     * $customer->name = $name;
     * $customer->email = $email;
     * $customer->save();
     * ```
     *
     * @param bool $runValidation whether to perform validation (calling [[validate()]])
     * before saving the record. Defaults to `true`. If the validation fails, the record
     * will not be saved to the database and this method will return `false`.
     * @param array $attributeNames list of attribute names that need to be saved. Defaults to null,
     * meaning all attributes that are loaded from DB will be saved.
     * @return bool whether the saving succeeded (i.e. no validation errors occurred).
     */
    public function save($runValidation = true, $attributeNames = null)
    {
        if ($this->getIsNewRecord()) {
            return $this->insert($runValidation, $attributeNames);
        }

        return $this->update($runValidation, $attributeNames) !== false;
    }
```

`$runValidation = true` 会验证规则,如下

```
    public function rules()
    {
        return [
            [['name','sort'], 'required'],
            [['classify', 'is_recommend', 'sort', 'status', 'create_at', 'update_at'], 'integer'],
            [['name'], 'string', 'max' => 64],
            [['images'], 'string', 'max' => 255],
        ];
    }
```

这里清楚的标记了哪些字段是有必须的，哪些字段的长度最大是多少

# 设定场景setScenario
我们先看源码`model`中的`rules`的部分

```php
    /**
     * Returns the validation rules for attributes.
     *
     * Validation rules are used by [[validate()]] to check if attribute values are valid.
     * Child classes may override this method to declare different validation rules.
     *
     * Each rule is an array with the following structure:
     *
     * ```php
     * [
     *     ['attribute1', 'attribute2'],
     *     'validator type',
     *     'on' => ['scenario1', 'scenario2'],
     *     //...other parameters...
     * ]
     * ```
     *
     * where
     *
     *  - attribute list: required, specifies the attributes array to be validated, for single attribute you can pass a string;
     *  - validator type: required, specifies the validator to be used. It can be a built-in validator name,
     *    a method name of the model class, an anonymous function, or a validator class name.
     *  - on: optional, specifies the [[scenario|scenarios]] array in which the validation
     *    rule can be applied. If this option is not set, the rule will apply to all scenarios.
     *  - additional name-value pairs can be specified to initialize the corresponding validator properties.
     *    Please refer to individual validator class API for possible properties.
     *
     * A validator can be either an object of a class extending [[Validator]], or a model class method
     * (called *inline validator*) that has the following signature:
     *
     * ```php
     * // $params refers to validation parameters given in the rule
     * function validatorName($attribute, $params)
     * ```
     *
     * In the above `$attribute` refers to the attribute currently being validated while `$params` contains an array of
     * validator configuration options such as `max` in case of `string` validator. The value of the attribute currently being validated
     * can be accessed as `$this->$attribute`. Note the `$` before `attribute`; this is taking the value of the variable
     * `$attribute` and using it as the name of the property to access.
     *
     * Yii also provides a set of [[Validator::builtInValidators|built-in validators]].
     * Each one has an alias name which can be used when specifying a validation rule.
     *
     * Below are some examples:
     *
     * ```php
     * [
     *     // built-in "required" validator
     *     [['username', 'password'], 'required'],
     *     // built-in "string" validator customized with "min" and "max" properties
     *     ['username', 'string', 'min' => 3, 'max' => 12],
     *     // built-in "compare" validator that is used in "register" scenario only
     *     ['password', 'compare', 'compareAttribute' => 'password2', 'on' => 'register'],
     *     // an inline validator defined via the "authenticate()" method in the model class
     *     ['password', 'authenticate', 'on' => 'login'],
     *     // a validator of class "DateRangeValidator"
     *     ['dateRange', 'DateRangeValidator'],
     * ];
     * ```
     *
     * Note, in order to inherit rules defined in the parent class, a child class needs to
     * merge the parent rules with child rules using functions such as `array_merge()`.
     *
     * @return array validation rules
     * @see scenarios()
     */
    public function rules()
    {
        return [];
    }
```

其中 关于场景的部分,通过on定义场景

```php
'on' => ['scenario1', 'scenario2'],
```
可以应用规则。 如果未设置此选项，则该规则将适用于所有场景。
```php
rule can be applied. If this option is not set, the rule will apply to all scenarios.
```

当规则对应的场景时，规则会被采用

```php
  ['password', 'compare', 'compareAttribute' => 'password2', 'on' => 'register'],
   ['password', 'authenticate', 'on' => 'login'],
```

那么，最后我们得到关于图片的字段约束规则如下

```php
            [['images'], 'required', 'on' => ['create'],'enableClientValidation'=>false],
            [['images'], 'defaultHead', 'skipOnEmpty' => false, 'on' => ['edit']],
```

# 参考资料
 - Yii2 源码


