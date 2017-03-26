---
title: HEXO自定义域名使用HTTPS
date: 2017-03-26 11:30:35
tags: hexo
---

首先说明原来的使用的DNSPod可以选择来源线路配置不同的解析规则，
从而可以解决Github屏蔽百度爬虫的问题（在DNSPost处配置来源百度爬虫的解析走国内的coding.net 配置服务）
而现在要选择HTTPS就得使用国外另外一个提供DNS解析的服务（无法选择线路配置解析规则），
所以就要从百度收录和HTTPS中二选一，而我选择了HTTPS，还好还有个Google可以收录

##使用cloudflare提供的免费ssl证书，简述流程如下：

* 设置cloudflare为你的域名的DNS解析服务器
* 设置cloudflare的DNS至Github的A记录
* 设置SSL解析模式为Flexible
* 设置pageRule "Always Use Https"

####使用如下命令查看DNS解析情况:
``` shell
dig sohow.cc
```
####查看DNS解析详情:
``` shell
dig +trace sohow.cc
```
