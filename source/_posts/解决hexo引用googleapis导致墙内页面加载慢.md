---
title: 解决hexo引用googleapis导致墙内页面加载慢
date: 2017-03-10 17:18:09
tags: hexo
---
  以next主题为例，我们需要修改一处代码和配置文件，
在next主题目录下搜索“fonts.googleapis.com”，在_config.yaf下加上我们自己的字体host,
然后在external-fonts.swig使用这个配置的地方做一些修改，
最后就是下载fonts.googleapis.com对应的字体文件放到我们可以访问到的地方就可以了。  