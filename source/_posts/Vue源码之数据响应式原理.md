---
title: Vue源码之数据响应式原理
tags: Vue源码
categories: Vue
keywords: Vue源码
descriptions: >-
  使用一般语法访问或添加对象的属性时，并不能被我们检测到，也就无法在修改数据时使页面中所有引用的数据都发生改变，所以我们必须在对象的每个属性身上绑定getter和setter方法，使我们每次访问和修改对象的任意一个属性时，都会被检测的到。当监听到对象的某个属性被修改时，就可以执行相应的回调函数，生成虚拟DOM进行diff比较，再将修改之后的数据应用到这个页面。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-343984765203576c7f973dd3b9309c24_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-67030349108004a92a7b07ce246886a3_1440w.jpg"
abbrlink: 6a32cfff
date: 2022-06-20 11:20:50
---

# Vue 源码之数据响应式原理

## 前言

使用一般语法访问或添加对象的属性时，并不能被我们检测到，也就无法在修改数据时使页面中所有引用的数据都发生改变，所以我们必须在对象的每个属性身上绑定 getter 和 setter 方法，使我们每次访问和修改对象的任意一个属性时，都会被检测的到。当监听到对象的某个属性被修改时，就可以执行相应的回调函数，生成虚拟 DOM 进行 diff 比较，再将修改之后的数据应用到这个页面。

## 依赖的安装

npm i webpack@5 -D

npm i webpack-cli@4 -D

npm i webpack-dev-server@4 -D

## 环境的配置

### 文件夹的建立

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-06-21_15-45-51.jpg)

### webpack.config.js 的配置

```js
const path = require("path");

module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    publicPath: "/xuni/",
    filename: "bundle.js",
  },
  devServer: {
    port: 8080,
    static: {
      directory: path.join(__dirname, "www"),
    },
  },
};
```

### html 页面的配置

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h1>数据响应式原理</h1>
    <script src="./xuni/bundle.js"></script>
  </body>
</html>
```

## 数据响应式原理

### 人口文件的配置

作为入口函数将要做响应式的对象传入 observe(obj)函数中，实现数据的响应式

```js
import { observe } from "./observe";
import Watcher from "./Watcher";

//需要做响应式的数据
var obj = {
  a: {
    m: {
      n: 5,
    },
  },
  b: 4,
  g: [1, 2, 3, 4],
};

//将传入的对象做响应式处理
observe(obj);

//构造一个监听对象，监听obj对象中的a.m.n属性，一旦读取该值就打印****启动监听***
//回调函数第一个值为修改之前的值，第二个为修改之后的值
new Watcher(obj, "a.m.n", (newValue, oldValue) => {
  console.log("****启动监听***", newValue, oldValue);
});
obj.a.m.n = 88;

//obj.g.push(5)
console.log(obj);
```

### observe.js

observe(value)：函数的作用

- 判断传入的值是否为一个对象，如果不是对象直接退出函数
- 如果传入的值是一个对象则判断在该对象上是否有**ob**属性
- 如果该对象身上有**ob**属性，就将该属性返回
- 如果该对象身上没有**ob**属性，new 一个 Observer 对象给该属性，即重构该属性
- 在 new 的时候会给该属性添加一个**ob**属性，并重写数组的方法，追加属性的 getter 和 setter 方法

```js
import Observer from "./Observer";
export const observe = function (value) {
  //如果value不是对象，直接退出函数
  if (typeof value != "object") return;
  //d定义ob
  var ob;
  //判断传入的value变量是否有__ob__属性
  //如果有直接赋值给ob
  //如果没有则重新构造__ob__属性及相关属性给ob
  if (typeof value.__ob__ != "undefined") {
    ob = value.__ob__;
  } else {
    //重构传入的value属性
    //在new的时候会给该属性添加一个__ob__属性，并重写数组的方法，追加属性的getter和setter方法
    ob = new Observer(value);
  }
  return ob;
};
```

### Observer.js

Observer 类的作用

- ​ 给传入的 value 属性绑定一个**ob**属性，并定义**ob**属性的控制属性
- ​ 判断传入的值是一个数组还是一个对象
- ​ 如果是一个数组，重写数组中的方法，遍历数组使数组中的每一个元素调用一次 observe()函数
- ​ 如果是一个对象，调用 walk 函数遍历对象中的每一项，然后给每一项做响应式处理

-

```js
import arrayMethods from "./array";
import defineReactive from "./defineProperty";
import Dep from "./Dep";
import { observe } from "./observe";
import { def } from "./utils";

