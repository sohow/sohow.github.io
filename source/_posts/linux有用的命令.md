---
title: linux有用的命令
date: 2017-05-12 11:17:25
tags: linux
---
## lsattr和chattr
这两个命令是用来查看和改变文件、目录属性的，与chmod这个命令相比，chmod只是改变文件的读写、执行权限，更底层的属性控制是由chattr来改变的
### 查看文件属性
``` shell
lsattr /etc/sudoers
```
### 设置文件只读属性
``` shell
chattr -i /etc/sudoers
chattr +i /etc/sudoers
```

## nc
``` shell
-v  显示指令执行过程。
-w  <超时秒数>   设置等待连线的时间。
-u  表示使用UDP协议
-z  使用0输入/输出模式 (用来告诉 nc 报告开放的端口，而不是启动连接)
```
### 端口扫描
``` shell
 nc -v -w 10 -z 192.168.0.100 8080  
 nc -w 2 -z 192.168.0.100 1-65535 
```
### 传输文件
* 监听
``` shell
nc -v -l -p 4444 > demo.txt
```
* 连接
``` shell
nc -v 192.168.0.100 4444 < demo.txt
```

## find
### 查找目录下的所有文件中是否含有某个字符串 
``` shell
find .|xargs grep -ri "IBM" 
```
### 查找目录下的所有文件中是否含有某个字符串,并且只打印出文件名 
``` shell
find .|xargs grep -ri "IBM" -l 
```
### 查找文件
``` shell
find . -name "*.txt"                    ;按照文件名
find . -empty                           ;查找空文件
find . -user himanshu -name "*.txt"     ;指定用户
find . -szie -1024c + 256c              ;指定文件大小范围>256字节<1024字节
```

## ll
### 文件排序
``` shell
ll -S
```
### 统计文件个数
``` shell
ll | grep "^-" | wc -l
```

## notify-send
* 每周4下午3点显示一个提醒弹窗
``` shell
00 15 * * 4 export DISPLAY=:0.0 && sudo -u zongwen1 /usr/bin/notify-send "[自定义提醒]" "该写周报了！"
```
* mac版弹窗命令
``` shell
osascript -e 'display notification "该写周报了！" with title "[自定义提醒]"'
```

## zssh
* zssh 配合 sz和rz 可以方便地通过shell传输文件

## alias
设置命令别名，可缩短命令长度
例如
``` shell
sudo vim /home/当前用户/.bashrc     ;对当前用户有效
sudo vim /etc/profile              ;全局用户有效
source /etc/profile
alias sqlmap=’python /path/sqlmap.py’
```

## sqlmap
* 需要python环境，下载后即可使用了

``` shell
git clone https://github.com/sqlmapproject/sqlmap.git
cd sqlmap
python sqlmap.py -u "http://192.168.0.1/?id=1" ;建议使用alias设置别名
```
