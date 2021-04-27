---
layout: post
title: openresty-lua + MD5 + AES
date: 2021-04-27
tags: nginx lua mds5 aes
---

有需求將 圖片加密 裝  nginx lua 看 openresty-lua 最容易

selinux 與 firewalld 就不寫了

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
yum update -y
yum install -y epel-release
yum install -y readline-devel pcre-devel openssl-devel
yum install libpcre3-dev libssl-dev perl make build-essential curl libreadline-dev libncurses5-dev -y
yum install pcre-devel openssl-devel gcc curl
yum install perl-Digest-MD5 -y
tar zxvf ngx_cache_purge-2.3.tar.gz
tar zxvf nginx-accesskey-2.0.5.tar.gz
tar zxvf openresty-1.19.3.1.tar.gz
cd openresty-1.19.3.1
./configure  --with-threads --with-file-aio --add-module=/root/nginx-accesskey-2.0.5 --add-module=/root/ngx_cache_purge-2.3
gmake
gmake install
cd ..
export PATH="/usr/local/openresty/bin:$PATH"
opm get openresty/lua-resty-string
mkdir work
cd work
mkdir conf
mkdir logs
vi conf/nginx.conf
vi /etc/systemd/system/nginx.service
systemctl enable nginx.service
systemctl start nginx.service
ls -l
```

所有檔案在
```
https://github.com/echochio-tw/openresty-lua
```

裡面沒有 最重要的 nginx.onf 節錄一下重要的
```
      location ~ /(.*)\.(jpg|png|jpeg)$ {
        accesskey on;
        accesskey_hashmethod md5;
        accesskey_arg "key";
        accesskey_signature "mypass1234$remote_addr";
	body_filter_by_lua_block{
                        local chunk, eof = ngx.arg[1], ngx.arg[2]
			.........

```

下面是抄的換 .... 去換 jpg 吧 ... 加解密自己寫吧 ... 這裡不貼了
```
worker_processes  1;
events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    log_format  logeverything  '$current_time - $remote_addr \n$request_headers \n\n$request_body\n';
    

    server {
            set $request_headers "";
            set $current_time "";

            server_name proxy.payloads.online;
            listen 80;

            location / {
                proxy_pass http://payloads.online;
                proxy_set_header Host payloads.online;
                header_filter_by_lua_block { ngx.header.content_length = nil}
                body_filter_by_lua_block{
                        local chunk, eof = ngx.arg[1], ngx.arg[2]
                        local buffered = ngx.ctx.buffered
                        if not buffered then
                           buffered = {}
                           ngx.ctx.buffered = buffered
                        end
                        if chunk ~= "" then
                           buffered[#buffered + 1] = chunk
                           ngx.arg[1] = nil
                        end
                        if eof then
                           local whole = table.concat(buffered)
                           ngx.ctx.buffered = nil
			               whole = string.gsub(whole, ".jpg",  ".jpg?key="..key_md5.."&time="..encrypt_time)
                           ngx.arg[1] = whole
                        end
                }
                log_by_lua_block {
                    ngx.var.current_time = ngx.localtime()                  
                    ngx.var.request_headers = ngx.req.raw_header()
                }
        }
        access_log /tmp/pricking/access.log logeverything;
    }
}
```
