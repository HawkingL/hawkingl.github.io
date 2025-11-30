---
title: ssl证书的申请流程
tags: ssl
categories: Https
keywords: ssl
descriptions: >-
  SSL证书是 数字证书 的一种，类似于驾驶证、护照和营业执照的电子副本。 因为配置在服务器上，也称为SSL服务器证书。 SSL 证书就是遵守 SSL协议
  ，由受信任的数字证书颁发机构CA，在验证服务器身份后颁发，具有服务器身份验证和数据传输加密功能。 SSL证书通过在客户端浏览器和 Web服务器
  之间建立一条SSL安全通道（Secure socket layer (SSL)安全协议是由Netscape Communication公司设计开发。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-d3880c197893d98ea5dcfe0c30931242_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-29_22-42-50.jpg"
abbrlink: 1da797b4
date: 2022-07-11 12:43:35
---

## ssl 证书的介绍

SSL 证书是数字证书的一种，类似于驾驶证、护照和营业执照的电子副本。因为配置在服务器上，也称为 SSL 服务器证书。

SSL 证书就是遵守 SSL 协议，由受信任的数字证书颁发机构 CA，在验证服务器身份后颁发，具有服务器身份验证和数据传输加密功能。

SSL 证书通过在客户端浏览器和 Web 服务器之间建立一条 SSL 安全通道（Secure socket layer(SSL)安全协议是由 Netscape Communication 公司设计开发。该安全协议主要用来提供对用户和服务器的认证；对传送的数据进行加密和隐藏；确保数据在传送中不被改变，即数据的完整性，现已成为该领域中全球化的标准。由于 SSL 技术已建立到所有主要的浏览器和 WEB 服务器程序中，因此，仅需安装服务器证书就可以激活该功能了），即通过它可以激活 SSL 协议，实现数据信息在客户端和服务器之间的加密传输，可以防止数据信息的泄露，保证了双方传递信息的安全性，而且用户可以通过服务器证书验证他所访问的网站是否是真实可靠。数字签名又名数字标识、签章 (即 Digital Certificate，Digital ID )，提供了一种在网上进行身份验证的方法，是用来标志和证明网络通信双方身份的数字信息文件，概念类似日常生活中的司机驾照或身份证。 数字签名主要用于发送安全电子邮件、访问安全站点、网上招标与投标、网上签约、网上订购、网上公文安全传送、网上办公、网上缴费、网上缴税以及网上购物等安全的网上电子交易活动。

## ssl 证书的申请流程

打开你的域名控制台，开启 ssl 证书

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-29_22-42-50.jpg)

点击 ssl 证书，申请免费证书

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-29_22-45-54.jpg)

填写完认证的相关信息后等待审核，大概几秒钟就好了，多刷新几遍

审核完后下载证书

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-29_22-47-55.jpg)

将解压后的两个证书文件上传至服务器，可自定义上传路径

上传后进行证书的安装

```shell
server {
    listen 443 ss1;
    server_name aa.abc.com;
#.pem文件所在的目录
    ss1_certificate   /data/cert/server.crt;
#.key所在的目录
    ss1_certificate_key  /data/cert/server.key;
}
```
