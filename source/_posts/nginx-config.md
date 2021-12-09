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

        # cross-origin
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





