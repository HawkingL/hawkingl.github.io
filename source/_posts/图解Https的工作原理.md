---
title: 图解Https的工作原理
tags: Https
categories: Https
keywords: Https
descriptions: >-
  HTTPS （全称：Hyper Text Transfer Protocol over SecureSocket Layer），是以安全为目标的 HTTP
  通道，在HTTP的基础上通过传输加密和身份认证保证了传输过程的安全性  。HTTPS 在HTTP 的基础下加入SSL，HTTPS 的安全基础是
  SSL，因此加密的详细内容就需要 SSL。 HTTPS 存在不同于 HTTP 的默认端口及一个加密/身份验证层（在 HTTP与 TCP
  之间）。这个系统提供了身份验证与加密通讯方法
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-d3dacbae50f0d48956913080ee09b611_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-29_13-26-11.jpg"
abbrlink: 7fb369f0
date: 2022-02-25 12:36:42
---

# 图解 Https 的的工作原理

## Https

HTTPS （全称：Hyper Text Transfer Protocol over SecureSocket Layer），是以安全为目标的 HTTP 通道，在 HTTP 的基础上通过传输加密和[身份认证](https://baike.baidu.com/item/身份认证/5294713)保证了传输过程的安全性 [1] 。HTTPS 在 HTTP 的基础下加入[SSL](https://baike.baidu.com/item/SSL/320778)，HTTPS 的安全基础是 SSL，因此加密的详细内容就需要 SSL。 HTTPS 存在不同于 HTTP 的默认端口及一个加密/身份验证层（在 HTTP 与 [TCP](https://baike.baidu.com/item/TCP/33012) 之间）。这个系统提供了身份验证与加密通讯方法

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-29_13-26-11.jpg)

## 非对称加密

### 公钥和密钥

公钥：密钥对的持有者公布给其他人的，用来加密和解密数据的

私钥：密钥对的持有者私有的不可以公布的，用来加密和解密数据的

### 什么是非对称加密

非对称加密（asymmetric cryptography），也称为公开密钥加密（Public-key cryptography），是密码学的一种算法，它需要两个密钥，一个是公开密钥，另一个是私有密钥。顾名思义，公钥可以任意对外发布；而私钥必须由用户自行严格秘密保管，绝不透过任何途径向任何人提供，也不会透露给要通信的另一方，即使他被信任。

### 特性

1. 加密的双向性：加密具有双向性，即公钥和私钥中的任一个均可用作加密，此时另一个则用作解密。
2. 公钥无法推导出私钥

## CA 机构

### 简介

证书颁发机构（CA, Certificate Authority）即颁发数字证书的机构。是负责发放和管理数字证书的权威机构，并作为电子商务交易中受信任的第三方，承担公钥体系中公钥的合法性检验的责任。
CA 中心为每个使用公开密钥的用户发放一个数字证书，数字证书的作用是证明证书中列出的用户合法拥有证书中列出的公开密钥。CA 机构的数字签名使得攻击者不能伪造和篡改证书。在 SET 交易中，CA 不仅对持卡人、商户发放证书，还要对获款的银行、网关发放证书。
CA 是证书的签发机构,它是 PKI 的核心。CA 是负责签发证书、认证证书、管理已颁发证书的机关。它要制定政策和具体步骤来验证、识别用户身份，并对用户证书进行签名，以确保证书持有者的身份和公钥的拥有权。
CA 也拥有一个证书（内含公钥）和私钥。网上的公众用户通过验证 CA 的签字从而信任 CA ，任何人都可以得到 CA 的证书（含公钥），用以验证它所签发的证书。
如果用户想得到一份属于自己的证书，他应先向 CA 提出申请。在 CA 判明申请者的身份后，便为他分配一个公钥，并且 CA 将该公钥与申请者的身份信息绑在一起，并为之签字后，便形成证书发给申请者。
如果一个用户想鉴别另一个证书的真伪，他就用 CA 的公钥对那个证书上的签字进行验证，一旦验证通过，该证书就被认为是有效的。
为保证用户之间在网上传递信息的安全性、真实性、可靠性、完整性和不可抵赖性，不仅需要对用户的身份真实性进行验证，也需要有一个具有权威性、公正性、唯一性的机构，负责向电子商务的各个主体颁发并管理符合国内、国际安全电子交易协议标准的电子商务安全证，并负责管理所有参与网上交易的个体所需的数字证书，因此是安全电子交易的核心环节。

### 客户端中的证书（内嵌在操作系统中，无法修改，在安装操作系统时就已经存在）

**打开 cmd -----> 搜索 certmgr.msc**

## 具体流程

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Https%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%9B%BE_%E9%AB%98%E6%B8%85.png)
