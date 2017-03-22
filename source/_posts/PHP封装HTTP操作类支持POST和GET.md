---
title: PHP封装HTTP操作类支持POST和GET
date: 2017-03-22 17:30:56
tags: [PHP, HTTP]
---
PHP封装HTTP操作类支持POST和GET
``` PHP
<?php
class Helper_Http
{
    public static function get($url, $header = array(),&$setcookie = '', $proxy = '')
    {
        $ch = curl_init();
        $needheader = empty($setcookie) ? 0 : 1;
        $opt = array(
            CURLOPT_URL => $url,
            CURLOPT_SSL_VERIFYPEER => false,
            CURLOPT_SSL_VERIFYHOST => false,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_FOLLOWLOCATION => 0,
            CURLOPT_HEADER=>$needheader
        );
        !empty($proxy) && $opt[CURLOPT_PROXY] = $proxy;
        //$header=array('Content-type: text/plain', 'Content-length: 100')
        curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
        curl_setopt_array($ch, $opt);
        $result = curl_exec($ch);

        if ($needheader == 1) {
            list($header, $body) = explode("\r\n\r\n", $result, 2);
            preg_match_all('/Set-Cookie:(.*);/iU', $header, $str);
            if (!empty($str[1])) {
                $setcookie = implode('; ', $str[1]) . ";";
            }
            return $body;
        }
        return $result;
    }

    public static function post($url, $data, $header = array(),&$setcookie = '', $proxy = '')
    {
        $ch = curl_init();
        $needheader = empty($setcookie) ? 0 : 1;
        $opt = array(
            CURLOPT_SSL_VERIFYPEER => false,
            CURLOPT_SSL_VERIFYHOST => false,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_POST => 1,
            CURLOPT_POSTFIELDS => $data,
            CURLOPT_URL => $url,
            CURLOPT_FOLLOWLOCATION=>0,
            CURLOPT_HEADER=>$needheader
        );

        !empty($proxy) && $opt[CURLOPT_PROXY] = $proxy;
        //$header=array('Content-type: text/plain', 'Content-length: 100')
        curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
        curl_setopt_array($ch, $opt);
        $result = curl_exec($ch);

        if ($needheader == 1) {
            list($header, $body) = explode("\r\n\r\n", $result, 2);
            preg_match_all('/Set-Cookie: (.*;)/iU', $header, $str);
            if (!empty($str[1])) {
                $setcookie = implode('; ', $str[1]) . ";";
            }
            return $body;
        }

        return $result;
    }
}
```
