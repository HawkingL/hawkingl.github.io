---
title: 微信小程序中npm的使用
tags: 小程序
categories: 小程序
keywords: 微信小程序中npm的使用
descriptions: >-
  由于小程序只识别miniprogram_npm文件中的包，所有需要将下载好的npm包转换成小程序能够识别的包，找到工具中的构建npm点击，微信开发者工具会自动将下载到node_modules中的包转移到miniprogram_npm中，并将起转换成小程序能够识别的包。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-f3d09f2930406c4c186d8fcf4e4d612d_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-3d7a580c5009e9e61e3cab628d497316_1440w.jpg"
abbrlink: 37dd0ac
date: 2021-08-12 13:05:45
---

# 微信小程序中 npm 的使用

## 启用 npm

打开微信开发者工具中的详情，在本地设置中勾选使用 npm，最新版的微信开发者工具已经自动配置好了环境，无需勾选。

## 初始化 npm

```
npm init
```

对项目进行初始化，使用 npm 管理整个项目

## npm 包的构建

由于小程序只识别 miniprogram_npm 文件中的包，所有需要将下载好的 npm 包转换成小程序能够识别的包，找到工具中的构建 npm 点击，微信开发者工具会自动将下载到 node_modules 中的包转移到 miniprogram_npm 中，并将起转换成小程序能够识别的包。
