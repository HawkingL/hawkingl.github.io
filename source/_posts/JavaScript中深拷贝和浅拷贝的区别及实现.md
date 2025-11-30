---
title: JavaScript中深拷贝和浅拷贝的区别及实现
tags: 深拷贝和浅拷贝
categories: JavaScript
keywords: 深拷贝和浅拷贝
descriptions: 浅拷贝：只复制指向某个对象的指针，而不复制对象本身，新旧内存共享一块内存。深拷贝：复制并创建一个一模一样的对象，不共享内存，修改新对象，旧对象保持不变。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-7cb0dbd256c6f738c73e60c9620a5470_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-adeeb3d5b0d44bbf5bcf2dbef53eaff5_1440w.jpg"
abbrlink: 2d3c4965
date: 2021-03-11 13:13:03
---

# JavaScript 中深拷贝和浅拷贝的区别及实现

## 区别

- 浅拷贝：只复制指向某个对象的指针，而不复制对象本身，新旧内存共享一块内存。
- 深拷贝：复制并创建一个一模一样的对象，不共享内存，修改新对象，旧对象保持不变。

## 浅拷贝的实现

`Object.assign(目标文件， 原文件)` 浅拷贝的方法

- 当对象只有一级属性为深拷贝；
- 当对象中有多级属性时，二级属性后就是浅拷贝；

```js
<script>
    // 浅拷贝只是拷贝一层, 更深层次对象级别的只拷贝引用(地址)
    var obj = {
        id: 1,
        name: 'admin',
        msg: {
          age: 18
        }
    };
    var o = {};
    console.log('--------------');
    // Object.assign(目标文件， 原文件)  浅拷贝的方法
    Object.assign(o, obj);
    console.log(o);
    o.msg.age = 20;
    console.log(obj);
    // 结果就是 o 和 obj 的 msg.age 都变成了20, 因为 o 和 obj 是共享一块内存的
</script>
```

## 深拷贝的实现

### 递归遍历所有层级实现深拷贝

```js
// 深拷贝拷贝多层, 每一级别的数据都会拷贝.
var obj = {
  id: 1,
  name: "andy",
  msg: {
    age: 18,
  },
  color: ["pink", "red"],
};

var o = {};
function deepCopy(newobj, oldobj) {
  for (var k in oldobj) {
    // 判断我们的属性值属于那种数据类型
    // 1. 获取属性值  oldobj[k]
    var item = oldobj[k];
    // 2. 判断这个值是否是数组(首先判断数组是因为数组也属于对象)
    if (item instanceof Array) {
      newobj[k] = [];
      deepCopy(newobj[k], item);
    } else if (item instanceof Object) {
      // 3. 判断这个值是否是对象
      newobj[k] = {};
      deepCopy(newobj[k], item);
    } else {
      // 4. 属于简单数据类型
      newobj[k] = item;
    }
  }
}
deepCopy(o, obj);
o.msg.age = 20;
console.log(o);
console.log(obj); // obj.msg.age 还是等于 18 没有改变
var arr = [];
console.log(arr instanceof Object); // true 数组也是对象
```

### 利用 JSON 对象实现深拷贝

```js
// 深拷贝拷贝多层, 每一级别的数据都会拷贝.
var obj = {
  id: 1,
  name: "andy",
  msg: {
    age: 18,
  },
  color: ["pink", "red"],
};

// 2. 利用 JSON 对象
function deepCopy(obj) {
  let _obj = JSON.stringify(obj);
  let newObj = JSON.parse(_obj);
  return newObj;
}

let newObj = deepCopy(obj);
newObj.msg.age = 22;
console.log(newObj); //  newObj.msg.age = 22
console.log(obj); // obj.msg.age 还是等于 18 没有改变
```

### lodash 函数库实现深拷贝

```js
//value : 要深拷贝的值。
//返回拷贝后的值

//vue 中使用 ：
//  	 a. npm i --save lodash     下载依赖
//  	 b. import {cloneDeep} from 'lodash'  在 组件 中引入

var objects = [{ a: 1 }, { b: 2 }];
var deep = cloneDeep(objects);
```
