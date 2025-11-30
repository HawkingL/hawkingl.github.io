---
title: JS中的闭包
tags: 前端
categories: JS
keywords: "JS, 函数"
description: JavaScript中闭包的原理及优点和缺点，如何创建一个闭包，闭包在程序中的应用。
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-780d2703ad47bb32a7fc7c218933d9a0_1440w.jpg
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-3d7a580c5009e9e61e3cab628d497316_1440w.jpg
abbrlink: d0e761e9
date: 2021-11-19 23:18:19
---

## 闭包的概念

{% note success no-icon %} 有权访问另一个函数作用域中的变量的函数。一般情况下就是一个函数中包含另一个函数。由此我们可以知道闭包就是一个函数，只不过这个函数可以访问到另一个函数的作用域。闭包可以访问到父级函数的变量，且该变量不会被销毁{% endnote %}

{% note danger no-icon %}函数的作用域是独立的封闭的，外部的执行环境是无法访问的，但是闭包具有这个能力。{% endnote %}

```js
function person() {
  var name = "LF";
  function cat() {
    console.log(name);
  }
  return cat;
}
var per = person(); //per的值就是return后的结果，即一个函数
per(); //LF per()相当于cat()
per(); //LF  变量在函数执行完之后一直没有被销毁，存在于内存中
per(); //LF
```

闭包的原理其实是利用了作用域链的特性，我们都知道作用域链就是当前执行环境下访问某个变量时，如果不存在就一直向外寻找，最终找到最外层就是全局作用域，这样就形成了一个作用域链

{% note success no-icon %} 闭包的优点：可以隐藏变量避免全局污染。可以读取函数内部的变量。

闭包的缺点：变量不会被回收，造成内存的消耗，不恰当的使用闭包可能会造成内存的泄露 {% endnote %}
