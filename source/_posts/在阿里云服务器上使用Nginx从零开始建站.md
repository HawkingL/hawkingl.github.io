---
title: 在阿里云服务器上使用Nginx从零开始建站
tags: 网站部署
categories: 服务器
keywords: Nginx
descriptions: >-
  Nginx (engine x)
  是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，公开版本1.19.6发布于2020年12月15日。其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、简单的配置文件和低系统资源的消耗而闻名。2022年01月25日，nginx
  1.21.6发布。Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like
  协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-e2d44e7a08227e26d892f827076b89df_r.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-25_16-14-40.jpg"
abbrlink: b830f3e
date: 2022-02-23 12:50:08
---

# 在阿里云服务器上使用 Nginx 从零开始建站

## 阿里云 ECS 服务器安装 Nginx

### 什么是 Nginx

Nginx (engine x) 是一个高性能的 HTTP 和反向代理 web 服务器，同时也提供了 IMAP/POP3/SMTP 服务。Nginx 是由伊戈尔·赛索耶夫为俄罗斯访问量第二的 Rambler.ru 站点（俄文：Рамблер）开发的，公开版本 1.19.6 发布于 2020 年 12 月 15 日。
其将源代码以类 BSD 许可证的形式发布，因它的稳定性、丰富的功能集、简单的配置文件和低系统资源的消耗而闻名。2022 年 01 月 25 日，nginx 1.21.6 发布。
Nginx 是一款轻量级的 Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在 BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上 nginx 的并发能力在同类型的网页服务器中表现较好。

### 安装 Nginx

```text
yum install nginx  //进入到根目录下的etc下创建nginx目录在目录中下载
```

### Nginx 的启动停止重启命令

```
[root@example ~]# service nginx start

[root@example ~]# service nginx stop

[root@example ~]# service nginx reload
```

### 查看 nginx 运行状态

```
ps -ef | grep nginx
```

## Nginx 的基础配置详解

```shell
#  用户以什么身份进入
user root;
#   工作的子进程个数
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

#  事件驱动模块
events {
    worker_connections 1024;
}


http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

#   数据零拷贝：nginx将要发送的文件信息发送到网络接口，使网络接口直接调用文件进行发送。当数据零拷贝关闭时，文件会被复制到nginx应用程序内存中再从程序内存中复制给网络接口进行发送
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
#   保持链接超时的时间
    keepalive_timeout   65;
    types_hash_max_size 4096;

#   将外部的配置文件引入到当前文件中（mime.type：文件类型与加载方式的映射表，设置在请求头中，用来标注该文件是什么类型的文件，使浏览器正确解析该文件）
    include             /etc/nginx/mime.types;
#   文件默认的类型：当文件类型在 mime,type 中找不到对应的类型时，使用默认的加载方式 （application/octet-stream：告诉浏览器，以文件流的形式加载）
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

# 一个 server 相当于一个虚拟主机
    server {
#       当前主机监听的端口号（不能重复）
        listen       80;
        listen       [::]:80;
#       当前主机的主机名（可以配置为域名或主机名）
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

#       匹配对应的uri路径
        location / {
#           当匹配到 / 时，从那个目录寻找对应的网页
            root /root/print;
#           默认展示的页面
            index index.html;
            try_files $uri $uri/ /index.html;
        }

#       尚品汇的反向代理路径
#       location /api {
#           proxy_pass http://gmall-h5-api.atguigu.cn;
#       }

        error_page 404 /404.html;
        location = /404.html {
        }

#       错误页面配置：当服务器内部发生500 502 503 504 错误时，跳转到 http://47.114.74.114/50x.html 页面
        error_page 500 502 503 504 /50x.html;
#       当用户访问 http://47.114.74.114/50x.html 时跳转到用户自定义的页面
        location = /50x.html {
#       自定义错误页面（root/html目录下寻找）
#            root /root/html
        }
    }

}
```

注意：

1. listen 和 server_name 相加必须是唯一的

2. 一个 server_name 可以配置多个主机

   ```shell
   server_name xxx.hawking-img.oss-cn-hangzhou.aliyuncs.com xxx.hawlingl.cn *.hawking-img.oss-cn-hangzhou.aliyuncs.com;
   ```

3. 修改完 Nginx 配置文件后记得重启 Nginx

## 进行域名解析

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-25_16-14-40.jpg)

- TCP/IP：使文件以数据流的形式进行传输
- HTTP：对传输的文件流进行协商传输
- DNS：域名解析服务

### 域名，DNS，IP 地址之间的关系

**在浏览器器上通过域名访问到 DNS 服务器，DNS 服务器返回域名对应的 IP 地址，浏览器再通过 IP 地址访问对应的服务器，其中 DNS 相当于一个映射表，将域名与 IP 进行映射。**

一个服务器可以有多个虚拟主机，一个 ip 地址可以绑定多个域名；实际生产环境中可以将多个域名同时指向一个服务器，服务器通过 Nginx 给不同的域名返回不同的页面。

### 阿里云域名解析

域名解析相当于在 DNS 服务器中保存你的域名和 IP，可以通过二级域名配置多个映射关系，将所有的域名都指向同一个 IP

1. 再域名服务中找到需要解析的域名，点击解析![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-25_17-10-15.jpg)

2. 添加解析记录![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-25_17-16-43.jpg)

3. 常用记录类型的说明![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-25_17-23-47.jpg)

4. 主机记录，即你在浏览器搜索时输入的域名的格式 ![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-25_17-26-22.jpg)

5. 记录值为你指向的 IP 地址或域名

## 正向代理

如果把局域网外的 `Internet `想象成一个巨大的资源库，则局域网中的客户端要访问 `Internet`，则需要通过代理服务器来访问，这种代理服务就称为正向代理，下面是正向代理的原理图。

## 反向代理

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-25_20-16-27.jpg)

其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 `IP `地址。

## 负载均衡

所谓负载均衡就是：就是把大量的请求按照我们指定的方式均衡的分配给集群中的每台服务器，从而不会产生集群中大量请求只请求某一台服务器，从而使该服务器宕机的情况。

## 防盗链

### Referer

Referer 是 HTTP 请求 header 的一部分，当浏览器（或者模拟浏览器行为）向 web 服务器发送请求的时候，头信息里有包含 Referer 。比如我在www.google.com 里有一个www.baidu.com 链接，那么点击这个www.baidu.com ，向它发送的请求头中就有 Referer 信息：

    Referer=http://www.google.com

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-29_11-14-47.jpg)

注意：

- Referer 在浏览器端是无法修改的，是浏览器自动加在请求头中的
- Referer 只有在一个页面中再次访问另一个网址时才会添加，第一次访问时不会添加

### 防盗链

将这个 http 请求发给服务器后，如果服务器要求必须是某个地址或者某几个地址才能访问，而你发送的 referer 不符合他的要求，就会拦截或者跳转到他要求的地址，然后再通过这个地址进行访问。

nginx 中防盗链的基本配置：

```shell
server{
	listen  80;
	server_name localhost;
	location / {
		rewrite /([0-9]+).html$ /index.jsp?pageNum=$1 break;
		proxy_pass http://httpds;
	}

#在location中配置防盗链
	location ~*/(js img css){
#只有来自 192.168.44.101 这个地址的请求才可以被响应，referer后也可以直接配置域名
	  valid referers 192.168.44.101;
#如果是其他地址的请求直接返回 403 错误
	  if ($invalid_referer){
	    return 403;
	  }
		root html;
		index index.html index.htm;
	}

	error_page 500 502 503 504 /50x.html;
	location =/50x.html {
	  root html;
	}

```
