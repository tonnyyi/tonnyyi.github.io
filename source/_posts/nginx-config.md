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

## nginx路由匹配规则

常见的路由匹配符号有:

1. `＝`：精确匹配
2.  `^~`：精确前缀匹配
3.  `~`：区分大小写的正则匹配;  `~*`：不区分大小写的正则匹配
4.  `/uri`：普通前缀匹配
5.   `/`：通用匹配

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


| 请求URI | 匹配路由规则 |
|-------|--------|
| http://localhost/	 | 规则A |
| http://localhost/login | 规则B |
| http://localhost/register | 规则F |
| http://localhost/static/a.html | 规则C |
| http://localhost/static/files/a.txt | 规则X |
| http://localhost/a.png | 规则D |
| http://localhost/a.PNG | 规则E |
| http://localhost/img/a.gif | 规则D |
| http://localhost/img/a.tiff | 规则Y |


> 同优先级精确度越高, 优先级越高
>
> 同级别的定义顺序越靠前, 优先级越高


## 反向代理配置
访问 http://xxx/a 时
```nginx
location /a {
    root /data/www/html;
    add_header Cache-Control no-store;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    proxy_pass http://192.168.1.100:80;
}
```
转到了 http://192.168.1.100:80/a

```nginx
location /a/ {
    proxy_pass http://192.168.1.100:80/;
}
```
则转到了 http://192.168.1.100:80/, 注意**规则里的两个`/`**



```nginx
location /proxy/ {
    proxy_pass http://test.com/;
}

http://test.com/proxy/index.html  ->  http://test.com/index.html
```

```nginx
location /proxy/ {
    proxy_pass http://test.com;
}

http://test.com/proxy/index.html  ->  http://test.com/proxy/index.html
```

```nginx
location /proxy/ {
    proxy_pass http://test.com/test/;
}

http://test.com/proxy/index.html  ->  http://test.com/test/index.html
```

```nginx
location /proxy/ {
    proxy_pass http://test.com/test;
}

http://test.com/proxy/index.html  ->  http://test.com/testindex.html
```



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
