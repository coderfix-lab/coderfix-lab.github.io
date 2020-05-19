---
layout: post
title:  "【ES】Mac部署ES本地开发环境-安装/docker集群"
date:   2020-05-19 23:33:00 +0800
categories: RaspberryPi
---
本文为本人原创，首发地址 https://coderfix.blog.csdn.net/article/details/106197448


# 环境说明
## mac
```
10.15.4
```

### ES
```
7.7.0
```

# 安装
## homebrew 

### elasticsearch

没有采用安装包安装的原因还是因为这个更方便~

```bash
lixiaoyu@localhost ~ % brew tap elastic/tap
Updating Homebrew...
==> Tapping elastic/tap
Cloning into '/usr/local/Homebrew/Library/Taps/elastic/homebrew-tap'...
remote: Enumerating objects: 156, done.
remote: Counting objects: 100% (156/156), done.
remote: Compressing objects: 100% (104/104), done.
remote: Total 464 (delta 108), reused 84 (delta 52), pack-reused 308
Receiving objects: 100% (464/464), 125.61 KiB | 400.00 KiB/s, done.
Resolving deltas: 100% (340/340), done.
Tapped 19 formulae (46 files, 228.2KB).
lixiaoyu@localhost ~ % brew install elastic/tap/elasticsearch-full
==> Installing elasticsearch-full from elastic/tap
==> Downloading https://artifacts.elastic.co/downloads/elasticsearch/elasticsear
######################################################################## 100.0%
Warning: tried to install empty array to /usr/local/etc/elasticsearch/jvm.options.d
==> codesign -f -s - /usr/local/Cellar/elasticsearch-full/7.7.0/libexec/modules/
==> find /usr/local/Cellar/elasticsearch-full/7.7.0/libexec/jdk.app/Contents/Hom
==> Caveats
Data:    /usr/local/var/lib/elasticsearch/elasticsearch_lixiaoyu/
Logs:    /usr/local/var/log/elasticsearch/elasticsearch_lixiaoyu.log
Plugins: /usr/local/var/elasticsearch/plugins/
Config:  /usr/local/etc/elasticsearch/

To have launchd start elastic/tap/elasticsearch-full now and restart at login:
  brew services start elastic/tap/elasticsearch-full
Or, if you don't want/need a background service you can just run:
  elasticsearch
==> Summary
🍺  /usr/local/Cellar/elasticsearch-full/7.7.0: 987 files, 472.4MB, built in 105 minutes 15 seconds
lixiaoyu@localhost ~ % brew services start elastic/tap/elasticsearch-full
==> Tapping homebrew/services
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-services'...
remote: Enumerating objects: 35, done.
remote: Counting objects: 100% (35/35), done.
remote: Compressing objects: 100% (26/26), done.
remote: Total 726 (delta 8), reused 23 (delta 4), pack-reused 691
Receiving objects: 100% (726/726), 200.78 KiB | 187.00 KiB/s, done.
Resolving deltas: 100% (280/280), done.
Tapped 1 command (40 files, 276.2KB).
==> Successfully started `elasticsearch-full` (label: homebrew.mxcl.elasticsearc

```

打开浏览器，打开地址`http://localhost:9200/`，就表示安装成功，服务成功运行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518172627499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)

停止服务

```bash
lixiaoyu@localhost ~ % brew services stop  elastic/tap/elasticsearch-full
Stopping `elasticsearch-full`... (might take a while)
==> Successfully stopped `elasticsearch-full` (label: homebrew.mxcl.elasticsearch-full)
```

### Kibana
Kibana 是为 Elasticsearch设计的开源分析和可视化平台。

```bash
lixiaoyu@localhost ~ % brew install elastic/tap/kibana-full
Updating Homebrew...
==> Installing kibana-full from elastic/tap
==> Downloading https://artifacts.elastic.co/downloads/kibana/kibana-7.7.0-darwin-x86_64.tar.gz?tap=elastic/homebrew
######################################################################## 100.0%

==> Caveats
Config: /usr/local/etc/kibana/
If you wish to preserve your plugins upon upgrade, make a copy of
/usr/local/opt/kibana-full/plugins before upgrading, and copy it into the
new keg location after upgrading.

To have launchd start elastic/tap/kibana-full now and restart at login:
  brew services start elastic/tap/kibana-full
Or, if you don't want/need a background service you can just run:
  kibana
==> Summary
🍺  /usr/local/Cellar/kibana-full/7.7.0: 149,308 files, 890.1MB, built in 69 minutes 38 seconds
lixiaoyu@localhost ~ % brew services start elastic/tap/kibana-full
==> Successfully started `kibana-full` (label: homebrew.mxcl.kibana-full)
```
Kibana安装完成，打开网址`localhost:5601`即可进行下一步操作。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518183038297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpYW5kaWFueGl5dQ==,size_16,color_FFFFFF,t_70)



## Docker
在一个文件夹内新建文件`docker-compose.yml`,内容如下

```yml
version: '2.2'
services:
  cerebro:
    image: lmenezes/cerebro:0.8.3  #  cerebro镜像
    container_name: cerebro
    ports:
      - "36006:9000"  #  容器端口映射到本地36006端口
    command:
      - -Dhosts.0.host=http://elasticsearch:5001  # 容器内部的ES地址
    networks:
      - es72net
  kibana:
    image: docker.elastic.co/kibana/kibana:7.2.0  # kibana7.2镜像
    container_name: kibana72  # 容器的名称
    environment:    #  容器内部环境变量
      #- I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED=true   
      - SERVER_PORT=5005     # Kibana端口
      - ELASTICSEARCH_HOSTS=http://elasticsearch:5001   # ES地址
    ports:
      - "36005:5005"   #  容器端口5005映射到本地36005端口
    networks:
      - es72net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0  # ES 7.2镜像
    container_name: es72_01 # 容器名称
    environment:
      - cluster.name=web_news_query  # 集群名称
      - node.name=news_5001_data   #  节点名称
      - http.port=5001    # 节点端口
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=-Xms512m -Xmx512m  # 内存分配
      - discovery.seed_hosts=es72_01,es72_02  
      - network.publish_host=elasticsearch  
      - cluster.initial_master_nodes=news_5001_data,news_5002_data
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es72data1:/es_data
    ports:
      - 36001:5001
    networks:
      - es72net
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
    container_name: es72_02
    environment:
      - cluster.name=web_news_query
      - node.name=news_5002_data 
      - http.port=5002
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
      - discovery.seed_hosts=es72_01,es72_02 
      - cluster.initial_master_nodes=news_5001_data,news_5002_data 
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es72data2:/es_data
    ports:
      - 36002:5002
    networks:
      - es72net

volumes:
  es72data1:
    driver: local
  es72data2:
    driver: local

networks:
  es72net:
    driver: bridge

```

在目录下执行

```
docker-compose up
```

稍等片刻，服务就运行起来了。

# 总结
如果你的开发机内存比较大，比如16G，推荐安装docker环境，更加轻便。
如果你的开发机内存不大，比如8G或者更少，推荐本地安装。

# 参考资料
- https://www.elastic.co/guide/en/kibana/7.7/brew.html
- https://www.elastic.co/cn/elasticsearch