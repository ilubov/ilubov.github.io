---
title: Nginx反向代理Google
date: 2020-12-06 18:22:18
tags:
    - nginx
    - google
categories:
    - nginx
---

### Nginx反向代理Google

#### Nginx 配置

```
upstream www.google.com {
    server 173.194.38.1:443;
    server 173.194.38.2:443;
    server 173.194.38.3:443;
    server 173.194.38.4:443;
}

server {
    listen 80;
    server_name ilubov.cn www.ilubov.cn;
    rewrite ^(.*)$ https://$host$1 permanent;
}

server {
    listen 443 ssl;
    server_name ilubov.cn;
    ssl_certificate /etc/nginx/conf/www_ilubov.cn.pem;
    ssl_certificate_key /etc/nginx/conf/www_ilubov.cn.key;
    ssl_protocols SSLv3 TLSv1;
    ssl_ciphers ALL:-ADH:+HIGH:+MEDIUM:-LOW:-SSLv2:-EXP;

    location / {
        proxy_redirect off;
        proxy_set_header Accept-Language "zh-CN";
        proxy_set_header Accept-Encoding "";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_pass https://www.google.com.hk;
    }
}
```