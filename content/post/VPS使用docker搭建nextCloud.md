---
title: "VPS使用docker安装nextCloud"
date: 2017-12-01T19:47:25+08:00
lastmod: 2017-12-01T19:47:25+08:00
draft: false
tags: ["VPS", "docker", "nextCloud"]
categories: ["技术"]

weight: 10

# you can close something for this content if you open it in config.toml.
comment: false
toc: true
# reward: false
# mathjax: false
comments: false
# icon: book
---
<!-- toc -->
## 准备
CentOS 7.4 x64 ([DigitalOcean VPS](https://m.do.co/c/00a39cb0d92f))

[nextCloud客户端](https://nextcloud.com/install/#install-clients)

## 工具

本人采用的iOS应用HyperApp，HyperApp提供了管理VPS 的GUI，能可视化安装配置docker应用，非常简单便捷。

## 安装MySQL

![mark](http://7xnocv.com1.z0.glb.clouddn.com/blog/171205/GC1m21I9gJ.PNG)

## 调整MySQL内存

对于内存只有512M的VPS，启动MySQL需要调整内存，土豪请忽略。

``` sh
#1. 查看MySQL容器名称，输入以下命令，看到MySQL对应的NAMES
docker container ls
#2. 复制容器内my.cnf配置文件至主机
docker cp 你的MySQL容器名称:/etc/mysql/mysql.conf.d/mysqld.cnf .
#编辑或添加相关设置参数
vi my.cnf
#3. 在[mysqld]内添加如下参数
performance_schema=OFF 
query_cache_size=32M
key_buffer_size=32M
table_definition_cache=600
table_open_cache=400
#4. 将修改后的配置文件复制进容器内
docker cp mysqld.cnf 你的MySQL容器名称:/etc/mysql/mysql.conf.d/mysqld.cnf
#5.  重启容器
docker restart 你的MySQL容器名称
#6. 检测容器占用资源情况
docker stats 你的MySQL容器名称
```

## 安装nextCloud

具体安装步骤请移步[HyperApp-Guide](https://github.com/waylybaye/HyperApp-Guide/blob/master/zh/nextcloud.md)

![mark](http://7xnocv.com1.z0.glb.clouddn.com/blog/171205/8ED084IFj4.PNG)

## 问题及解决办法

### 无法上传大文件

上传小于1M的文件没问题，大于1M的报错”HTTP Entity too large“，我遇到的这个问题是由jrcs/letsencrypt-nginx-proxy-companion未修改配置导致的。

解决办法：在/srv/docker/nginx/vhost.d/下新建配置文件，文件名：你的nextCloud域名，文件内容如下。

`client_max_body_size 20M;`

根据自身情况配置，我这里配置的是最大20M。

### 申请ssl证书出错

TOS hash mismatch. Found: cc88d8d9517f490191401e7b54e9ffd1a2b9082ec7a1d4cec6101f9f1647e7c. 
解决办法：升级 jrcs/letsencrypt-nginx-proxy-companion

### nextCloud客户端

在配置nextCloud客户端时，输入https://我的nextCloud域名，报错“not fund ownCloud/status.php”

这是由于nextCloud所在服务器由于未能成功申请到证书https域名未生效所致。

解决办法：看上面“申请ssl证书出错”的解决办法，或者使用http域名。
