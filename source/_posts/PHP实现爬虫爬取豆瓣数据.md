---
title: PHP实现爬虫爬取豆瓣数据
date: 2017-03-22 17:18:19
tags: [PHP, 爬虫]
---
PHP实现爬虫爬取豆瓣数据，由于豆瓣有单个IP频次限制，所以使用了代理IP，代理IP的来源于网络公开免费的代理，
另一篇文章介绍了使用Python爬取代理IP, 代码中的Helper_Http参见另外一篇文章
``` PHP
<?php
include_once "http.php";
class Douban {

    public static function get_list($tag)
    {
        $url = 'https://movie.douban.com/j/search_subjects';
        $page_limit = 500;
        $page_start = 0;
        $param = array(
            'type' => 'tv',
            'sort'=>'time',
            'tag'=> $tag,
            'page_limit'=>500,
            'page_start'=>0
        );

        $list = array();
        do {
            $result = self::http_get($url . '?' . http_build_query($param));
            $data = json_decode($result, true);
            $page_start += $page_limit;
            $param['page_start'] = $page_start;
            if (is_array($data['subjects'])) {
                foreach ($data['subjects'] as $subject) {
                    $list[] = $subject['id'];
                }
            }
        } while (!empty($data['subjects']));

        return $list;
    }

    public static function get_info($id)
    {
        $url = 'https://api.douban.com/v2/movie/subject/';
        $result = self::http_get($url . $id);
        return $result;
    }

    public static function http_get($url, $try=10)
    {
        static $proxy = '';
        if ($try <= 0) {
            return '';
        }
        $cookie = '';
        $result = Helper_Http::get($url, array(), $cookie, $proxy);
        $data = json_decode($result, true);
        if (isset($data['msg']) && $data['code'] == 112) {
            $proxy = self::get_proxy();
            return self::http_get($url, $try-1);
        }
        return $result;
    }

    public static function get_proxy()
    {
        //http://www.sslproxies.org/
        //http://www.xicidaili.com/
        static $proxys = array(
            '27.54.185.16:3128',
            '217.33.216.114:8080',
            '144.217.248.180:8080',
            '185.107.80.44:3128',
            '200.229.202.214:8080',
            '51.141.32.241:8080',
            '94.177.243.78:8080',
            '52.58.9.56:8083',
            '201.16.140.120:8080',
            '77.73.66.26:3128',
            '103.14.8.239:8080',
            '51.15.46.137:80',
            '192.34.56.155:80',
            '54.87.138.13:8083'
        );
        static $cur = 0;

        $proxy = $proxys[$cur];
        $cur = ($cur + 1) % count($proxys);
        return $proxy;
    }

	public static function run()
	{
		$tags = array(
			'国产剧',
			'综艺'
		);

		$ids = array();
		foreach ($tags as $tag) {
			$list = self::get_list($tag);
			$ids = array_merge($ids, $list);
		}
		$ids = array_flip($ids);
		foreach ($ids as $id=>$val) {
			$info = self::get_info($id);
			echo $info . "\n";
			sleep(2);
		}
	}

}

Douban::run();
```