---
title: 使用coding.net解决百度爬虫无法抓取githubpage
date: 2017-03-15 10:20:22
tags: hexo
---
假设你已经在github上使用hexo搭建好了一个可以访问的page页，
我们使用百度的站长平台的抓取诊断功能，诊断如下链接：yourusername.github.io ,
会提示抓取失败，原因就是github pages屏蔽的百度的爬虫，毕竟自身的安全才是最重要的，具体原因大家都懂得。
coding.net是国内的一个类似github的，也支持pages服务，看到这可能也都清楚了，怎么做来解决百度爬虫无法抓取githubpage了，
简单来说就是，github page和coding.net page绑定到同一个域名A，
使用域名解析服务DNSPod把域名A根据不同的线路cname到github page或者coding.net page,
在此我们选择百度线路的cname到coding.net , 其它cname到github page, 
域名可以自己购买或者使用免费的如tk顶级域名,
最后在hexo的配置文件里修改代码的部署，同时部署到github和coding.net, 如下
deploy:
  type: git
  repo:
    github: git@github.com:yourname/yourname.github.io.git,master
    coding: git@git.coding.net:yourname/yourname.coding.me.git,master
