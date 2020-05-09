---
layout: post
title:  "【Yii2】yii2-editable在GridView上修改数值并刷新页面"
date:   2020-05-09 21:12:06 +0800
categories: docker crontab
---

本文为本人原创，首发地址 https://blog.csdn.net/diandianxiyu_geek/article/details/105714005

# 问题描述
需要定时执行docker内的命令，已经在宿主上编辑

```
crontab -e
```

经过查看命令任务确实存在

```
crontab -l
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200423181104981.png)
但是就是不执行

# 解决方案
docker执行命令去掉 `-it`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200423181244713.png)


问题解决。