---
title: nginx config
tags:
  - nginx
  - linux
categories:
  - linux
  - nginx
date: 2020-06-22 18:44:29
---

## 请求代理模式

`proxy_pass`中只有IP和端口, 称为不带URI, 如:`proxy_pass http://1270.0.0.1:8080`, 另外一种是端口号之后包含其他路径的, 如: `proxy_pass http://127.0.0.1:8080/`(哪怕只带了一个`/`) 或 `proxy_pass http://127.0.0.1:8080/test`

- **对于不带URI的方式, nginx会保留location中的路径部分**, 如: 

  ```nginx
  location /api1/ {
      proxy_pass http://127.0.0.1:8080;
  }
  http://127.0.0.1/api1/xxx  ->  http://127.0.0.1:8080/api1/xxx
  ```

- **对于带URI的方式, nginx会对URL进行替换**

  ```nginx
  location /api2/ {
      proxy_pass http://127.0.0.1:8080/;
  }
  http://127.0.0.1/api2/xxx  ->  http://127.0.0.1:8080/xxx
  ```

  `http://127.0.0.1/api2/`被替换成`http://127.0.0.1:8080/`, 再加上剩下的`xxx`, 最终成为`http://127.0.0.1:8080/xxx`

  又比如:

  ```nginx
  location /api3/ {
      proxy_pass http://127.0.0.1:8080/test;
  }
  http://127.0.0.1/api3/xxx  -> http://127.0.0.1:8080/testxxx
  ```

  `http://127.0.0.1/api3/`被替换成了`http://127.0.0.1:8080/test`, 再加上剩余的`xxx`, 就变成了`http://127.0.0.1:8080/testxxx`

### 规则总结

nginx代理8080端口, 转发到8989端口, 客户端请求地址: http://localhost:8080/api/user, nginx配置格式如下:

```nginx
server {
    listen       8080;
    access_log  logs/host.access.log  main;

	location /api {
    	proxy_pass http://localhost:8989/service/;
	}
}
```

`location`和`proxy_pass`不同配置搭配, 转发结果如下:

|                                            | location /api                          | location /api/                         |
| ------------------------------------------ | -------------------------------------- | -------------------------------------- |
| proxy_pass http://localhost:8989;          | http://localhost:8989/api/user         | http://localhost:9809/api/user         |
| proxy_pass http://localhost:8989/;         | http://localhost:8989//user            | **http://localhost:8989/user**         |
| proxy_pass http://localhost:8989/service;  | **http://localhost:8989/service/user** | http://localhost:8989/serviceuser      |
| proxy_pass http://localhost:8989/service/; | http://localhost:8989/service//user    | **http://localhost:8989/service/user** |

> 建议: **location 和 proxy_pass结尾斜线保持一致**

## 本地资源模式

```nginx
location / {
    root   /tmp/html;	# root /tmp/html/; 结果一样
}
localhost:8080  ->  /tmp/html/index.html
localhost:8080/index.html  ->  /tmp/html/index.html
localhost:8989/api  -> 301重定向http://localhost:8989/api/
localhost:8989/api/  ->  /tmp/html/api/index.html
localhost:8989/api/index.html  ->  /tmp/html/api/index.html
```

```nginx
# location 和 root 结尾有没有斜线都一样
location /img {
    root   /tmp/html;
}
localhost:8989/img/a.jpg  ->  /tmp/html/img/a.jpg
localhost:8989/img/api/a.jpg  -> /tmp/html/img/api/a.jpg
```

```nginx
location /img {
    alias	/tmp/html/;
}
localhost:8989/img/a.jpg  ->  /tmp/html/a.jpg
localhost:8989/img/api/a.jpg  -> /tmp/html/api/a.jpg
```

> - root配置时, 访问的资源是root+location目录下的
>
> - alias配置时, 访问的资源是alias目录下. **如果Path不加斜线则不影响, 如果Path以斜线结尾, alias必须以斜线结尾**

## nginx路由匹配规则

常见的路由匹配符号有:

1. `＝`：精确匹配
2. `^~`：精确前缀匹配
3. `~`：区分大小写的正则匹配;  `~*`：不区分大小写的正则匹配
4. `/uri`：普通前缀匹配
5. `/`：通用匹配

```nginx
location = / {
   echo "规则A";
}
location = /login {
   echo "规则B";
}
location ^~ /static/ {
   echo "规则C";
}
location ^~ /static/files {
    echo "规则X";
}
location ~ \.(gif|jpg|png|js|css)$ {
   echo "规则D";
}
location ~* \.png$ {
   echo "规则E";
}
location /img {
    echo "规则Y";
}
location / {
   echo "规则F";
}
```


