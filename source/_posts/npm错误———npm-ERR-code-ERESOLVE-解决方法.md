---
title: npm错误———npm ERR! code ERESOLVE 解决方法
tags: npm错误
categories: node
keywords: npm ERR! code ERESOLVE 解决方法
description: >-
  其实npm@7与ERESOLVE有关的问题还是比较常见的，这是因为npm7.x对于某些事情要比npm6.x更加严格，通常解决办法就是使用 npm
  install --legacy-peer-deps 或者使用 npm@6。如果这些办法不能立即生效的话，可以先把项目中的 node_modules 和
  package-lock.json 删除，它们将会被重新创建。
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/image-20220217210719220.png
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/image-20220217210639346.png
abbrlink: f0d9c826
date: 2022-02-17 20:50:30
---

```js
PS D:\Vue\Project\SPH\app> npm i vue-router
npm ERR! code ERESOLVE
npm ERR! ERESOLVE unable to resolve dependency tree
npm ERR!
npm ERR! While resolving: app@0.1.0
npm ERR! Found: vue@2.6.14
npm ERR! node_modules/vue
npm ERR!   vue@"^2.6.11" from the root project
npm ERR!
npm ERR! Could not resolve dependency:
npm ERR! peer vue@"^3.0.0" from vue-router@4.0.12
npm ERR! node_modules/vue-router
npm ERR!   vue-router@"*" from the root project
npm ERR!
npm ERR! Fix the upstream dependency conflict, or retry
npm ERR! this command with --force, or --legacy-peer-deps
npm ERR! to accept an incorrect (and potentially broken) dependency resolution.
npm ERR!
npm ERR! See C:\Users\**\AppData\Local\npm-cache\eresolve-report.txt for a full report.

npm ERR! A complete log of this run can be found in:
npm ERR!     C:\Users\**\AppData\Local\npm-cache\_logs\2022-02-17T09_55_46_246Z-debug.log
```

在安装组件的时候出现以上问题，是 npm 版本问题报错

解决方法：在命令后面加上 --legacy-peer-deps

```js
npm install xxx --legacy-peer-deps
```

如图：

![image-20220217210639346](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/image-20220217210639346.png)

![image-20220217210719220](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/image-20220217210719220.png)