export default class Observer {
  constructor(value) {
    //每一个Observer的实例身上都有一个dep
    this.dep = new Dep();
    //给传入的对象绑定__ob__属性并定义__ob__的控制属性，其中__ob__属性的值为实例对象本身
    def(value, "__ob__", this, false);
    //判断该属性是对象还是数组
    //如果是数组，就重写数组内置的方法，并遍历查找数组中的每一个元素是否有子项
    //如果是对象，就为每一个对象的属性做响应式处理
    if (Array.isArray(value)) {
      //修改对象的原型对象，将value数组的原型指向arrayMethods对象，arrayMethods是重写之后的数组方法
      Object.setPrototypeOf(value, arrayMethods);
      this.observeArray(value);
    } else {
      this.walk(value);
    }
  }
  //遍历value对象的每一个属性，对每一个遍历出来的对象做响应式处理
  walk(value) {
    for (let k in value) {
      defineReactive(value, k);
    }
  }
  //遍历数组中的每一项，对数组中的子项进行处理
  observeArray(arr) {
    for (let i = 0; i < arr.length; i++) {
      observe(arr[i]);
    }
  }
}
```

### defineProperty.js

defineReactive()：函数的功能

- ​ 如果给函数传入的参数只有两个，就将传入对象属性的值赋值给 val 变量
- ​ 对 val 进行递归，处理传入对象的子项
- ​ 给传入对象的属性绑定 getter 和 setter 方法

```js
import Dep from "./Dep";
import { observe } from "./observe";

export default function defineReactive(obj, key, val) {
  const dep = new Dep();
  //如果只传入需要实现响应式的对象和属性时将属性的值赋值给val变量
  if (arguments.length == 2) {
    val = obj[key];
  }

  //对属性的值实现递归进行递归
  let childOb = observe(val);

  //给传入对象的属性添加getter和setter
  Object.defineProperty(obj, key, {
    //可枚举
    enumerable: true,
    //可被配置
    configurable: true,
    get() {
      console.log(`访问${key}属性`);
      //如果现在处于依赖的收集阶段
      if (Dep.target) {
        dep.depend();
        if (childOb) {
          childOb.dep.depend();
        }
      }
      return val;
    },
    set(newValue) {
      console.log(`${key}属性被修改了`);
      if (val == newValue) return;
      //将新传入的属性值赋值给val变量
      val = newValue;
      //对新传入的值进行递归操作
      childOb = observe(newValue);
      //发布订阅模式，通知dep
      dep.notify();
    },
  });
}
```

### array.js

重写数组中内置的方法

- ​ 先将数组内置的方法进行备份
- ​ 然后创建一个对象，该对象以数组内置方法的备份作为原型
- ​ 最后重写刚刚创建对象原型上的备份方法

```js
import { def } from "./utils";

//备份数组中内置的方法
const arrayPrototype = Array.prototype;

//以arrayPrototype为原型创建arrayMethods对象，Object.create() 方法用于创建一个新对象，使用现有的对象来作为新创建对象的原型（prototype）。
const arrayMethods = Object.create(arrayPrototype);

//列出需要进行重写的方法的方法名
const methodsNeedChange = [
  //在原数组末尾添加一个元素
  "push",
  //删除数组的最后一个元素
  "pop",
  //删除数组的第一个元素
  "shift",
  //在数组的开头添加一个元素
  "unshift",
  //添加或删除一个或多个元素
  "splice",
  //对数组元素进行排序
  "sort",
  //颠倒数组元素的顺序
  "reverse",
];

