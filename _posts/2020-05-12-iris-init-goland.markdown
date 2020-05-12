---
layout: post
title:  "【GO】Iris框架项目初始化并解决GoLand的代码提示问题"
date:   2020-05-12 10:53:00 +0800
categories: go iris
---

本文为本人原创，首发地址 https://coderfix.blog.csdn.net/article/details/106071006

# 为什么使用Iris
因为它是少数Go框架中支持MVC的框架，并且是最快的go框架

# 具体操作
## 创建项目

创建项目 IDE采用goland，直接选择第一个来创建项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512103538331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
## 初始化mod
go mod 可以让你摆脱GOPATH对项目的约束，同时也是解决GoLand问题的核心

```
export GO111MODULE=on
lixiaoyu@localhost projectApi % go mod init projectApi
go: creating new go.mod: module projectApi
```
## 引入Iris

```
 go get -u github.com/kataras/iris
```
go会从github中下载文件到本地，执行完成之后，我们得到的`go.mod`内容如下

```
module projectApi

go 1.14

require (
   github.com/BurntSushi/toml v0.3.1 // indirect
   github.com/Joker/jade v1.0.0 // indirect
   github.com/Shopify/goreferrer v0.0.0-20181106222321-ec9c9a553398 // indirect
   github.com/aymerick/raymond v2.0.2+incompatible // indirect
   github.com/eknkc/amber v0.0.0-20171010120322-cdade1c07385 // indirect
   github.com/fatih/structs v1.1.0 // indirect
   github.com/flosch/pongo2 v0.0.0-20200509134334-76fc00043fe1 // indirect
   github.com/gorilla/schema v1.1.0 // indirect
   github.com/iris-contrib/blackfriday v2.0.0+incompatible // indirect
   github.com/iris-contrib/formBinder v5.0.0+incompatible // indirect
   github.com/iris-contrib/go.uuid v2.0.0+incompatible // indirect
   github.com/json-iterator/go v1.1.9 // indirect
   github.com/juju/errors v0.0.0-20200330140219-3fe23663418f // indirect
   github.com/kataras/golog v0.0.13 // indirect
   github.com/kataras/iris v11.1.1+incompatible // indirect
   github.com/klauspost/compress v1.10.5 // indirect
   github.com/microcosm-cc/bluemonday v1.0.2 // indirect
   github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd // indirect
   github.com/modern-go/reflect2 v1.0.1 // indirect
   github.com/ryanuber/columnize v2.1.0+incompatible // indirect
   github.com/shurcooL/sanitized_anchor_name v1.0.0 // indirect
   golang.org/x/crypto v0.0.0-20200510223506-06a226fb4e37 // indirect
   golang.org/x/net v0.0.0-20200506145744-7e3656a0809f // indirect
   golang.org/x/sys v0.0.0-20200511232937-7e40ca221e25 // indirect
   golang.org/x/text v0.3.2 // indirect
   gopkg.in/yaml.v2 v2.2.8 // indirect
)
```

## 创建项目入口main.go
main.go的内容如下,这里仅仅是个示例，目的是展示入口引入的库。

```
package main

import (
	"github.com/kataras/iris"
	"github.com/kataras/iris/mvc"

	"github.com/kataras/iris/middleware/logger"
	"github.com/kataras/iris/middleware/recover"
)


func newApp() *iris.Application {
	app := iris.New()
	app.Use(recover.New())
	app.Use(logger.New())

	mvc.New(app).Handle(new(ExampleController))
	return app
}

func main() {
	app := newApp()

	app.Run(iris.Addr(":8888"))
}

type ExampleController struct{}

func (c *ExampleController) Get() mvc.Result {
	return mvc.Response{
		ContentType: "text/html",
		Text:        "<h1>Welcome</h1>",
	}
}

func (c *ExampleController) GetPing() string {
	return "pong"
}

func (c *ExampleController) GetHello() interface{} {
	return map[string]string{"message": "Hello Iris!"}
}

func (c *ExampleController) BeforeActivation(b mvc.BeforeActivation) {
	anyMiddlewareHere := func(ctx iris.Context) {
		ctx.Application().Logger().Warnf("Inside /custom_path")
		ctx.Next()
	}
	b.Handle("GET", "/custom_path", "CustomHandlerWithoutFollowingTheNamingGuide", anyMiddlewareHere)

}

func (c *ExampleController) CustomHandlerWithoutFollowingTheNamingGuide() string {
	return "hello from the custom handler without following the naming guide"
}
```

这时候我们发现相关的引用是错误的，我们也不能使用代码提示。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512104106748.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

但是项目是可以运行的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512104250650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

## 外部库下载到本地
虽然现在能用，但是作为开发者的体验极差，所以我们需要把库引入到项目的目录下。

```
lixiaoyu@localhost projectApi % go mod vendor 
```
执行之后，我们会发现项目的根目录下增加了一个`vendor`的文件夹，文件夹的名字和PHP的composer生成的文件夹相同，当然作用也是一样的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512104603264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

这时`main.go`的引用和代码提示也都正常了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051210471881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)
## 回顾
经过这一系列的操作，我们得到了一个项目基础。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512104818620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

包含了

- 入口文件
- go.mod
- vendor

然后在这基础上可以进行功能的开发了。

# 参考资料
- https://iris-go.com/
- https://golang.google.cn/cmd/go/