| 请求URI                             | 匹配路由规则 |
| ----------------------------------- | ------------ |
| http://localhost/                   | 规则A        |
| http://localhost/login              | 规则B        |
| http://localhost/register           | 规则F        |
| http://localhost/static/a.html      | 规则C        |
| http://localhost/static/files/a.txt | 规则X        |
| http://localhost/a.png              | 规则D        |
| http://localhost/a.PNG              | 规则E        |
| http://localhost/img/a.gif          | 规则D        |
| http://localhost/img/a.tiff         | 规则Y        |


> 同优先级精确度越高, 优先级越高
>
> 同级别的定义顺序越靠前, 优先级越高

## 负载均衡配置

```nginx
upstream groups1 {
    #  配置了不同权重
    server 192.168.1.110:80 weight=10;
    server 192.168.1.111:81 weight=3;
}

location /a/ {
    proxy_pass http://groups1/;
}
```

## 文件服务器配置

```nginx
server {
  listen 80; 
  server_name 10.1.2.3; # 自己PC的ip或者服务器的域名 
  charset utf-8; # 避免中文乱码 
  root /home/xx/share; # 存放文件的目录 
  location / { 
    autoindex on; # 索引 
    autoindex_exact_size on; # 显示文件大小 
    autoindex_localtime on; # 显示文件时间 
  }
}
```



### 典型配置

