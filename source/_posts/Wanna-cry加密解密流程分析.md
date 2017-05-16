---
title: Wanna cry加密解密流程分析
date: 2017-05-16 10:52:29
tags: 加密解密
---
永恒之蓝勒索病毒文件加密解密流程，核心使用RSA+AES
## 加密流程
* RSA秘钥A，公钥A.public硬编码在加密程序中，私钥A.private作者自己持有
* 每个用户随机生成一对RSA秘钥B，公钥B.public用于加密AES的key，私钥B.private被作者的公钥进行RSA加密B.private.encypt
* 遍历加密后缀列表(没有.torrent)中的文件进行加密，每个文件随机生成一个128位的key，使用B.public对key进行RSA加密得key2
* 对原始文件F使用key2进行AES加密得F.encrypt
* 删除原始文件，把key2和F.encrypt写入新文件

## 解密流程
* 把B.private.encypt发给作者
* 作者使用A.private将B.private.encypt进行RSA解密还原成B.private
* 从被加密的文件中提取key2，使用B.private对key进行RSA解密还原成key
* 使用key对F.encrypt进行AES解密得F（原始文件）

以上可知，只要作者的RSA私钥A.private不泄露，那些被加密的文件理论无法破解
而作者提出使用比特币作为支付方式，是利用比特币的匿名性和无法被冻结，但比特币的交易记录是公开的，作者没有办法把每笔交易和某个感染用户关联，所以即使支付赎金，也可能无法解密文件