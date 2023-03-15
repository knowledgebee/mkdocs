---
title: 使用 Nginx
date: 2022-05-15 20:02:26
tags: 
    - Nginx
    - Encryption
    - HTTPS
---

如何使用开源的Web服务器/反向代理服务器 Nginx

<!--more-->

Nginx 是一个开源高性能的Web服务器/反向代理服务器，轻量高效支持高并发，稳定性也有目共睹。

### 载入配置文件

***

通过软件包方式安装 Nginx 的，配置文件一般在 *`/etc/nginx`* 目录下，默认主配置文件为 *nginx.conf* ,可以使用 include 语句来包含其他配置文件，包含的文件中的内容会替换 include 语句，include 可以使用相对路径也可以使用绝对路径，修改之后检查配置文件语法没有问题便可使 Nginx 重新载入配置文件：

> *nginx -t*

如果没有问题会提示：

~~~text
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
~~~

此时便可以让 Nginx 重新载入配置文件：

> *nginx -s reload*

该命令是 Nginx 的软重启，Nginx 会在处理完会话后重新载入配置文件，并不会中断当前的业务。当然在配置的有问题时，Nginx 会直接报错，常见的有证书权限不够、监听端口冲突:

* 在 Nginx 监听的端口与其他服务监听的端口有冲突时，即使配置文件没有语法格式错误，Nginx 也无法正常启动或者重载入配置文件。

* 如果在语法检测时有问题产生会直接报错，如果最后带有 *`Permission denied`*,则为权限不足，需要使用超级管理员权限，如果在 Nginx 中启用 SSL 来实现 HTTPS 还需要配置域名证书，Nginx 无权打开域名证书文件在语法检测环节不会报错，但在发送重载入配置文件会报权限不足的错误。

当出现这种问题时，Nginx 可能已经崩溃，无法通过软重启来重新载入配置文件，这个时候可以使用 systmctl 工具来让 Nginx 强行重启：

> *systemctl restart nginx*

### 配置文件

***

nginx的配置文件分为三个区：*main* ，*events* ，*http*：

* main：该区域为 Nginx 主要配置如何使用系统资源。

* events：该区域为 Nginx 主要配置如何处理事件和进程。

* http：该区域为主要网站配置区域，Nginx 的监听端口、监听域名、网站根目录以及代理等设置。

* server：该区域在 http 区域下，为监听端口和域名的详细配置。

不同区域的配置语句有相应的有效范围，粒度更细的配置优先级高于大范围的配置，例如在 http 块中的 server 的配置就要优先于 http 中的配置，在 server 中未指定的会使用上一级 http 中的配置，如果都未配置，将会使用默认配置。

### SSL加密

***

为了给 Web 服务器提供更好的安全性，建议启用 SSL 加密，以防止明文传输造成的信息泄露。

SSL加密的配置十分简单：

* 在配置监听时开启 SSL
* 在 ssl_certificate 配置证书路径
* 在 ssl_certificate_key 配置证书私钥路径
* 在 ssl_ciphers 指定 SSL 使用的加密套件 *(非必要)*
* 在 ssl_protocols  指定使用的 TLS 版本 *(非必要)*

在配置证书时要注意证书的权限，Nginx 无法读取证书会导致 Nginx 报错。

### 完整的配置文件

~~~nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
pcre_jit on;
worker_cpu_affinity auto;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
    }

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  example.com;
        location / {
                    return 301 https://$host$request_uri;
            }
        location ^~ /.well-known/acme-challenge {
                    default_type "text/plain";
        root        /usr/share/nginx/html;
            }
        }
    server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  example.com;
        location / {
        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_read_timeout  90;
        proxy_pass http://127.0.0.1:8080;
        proxy_redirect      http://127.0.0.1:8080 https://example.com;
                    }
        ssl_certificate .acme.sh/example.com_ecc/example.com.cer;
        ssl_certificate_key .acme.sh/example.com_ecc/example.com.key;
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
 #       ssl_ciphers ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384;
        ssl_prefer_server_ciphers on;
        ssl_protocols TLSv1.3;
        access_log  /var/log/nginx/host.access.log  main;
        error_page 404 /404.html;
            location = /40x.html {
                }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
                }
            }
    }
~~~
