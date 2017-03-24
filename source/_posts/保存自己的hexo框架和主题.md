---
title: 保存自己的hexo框架和主题
date: 2017-03-24 11:23:01
tags: hexo
---
当我们在家里的电脑上安装完hexo和主题后，如果换一台电脑就不能继续写作了，
因此我把hexo框架和使用next主题以及自定义的配置文件也放在github，
首先在hexo网站的github仓库下新建一个分支，如: hexo
然后fork一份你所使用的主题，如: next
接下来你应该知道怎么做了吧，
把hexo框架代码提交到hexo分支
使用git submodule 拉取你fork的next主题

注意： 
1. 建立.gitignore, 
如我的内容如下：  public/
                .idea/
                .deploy_git/
                db.json
2. 使用时只clone hexo分支， git clone -b branch_name
3. clone完hexo分支后，需要 git submodule update --init , 初始化子模块next
4. 修改submodule next前需要切换到next的master
5. 修改submodule next后，要在hexo分支下进行一次提交 
