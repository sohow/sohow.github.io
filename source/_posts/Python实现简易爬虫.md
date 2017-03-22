---
title: Python实现简易爬虫
date: 2017-03-22 17:17:45
tags: [Python, 爬虫, 代理IP]
---
使用Python爬取代理IP， 爬取的代理IP来源网站一个是国内的一个是国外的，注意爬取国外的需要翻墙，
如无法翻墙可选择注释爬取国外的相关代码, 使用了两个第三方库requests和BeautifulSoup4，
简单说明一下，requests库可以更方便操作HTTP请求,
BeautifulSoup4库用来解析HTML，类似于JavaScript操作DOM一样方便，免去正则匹配的繁琐
``` Python
#!/usr/bin/python
#coding=utf-8

import json
import requests
from bs4 import BeautifulSoup

class Spider:
    def get_ips(self, url, table_id):
        headers = {'user-agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.90 Safari/537.36'}
        html = requests.get(url, headers=headers)
        soup = BeautifulSoup(html.text, 'html.parser')
        table = soup.find(id=table_id)
        if not table:
            return []
        tbody = table.find('tbody')
        if not tbody:
            tbody = table
        ips = []
        for tr in tbody.find_all('tr'):
            ip = []
            for td in tr.find_all('td'):
                ip.append(td.text)
            if len(ip):
                ips.append(ip)
        return ips

    def get_sslproxies(self):
        url = 'http://www.sslproxies.org/'
        table_id = "proxylisttable"
        return self.get_ips(url, table_id)

    def get_xicidaili(self):
        url = 'http://www.xicidaili.com/wn/'
        table_id = "ip_list"
        return self.get_ips(url, table_id)

    def write_file(self, str):
        fp = open('ips.txt', 'w+')
        fp.write(str)
        fp.close()

    def run(self):
        sslproxies = self.get_sslproxies()
        xicidaili = self.get_xicidaili()
        ips = {}
        ips['sslproxies'] = sslproxies
        ips['xicidaili'] = xicidaili
        ips = json.dumps(ips)
        self.write_file(ips)

spider = Spider()
spider.run()
```


