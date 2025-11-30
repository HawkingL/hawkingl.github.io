---
title: 手写this进行显示绑定的函数
tags: 内置函数的实现原理
categories: JS
keywords: 显示绑定
descriptions: 自己编写显示绑定的函数
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-343984765203576c7f973dd3b9309c24_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-b3a79ca1f0f6f7d859b43b97777f09a2_1440w.jpg"
abbrlink: 83f9c9ed
date: 2023-03-23 23:48:21
---

# 自己编写显示绑定的函数

## call()函数的实现

```js
// @ts-nocheck
/* 自己编写一个myCall函数实现call()函数的功能 */
Function.prototype.myApply = function (obj, args) {
  //this为调用myCall()函数的函数对象
  var fn = this;
  //判断传入的obj是不是一个null或undefined，如果是null或undefined则将obj指向window
  obj = obj ? obj : window;
  //将要传入的obj转换成一个对象
  var thisObj = new Object(obj);
  //将函数对象绑定到this要指向的对象
  thisObj.fn = fn;
  //判断传入的值有没有参数
  var result;
  if (args) {
    //通过传入的对象调用fn()函数
    result = thisObj.fn(...args);
  } else {
    result = thisObj.fn();
  }
  //删除thisObj身上的fn属性
  delete thisObj.fn;
  //将调用fn()函数的返回值进行返回
  return result;
};
var obj = {
  name: "lff",
};

function foo(state) {
  console.log(this.name + state);
}
//调用自定义的myCall函数将foo()函数的this绑定到obj对象上
foo.myApply(obj, ["敲代码"]);
```

## apply()函数的实现

```js
// @ts-nocheck
/* 自己编写一个myCall函数实现call()函数的功能 */
Function.prototype.myApply = function (obj, args) {
  //this为调用myCall()函数的函数对象
  var fn = this;
  //判断传入的obj是不是一个null或undefined，如果是null或undefined则将obj指向window
  obj = obj ? obj : window;
  //将要传入的obj转换成一个对象
  var thisObj = new Object(obj);
  //将函数对象绑定到this要指向的对象
  thisObj.fn = fn;
  //判断传入的值有没有参数
  var result;
  if (args) {
    //通过传入的对象调用fn()函数
    result = thisObj.fn(...args);
  } else {
    result = thisObj.fn();
  }
  //删除thisObj身上的fn属性
  delete thisObj.fn;
  //将调用fn()函数的返回值进行返回
  return result;
};
var obj = {
  name: "lff",
};

function foo(state) {
  console.log(this.name + state);
}
//调用自定义的myCall函数将foo()函数的this绑定到obj对象上
foo.myApply(obj, ["敲代码"]);
```

## bind()函数的实现

```js
// @ts-nocheck
Function.prototype.myBind = function (obj, ...bindArgs) {
  //获取调用myBind的对象
  var fn = this
  //将要绑定的值转换为一个对象
  var thisObj = new Object(obj)
  //返回一个函数
  return function (...funArgs) {
    //将绑定时传入的参数和调用时传入的参数整合成一个参数
    var proxyArgs = [...bindArgs, ...funArgs]
    //将需要改变this指向的函数绑定到thisObj对象上
    thisObj.fn = fn;
    var result
    //使用thisObj调用函数
    if (proxyArgs) {
      result = thisObj.fn(...proxyArgs);
    } else {
      result = thisObj.fn()
    }
    //删除绑定在传入对象上的函数
    delete thisObj.fn;
    return result
  }
}

function foo(read, watch) {
  console.log(this.name, read, watch);
}
var obj = {
  name: 'lff'
};
var zab = foo.myBind(obj, "杀死一只知更鸟")
zab("怪奇物语"）
```