可以通过[digitalocean](https://www.digitalocean.com/community/tools/nginx)的页面配置需要的功能, 然后下载生成好的配置文件

```nginx
user                 www-data;
pid                  /run/nginx.pid;
worker_processes     auto;
worker_rlimit_nofile 65535;

# Load modules
include              /etc/nginx/modules-enabled/*.conf;

events {
    multi_accept       on;
    worker_connections 65535;
}

http {
	# load balence
	upstream user {
		server 192.168.1.100:8080 weight=1 max_fails=2 fail_timeout=30s;
		server 192.168.1.101:8080 weight=1 max_fails=2 fail_timeout=30s;

		# nginx TO server 最大空闲连接数, 长连接数量的10%-30%
		keepalive	300;
	}

	# client TO nginx
    keepalive_timeout  120s 120s;
    keepalive_requests 10000;

    charset                utf-8;
    sendfile               on;
    tcp_nopush             on;
    tcp_nodelay            on;
    server_tokens          off;
    types_hash_max_size    2048;
    types_hash_bucket_size 64;
    client_max_body_size   16M;

    # MIME
    include                mime.types;
    default_type           application/octet-stream;

    # Logging
    access_log             /var/log/nginx/access.log;
    error_log              /var/log/nginx/error.log warn;

    # Connection header for WebSocket reverse proxy
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ""      close;
    }

    map $remote_addr $proxy_forwarded_elem {

        # IPv4 addresses can be sent as-is
        ~^[0-9.]+$        "for=$remote_addr";

        # IPv6 addresses need to be bracketed and quoted
        ~^[0-9A-Fa-f:.]+$ "for=\"[$remote_addr]\"";

        # Unix domain socket names cannot be represented in RFC 7239 syntax
        default           "for=unknown";
    }

    # Load configs
    include /etc/nginx/conf.d/*.conf;

    # localhost
    server {
        listen                             80;
        listen                             [::]:80;
        server_name                        localhost;
        root                               /var/www/localhost/public;

        # . files
        location ~ /\.(?!well-known) {
            deny all;
        }

        # logging
        access_log /var/log/nginx/localhost.access.log;
        error_log  /var/log/nginx/localhost.error.log warn;

        # index.html fallback
        location / {
            try_files $uri $uri/ /index.html;
        }

        # favicon.ico
        location = /favicon.ico {
            log_not_found off;
            access_log    off;
        }

        # robots.txt
        location = /robots.txt {
            log_not_found off;
            access_log    off;
        }

        # assets, media
        location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
            expires    7d;
            access_log off;
        }

        # svg, fonts
        location ~* \.(?:svgz?|ttf|ttc|otf|eot|woff2?)$ {
            add_header Access-Control-Allow-Origin "*";
            expires    7d;
            access_log off;
        }

        # gzip
        gzip            on;
        gzip_vary       on;
        gzip_proxied    any;
        gzip_comp_level 6;
        gzip_types      text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;

        # cross-origin跨域配置
        add_header Access-Control-Allow-Origin $http_origin;
        add_header Access-Control-Allow-Methods *;
				# 如果不需要跨域传递cookie, 可以删除下面两行, 并把上面的$http_origin 改为*
        add_header Access-Control-Allow-Credentials true;       
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
        if ($request_method = OPTIONS){
            return 204;
        }

        # reverse proxy
        location /test {
            proxy_pass                         http://127.0.0.1:3000;
            proxy_http_version                 1.1;
            proxy_cache_bypass                 $http_upgrade;

            # Proxy headers
            proxy_set_header Upgrade           $http_upgrade;
            proxy_set_header Connection        $connection_upgrade;
            proxy_set_header Host              $host;
            proxy_set_header X-Real-IP         $remote_addr;
            proxy_set_header Forwarded         $proxy_add_forwarded;
            proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host  $host;
            proxy_set_header X-Forwarded-Port  $server_port;

            # Proxy timeouts
            proxy_connect_timeout              60s;
            proxy_send_timeout                 60s;
            proxy_read_timeout                 60s;
        }
    }
}
```





## 编译安装

> 参考: [CentOS 7 下安装 Nginx](https://www.cnblogs.com/eaglezb/p/6073661.html)

### 1. 安装依赖库

```bash
# 安装gcc环境
yum install gcc-c++

# nginx 的 http 模块使用 pcre 来解析正则表达式
yum install -y pcre pcre-devel

# nginx 使用 zlib 对 http 包的内容进行 gzip
yum install -y zlib zlib-devel

# nginx 支持 https 协议需要 openssl
yum install -y openssl openssl-devel
```

#### 依赖以及间接依赖

- pcre-8.32-17.el7.x86_64.rpm
- pcre-devel-8.32-17.el7.x86_64.rpm
- zlib-1.2.7-20.el7_9.x86_64.rpm
- zlib-devel-1.2.7-20.el7_9.x86_64.rpm
- openssl openssl-devel

```
===============================================================================
 Package              架构       版本                 源                    大小
===============================================================================
正在安装:
 openssl-devel        x86_64   1:1.0.2k-25.el7_9    iflytekdc-updates    1.5 M
正在更新:
 openssl              x86_64   1:1.0.2k-25.el7_9    iflytekdc-updates    494 k
为依赖而安装:
 keyutils-libs-devel  x86_64   1.5.8-3.el7          c7.6                  37 k
 krb5-devel           x86_64   1.15.1-54.el7_9      iflytekdc-updates    273 k
 libcom_err-devel     x86_64   1.42.9-19.el7        iflytekdc-os          32 k
 libkadm5             x86_64   1.15.1-54.el7_9      iflytekdc-updates    179 k
 libselinux-devel     x86_64   2.5-15.el7           iflytekdc-os         187 k
 libsepol-devel       x86_64   2.5-10.el7           c7.6                  77 k
 libverto-devel       x86_64   0.2.5-4.el7          c7.6                  12 k
为依赖而更新:
 e2fsprogs            x86_64   1.42.9-19.el7        iflytekdc-os         701 k
 e2fsprogs-libs       x86_64   1.42.9-19.el7        iflytekdc-os         168 k
 krb5-libs            x86_64   1.15.1-54.el7_9      iflytekdc-updates    810 k
 libcom_err           x86_64   1.42.9-19.el7        iflytekdc-os          42 k
 libselinux           x86_64   2.5-15.el7           iflytekdc-os         162 k
 libselinux-python    x86_64   2.5-15.el7           iflytekdc-os         236 k
 libselinux-utils     x86_64   2.5-15.el7           iflytekdc-os         151 k
 libss                x86_64   1.42.9-19.el7        iflytekdc-os          47 k
 openssl-libs         x86_64   1:1.0.2k-25.el7_9    iflytekdc-updates    1.2 M

事务概要
=====================================================================================================================================================
安装  1 软件包 (+7 依赖软件包)
升级  1 软件包 (+9 依赖软件包)

总下载量：6.2 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/18): keyutils-libs-devel-1.5.8-3.el7.x86_64.rpm                    |  37 kB  00:00:00     
(2/18): krb5-devel-1.15.1-54.el7_9.x86_64.rpm                         | 273 kB  00:00:00     
(3/18): e2fsprogs-libs-1.42.9-19.el7.x86_64.rpm                       | 168 kB  00:00:00     
(4/18): e2fsprogs-1.42.9-19.el7.x86_64.rpm                            | 701 kB  00:00:00     
(5/18): krb5-libs-1.15.1-54.el7_9.x86_64.rpm                          | 810 kB  00:00:00     
(6/18): libcom_err-1.42.9-19.el7.x86_64.rpm                           |  42 kB  00:00:00     
(7/18): libcom_err-devel-1.42.9-19.el7.x86_64.rpm                     |  32 kB  00:00:00     
(8/18): libselinux-2.5-15.el7.x86_64.rpm                              | 162 kB  00:00:00     
(9/18): libkadm5-1.15.1-54.el7_9.x86_64.rpm                           | 179 kB  00:00:00     
(10/18): libselinux-python-2.5-15.el7.x86_64.rpm                      | 236 kB  00:00:00     
(11/18): libselinux-devel-2.5-15.el7.x86_64.rpm                       | 187 kB  00:00:00     
(12/18): libselinux-utils-2.5-15.el7.x86_64.rpm                       | 151 kB  00:00:00     
(13/18): libss-1.42.9-19.el7.x86_64.rpm                               |  47 kB  00:00:00     
(14/18): libsepol-devel-2.5-10.el7.x86_64.rpm                         |  77 kB  00:00:00     
(15/18): libverto-devel-0.2.5-4.el7.x86_64.rpm                        |  12 kB  00:00:00     
(16/18): openssl-1.0.2k-25.el7_9.x86_64.rpm                           | 494 kB  00:00:00     
(17/18): openssl-devel-1.0.2k-25.el7_9.x86_64.rpm                     | 1.5 MB  00:00:00     
(18/18): openssl-libs-1.0.2k-25.el7_9.x86_64.rpm                      | 1.2 MB  00:00:00     
-----------------------------------------------------------------------------------------------------------------------------------------------------
总计                                                             18 MB/s | 6.2 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在更新    : libcom_err-1.42.9-19.el7.x86_64                      1/28 
  正在更新    : libselinux-2.5-15.el7.x86_64                         2/28 
  正在更新    : 1:openssl-libs-1.0.2k-25.el7_9.x86_64                3/28 
  正在更新    : krb5-libs-1.15.1-54.el7_9.x86_64                     4/28 
  正在安装    : libkadm5-1.15.1-54.el7_9.x86_64                      5/28 
  正在更新    : e2fsprogs-libs-1.42.9-19.el7.x86_64                  6/28 
  正在更新    : libss-1.42.9-19.el7.x86_64                           7/28 
  正在安装    : libcom_err-devel-1.42.9-19.el7.x86_64                8/28 
  正在安装    : libsepol-devel-2.5-10.el7.x86_64                     9/28 
  正在安装    : libselinux-devel-2.5-15.el7.x86_64                  10/28 
  正在安装    : libverto-devel-0.2.5-4.el7.x86_64                   11/28 
  正在安装    : keyutils-libs-devel-1.5.8-3.el7.x86_64              12/28 
  正在安装    : krb5-devel-1.15.1-54.el7_9.x86_64                   13/28 
  正在安装    : 1:openssl-devel-1.0.2k-25.el7_9.x86_64              14/28 
  正在更新    : e2fsprogs-1.42.9-19.el7.x86_64                      15/28 
  正在更新    : 1:openssl-1.0.2k-25.el7_9.x86_64                    16/28 
  正在更新    : libselinux-utils-2.5-15.el7.x86_64                  17/28 
  正在更新    : libselinux-python-2.5-15.el7.x86_64                 18/28 
  清理        : e2fsprogs-1.42.9-13.el7.x86_64                      19/28 
  清理        : 1:openssl-1.0.2k-16.el7.x86_64                      20/28 
  清理        : krb5-libs-1.15.1-34.el7.x86_64                      21/28 
  清理        : 1:openssl-libs-1.0.2k-16.el7.x86_64                 22/28 
  清理        : e2fsprogs-libs-1.42.9-13.el7.x86_64                 23/28 
  清理        : libss-1.42.9-13.el7.x86_64                          24/28 
  清理        : libselinux-python-2.5-14.1.el7.x86_64               25/28 
  清理        : libselinux-utils-2.5-14.1.el7.x86_64                26/28 
  清理        : libselinux-2.5-14.1.el7.x86_64                      27/28 
  清理        : libcom_err-1.42.9-13.el7.x86_64                     28/28 
  验证中      : libselinux-devel-2.5-15.el7.x86_64                    1/28 
  验证中      : keyutils-libs-devel-1.5.8-3.el7.x86_64                2/28 
  验证中      : libselinux-2.5-15.el7.x86_64                          3/28 
  验证中      : 1:openssl-1.0.2k-25.el7_9.x86_64                      4/28 
  验证中      : libcom_err-1.42.9-19.el7.x86_64                       5/28 
  验证中      : e2fsprogs-1.42.9-19.el7.x86_64                        6/28 
  验证中      : krb5-devel-1.15.1-54.el7_9.x86_64                     7/28 
  验证中      : libverto-devel-0.2.5-4.el7.x86_64                     8/28 
  验证中      : libselinux-utils-2.5-15.el7.x86_64                    9/28 
  验证中      : libkadm5-1.15.1-54.el7_9.x86_64                      10/28 
  验证中      : e2fsprogs-libs-1.42.9-19.el7.x86_64                  11/28 
  验证中      : 1:openssl-libs-1.0.2k-25.el7_9.x86_64                12/28 
  验证中      : libselinux-python-2.5-15.el7.x86_64                  13/28 
  验证中      : 1:openssl-devel-1.0.2k-25.el7_9.x86_64               14/28 
  验证中      : libsepol-devel-2.5-10.el7.x86_64                     15/28 
  验证中      : libss-1.42.9-19.el7.x86_64                           16/28 
  验证中      : krb5-libs-1.15.1-54.el7_9.x86_64                     17/28 
  验证中      : libcom_err-devel-1.42.9-19.el7.x86_64                18/28 
  验证中      : 1:openssl-libs-1.0.2k-16.el7.x86_64                  19/28 
  验证中      : 1:openssl-1.0.2k-16.el7.x86_64                       20/28 
  验证中      : libss-1.42.9-13.el7.x86_64                           21/28 
  验证中      : libselinux-python-2.5-14.1.el7.x86_64                22/28 
  验证中      : e2fsprogs-libs-1.42.9-13.el7.x86_64                  23/28 
  验证中      : krb5-libs-1.15.1-34.el7.x86_64                       24/28 
  验证中      : libselinux-utils-2.5-14.1.el7.x86_64                 25/28 
  验证中      : libcom_err-1.42.9-13.el7.x86_64                      26/28 
  验证中      : libselinux-2.5-14.1.el7.x86_64                       27/28 
  验证中      : e2fsprogs-1.42.9-13.el7.x86_64                       28/28 

已安装:
  openssl-devel.x86_64 1:1.0.2k-25.el7_9                                                                                                             

作为依赖被安装:
  keyutils-libs-devel.x86_64 0:1.5.8-3.el7           krb5-devel.x86_64 0:1.15.1-54.el7_9            libcom_err-devel.x86_64 0:1.42.9-19.el7          
  libkadm5.x86_64 0:1.15.1-54.el7_9                  libselinux-devel.x86_64 0:2.5-15.el7           libsepol-devel.x86_64 0:2.5-10.el7               
  libverto-devel.x86_64 0:0.2.5-4.el7               

更新完毕:
  openssl.x86_64 1:1.0.2k-25.el7_9                                                                                                                   

作为依赖被升级:
  e2fsprogs.x86_64 0:1.42.9-19.el7      e2fsprogs-libs.x86_64 0:1.42.9-19.el7 krb5-libs.x86_64 0:1.15.1-54.el7_9   libcom_err.x86_64 0:1.42.9-19.el7
  libselinux.x86_64 0:2.5-15.el7        libselinux-python.x86_64 0:2.5-15.el7 libselinux-utils.x86_64 0:2.5-15.el7 libss.x86_64 0:1.42.9-19.el7     
  openssl-libs.x86_64 1:1.0.2k-25.el7_9
```



### 2. 下载源码编译安装

```bash
wget -c http://nginx.org/download/nginx-1.22.0.tar.gz

tar -xvf nginx-1.22.0.tar.gz

cd nginx-1.22.0

# 使用默认配置(推荐)
./configure

# 使用自定义配置, 将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录
./configure \
--prefix=/usr/local/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--pid-path=/usr/local/nginx/conf/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi

# 编译安装
make 
make install

# 查找安装路径
whereis nginx
```

### 3. 管理

```bash
cd /usr/local/nginx/sbin/

# 启动
./nginx 

# 强制杀掉进程
./nginx -s stop

# 待nginx进程处理任务完毕进行停止
./nginx -s quit

# 测试配置文件是否有语法错误
./nginx -t

# 重新加载配置文件
./nginx -s reload
```

### 4. 开机启动

```bash
vim /etc/rc.local
# 添加如下一行
/usr/local/nginx/sbin/nginx

# 设置执行权限
chmod 755 rc.local
```