//重写数组指定的方法
methodsNeedChange.forEach((methodName) => {
  //备份数组原型上的方法
  const original = arrayPrototype[methodName];
  //重写该方法，
  def(
    arrayMethods,
    methodName,
    function () {
      //apply(original的this要指向的对象, original()函数需要传入的参数,参数为一个数组)
      //这里的this指向调用重写方法的对像，arguments为调用重写方法时传入的参数
      let result = original.apply(this, arguments);
      //将arguments伪数组变为真数组
      const args = [...arguments];
      //获取数组上的__ob__属性
      const ob = this.__ob__;
      //定义一个数组存放要添加的元素
      let inserted = [];
      switch (methodName) {
        case "push":
          inserted = args;
          break;
        case "shift":
          inserted = args;
          break;
        case "unshift":
          inserted = args;
          break;
        case "splice":
          inserted = args.slice(2);
          break;
      }

      //如果有要添加的元素
      if (inserted) {
        ob.observeArray(inserted);
      }

      ob.dep.notify();

      return result;
    },
    false
  );
});

export default arrayMethods;
```

### utils.js

def()函数的功能：给传入的对象绑定一个属性并给属性赋值，定义属性的控制属性

```js
export const def = function (obj, key, value, enumerable) {
  Object.defineProperty(obj, key, {
    value,
    //是否可以被枚举
    enumerable,
    //当且仅当该属性的 writable 键值为 true 时，属性的值，也就是上面的 value，才能被赋值运算符 (en-US)改变。
    writable: true,
    //当且仅当该属性的 configurable 键值为 true 时，该属性的描述符才能够被改变，同时该属性也能从对应的对象上被删除。
    configurable: true,
  });
};
```

### Dep.js

在对象及对象属性的 \_\_ob\_\_ 属性上实例化一个 dep 存放需要添加的依赖

```js
//定义一个id属性，每实例化一个Dep就会自增1
var uid = 0;
export default class Dep {
  constructor() {
    console.log("我是Dep类的构造器");
    //在实例化的对象身上绑定一个id保证dep的唯一性
    this.id = uid++;
    //用数组存储自己的订阅者，即Watcher的实例对象
    this.subs = [];
  }
  //添加订阅
  addSub(sub) {
    this.subs.push(sub);
  }
  //添加依赖
  depend() {
    //Dep.target就是一个我们自己指定的全局的位置（全局唯一）
    if (Dep.target) {
      this.addSub(Dep.target);
    }
  }
  //通知更新
  notify() {
    console.log("我是notify");
    //浅克隆一份
    const subs = this.subs.slice();
    //遍历subs数组
    for (let i = 0; i < subs.length; i++) {
      subs[i].update();
    }
  }
}
//在Dep类身上定义一个静态属性target
Dep.target = undefined;
```

### Watcher.js

实例化一个 Watcher 实例，表明需要监听的对象，监听对象那个属性，属性值被修改之后的回调函数

```js
import Dep from "./Dep";

//定义一个id属性，每实例化一个Watcher就会自增1
var uid = 0;
export default class Watcher {
  //target：要监听的对象，即哪一个组件
  //expression：要监听对象的哪一个属性
  //callback：监听到该对象之后要做到事情
  constructor(target, expression, callback) {
    console.log("我是Watcher类的实例");
    //给当前依赖添加一个id保证当前依赖的唯一性
    this.id = uid++;
    this.target = target;
    //该函数返回一个函数，返回的函数可以查询传入对象属性的值
    this.getter = parsePath(expression);
    //当监听到指定对象的属性时的回调函数
    this.callback = callback;
    this.value = this.get();
  }
  update() {
    this.run();
  }
  get() {
    //进入依赖收集的阶段，让全局的Dep.target设置为Watcher本身，那么就表明进入了依赖收集
    Dep.target = this;

    //将当前监听的对象赋值给obj
    const obj = this.target;
    var value;
    try {
      //找到监听对象的属性的值
      value = this.getter(obj);
    } finally {
      //找到对应属性的值后，退出依赖收集阶段
      Dep.target = null;
    }
    return value;
  }
  run() {
    this.getAndInvoke(this.callback);
  }
  //获取和调用监听对象的属性的修改前后的值
  getAndInvoke(cb) {
    const value = this.get();
    if (value !== this.value || typeof value == "object") {
      const oldValue = this.value;
      this.value = value;
      cb.call(this.target, value, oldValue);
    }
  }
}

//将传入的字符串拆分成数组，并返回一个函数
//返回的函数可以通过拆分的数组遍历出传入对象属性的值
function parsePath(str) {
  var segments = str.split(".");
  return (obj) => {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return;
      obj = obj[segments[i]];
    }
    return obj;
  };
}
```
