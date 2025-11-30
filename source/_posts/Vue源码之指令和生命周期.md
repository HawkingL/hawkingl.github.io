---
title: Vue源码之指令和生命周期
tags: Vue源码
categories: Vue
keywords: Vue源码
description: 该篇主要是对前面所有技术的总结，每一个算法在整个Vue框架中的哪一个环节运行，在什么时候执行，只针对Vue框架中的核心功能进行演示，不对细节进行处理。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-7cb0dbd256c6f738c73e60c9620a5470_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-8fe77e92614d1a2e35d8d355eeeee45c_1440w.jpg"
abbrlink: f966f55d
date: 2022-06-28 10:26:42
---

# Vue 源码之指令和生命周期

## 前言

该篇主要是对前面所有技术的总结，每一个算法在整个 Vue 框架中的哪一个环节运行，在什么时候执行，只针对 Vue 框架中的核心功能进行演示，不对细节进行处理。

## 测试环境的搭建

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-06-21_15-45-51.jpg)

### 安装依赖

npm i webpack@5 -D

npm i webpack-cli@4 -D

npm i webpack-dev-server@4 -D

### webpack.config.js

配置整个程序的入口和出口文件

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

## 源码解析

index.js

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
    <h1>指令和生命周期</h1>
    <div id="app">
      你好{{a}}
      <h3 class="box" v-if="a > 0">你好{{a}}</h3>
      <input type="text" v-model="a" />
      <ul>
        <li>A</li>
        <li>B</li>
        <li>C</li>
      </ul>
    </div>
    <script src="./xuni/bundle.js"></script>
    <script>
      var vm = new Vue({
        el: "#app",
        data: {
          a: 10,
        },
        watch: {
          a() {
            console.log("a改变了");
          },
        },
      });
    </script>
  </body>
</html>
```

index.js

```js
import Vue from "./Vue";

// @ts-ignore
window.Vue = Vue;
```

Vue.js

```js
import Compile from "./Compile";
import { observe } from "./observe";
import Watcher from "./Watcher";

export default class Vue {
  constructor(options) {
    this.$options = options || {};
    //数据
    this._data = options.data || undefined;
    //将传入的data中的数据变为响应式的
    observe(this._data);
    //将传入的data中的数据绑定在Vue实例上
    this._initData();
    //默认调用watch
    this._initWatch();
    //将数据变为响应式的
    //模板编译
    new Compile(options.el, this);
  }
  _initData() {
    var self = this;
    //将传入的data中的数据绑定在Vue实例上
    Object.keys(this._data).forEach((key) => {
      Object.defineProperty(self, key, {
        get() {
          return self._data[key];
        },
        set(newVal) {
          self._data[key] = newVal;
        },
      });
    });
  }
  _initWatch() {
    var self = this;
    var watch = this.$options.watch;
    Object.keys(watch).forEach((key) => {
      new Watcher(self, key, watch[key]);
    });
  }
}
```

Complie.js

```js
import Watcher from "./Watcher";

export default class Compile {
  constructor(el, vue) {
    //vue实例
    this.$vue = vue;
    //挂载点
    this.$el = document.querySelector(el);
    //如果用户传入了挂载点时
    if (this.$el) {
      //调用函数，让节点变为fragment,实际上就是将真实节点变为mustache中的tokens数组，使用的是AST抽象语法树
      let $fragment = this.node2Fragment(this.$el);
      //编译模板
      this.compile($fragment);
      //将编译好的模板上树
      this.$el.appendChild($fragment);
    }
  }
  node2Fragment(el) {
    var fragment = document.createDocumentFragment();
    var child;
    //注意：使用appendChild添加子元素时，如果该子元素是从真实DOM节点中获取到的则会删除他原来在真实DOM节点中的位置
    while ((child = el.firstChild)) {
      fragment.appendChild(child);
    }
    return fragment;
  }
  compile(el) {
    //获取节点子元素
    var childNodes = el.childNodes;
    var self = this;
    var reg = /\{\{(.*)\}\}/;

    childNodes.forEach((node) => {
      var text = node.textContent;

      if (node.nodeType == 1) {
        self.compileElement(node);
      } else if (node.nodeType == 3 && reg.test(text)) {
        let name = text.match(reg)[1];
        self.compileText(node, name);
      }
    });
  }
  compileElement(node) {
    var nodeAttrs = node.attributes;
    Array.prototype.slice.call(nodeAttrs).forEach((attr) => {
      //这里开始分析指令
      var attrName = attr.name;
      var value = attr.value;
      //指令开头都是v-开头的
      var dir = attrName.substring(2);
      var self = this;

      //判断是否为指令
      if (attrName.indexOf("v-") == 0) {
        //v-开头的就是指令
        if (dir == "model") {
          //双向数据绑定
          new Watcher(self.$vue, value, (value) => {
            node.value = value;
          });
          var v = self.getVueVal(self.$vue, value);
          node.value = v;

          node.addEventListener("input", (e) => {
            var newVal = e.target.value;

            self.setVueVal(self.$vue, value, newVal);
            v = newVal;
          });
        } else if (dir == "if") {
          console.log("发现了if指令", value);
        }
      }
    });
  }
  compileText(node, name) {
    node.textContent = this.getVueVal(this.$vue, name);
    new Watcher(this.$vue, name, (value) => {
      node.textContent = value;
    });
  }

  getVueVal(vue, exp) {
    var val = vue;
    exp = exp.split(".");
    exp.forEach((k) => {
      val = val[k];
    });
    return val;
  }

  setVueVal(vue, exp, value) {
    var val = vue;
    exp = exp.split(".");
    exp.forEach((k, i) => {
      if (i < exp.length - 1) {
        val = val[k];
      } else {
        val[k] = value;
      }
    });
  }
}
```
