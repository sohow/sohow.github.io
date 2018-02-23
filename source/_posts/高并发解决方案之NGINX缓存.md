---
title: 高并发解决方案之NGINX缓存
date: 2018-02-23 17:02:08
tags: [性能,缓存]
---

## 简介
使用ngx_lua模块在Nginx层做缓存，可动态控制缓存开关，可做静态方案或者降级方案，
在公司的一个专题页项目中使用该方案，QPS提高了20倍
## 架构图
![](/images/nginx_cache.png)


## Lua代码文件

### header_filter.lua文件
``` Lua
ngx.header.NgxCache = ngx.var.is_cache

```

### access.lua文件
``` Lua
function get_from_cache(key)
        local cache_ngx = ngx.shared.ngx_cache
        local value = cache_ngx:get(key)
        return value
end

function set_to_cache(key, value, exptime)
        if not exptime then
                exptime = 0
        end
        local cache_ngx = ngx.shared.ngx_cache
        local succ, err, forcible = cache_ngx:set(key, value, exptime)
        return succ
end

function safe_add(key, value, exptime)
        if not exptime then
                exptime = 0
        end
        local cache_ngx = ngx.shared.ngx_cache
        local succ, err, forcible = cache_ngx:safe_add(key, value, exptime)
        return succ
end

function get_from_php(k, kk, flag, exptime)
        local options = {}
        options["method"] = ngx.var.request_method == "GET" and ngx.HTTP_GET or ngx.HTTP_POST
        options["body"] = ngx.var.request_body
        if ngx.var.args == nil then
                ngx.var.args = "debug=0"
        end
        local args = ngx.decode_args(ngx.var.args)
        args.subrequest = flag
        options["args"] = args
        local uri = ngx.var.uri
        local res = ngx.location.capture("/index.php"..uri, options)
        if res.status == 200 then
                if flag == "open" then
                        set_to_cache(k, res.body, exptime)
                        set_to_cache(kk, res.body, exptime*3)
                end
                return res.body
        else
                return res.status
        end
end



local open = "open"
local close = "close"
local time = 300

ngx.req.read_body()
local k = ngx.req.get_post_args()["containerid"] or "default"
local ctrlkey = ngx.req.get_uri_args()["ctrlkey"] or ""
if ctrlkey == open or ctrlkey == close then
        set_to_cache("ctrlkey", ctrlkey, 0)
end
if k == "default" then
	time = 10	
end

local uri = string.sub(ngx.var.uri,1,100)
k = ngx.var.request_method .. ngx.var.host .. uri .. k
local content = nil
local sw = close
local kk = k .. "old"

sw = get_from_cache("ctrlkey")

if sw == open then
        content = get_from_cache(k)
        if content ~= nil then
                ngx.var.is_cache = 1
        else
                local atom = safe_add(k .. sw, 1, 5)
                if atom == nil then
                        content = get_from_cache(kk)
                        if content == nil then
                        	content = get_from_php(k, kk, sw, time)
                                ngx.var.is_cache = 4
                        else
                                ngx.var.is_cache = 2
                        end
                else
                        content = get_from_php(k, kk, sw, time)
                        ngx.var.is_cache = 3
                end
        end
else
        content = get_from_php(k, kk, sw, time)
end
ngx.print(content)


```
## Nginx主要配置
``` Shell
lua_code_cache on;
lua_shared_dict ngx_cache 32m;

#...

set $flag 1;
if ($arg_subrequest = "") {
    set $flag "1${flag}";
}

if ($arg_containerid ~ "newartificial") {
    set $flag "1${flag}";
}
if ($arg_containerid = "") {
    set $flag "1${flag}";
}

if ($uri ~ "^/(path.*)$") {
    set $flag "1${flag}";
}
if ($flag = "1111") {
    rewrite "(.*)" $1 break;
}

#...

location ~ /(.*)\.lua$ {
    deny all;
}

location ^~ /path {
    default_type text/html;
    set $is_cache 0;

    header_filter_by_lua_file /XXXXX/lua/header_filter.lua;
    access_by_lua_file /XXXXX/lua/access.lua;
}


```

## 缓存开关控制脚本
``` Shell
#!/bin/bash
# ./ctrl.sh open 开启缓存
# ./ctrl.sh open 关闭缓存

ips=(10.x.x.x 10.x.x.x 10.x.x.x 10.x.x.x 10.x.x.x 10.x.x.x)
if [ x$1 != x ]; then
    state=$1
else
	echo "need param open or close"
    exit 1
fi

state=$1
for ip in ${ips[@]};
do
    curl -v -x "$ip:80" -d "uid=xxx" "http://host/path?ctrlkey=$state"
	echo ""
done
```