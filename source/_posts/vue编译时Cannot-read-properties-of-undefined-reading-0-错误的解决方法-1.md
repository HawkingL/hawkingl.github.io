---
title: vue编译时Cannot read properties of undefined (reading '0')错误的解决方法
tags: vue常见错误
categories: 项目日记
keywords: Cannot read properties of undefined
description: 由于页面在挂载时请求的数据没有及时的返回，使其解析时读取到的数据为空，如果连续读取空数据的属性值时，这时浏览器就会报错。
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/20210915231539_e0f2a.jpeg
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/20220308203404.png
abbrlink: 450b38d8
date: 2022-03-08 20:27:58
---

错误原因:sob:：由于页面在挂载时请求的数据没有及时的返回，使其解析时读取到的数据为空，如果连续读取空数据的属性值时，这时浏览器就会报错:broken_heart:，读取空数据的属性值时会返回 undefined，这时如果再次读取 undefined 的属性值时就会报错:rocket:。
![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/20220308203404.png)

解决方法：数据仓库返回数据时对数据进行加工，让其至少返回一个空数组，或对象:thumbsup:

```
const getters = {
  categoryView(state) {
    return state.goodInfo.categoryView || {};
  },
}
```

不过值得注意的是，这种错误报的错误是一种假的错误，并不会影响代码的编译:joy:。
