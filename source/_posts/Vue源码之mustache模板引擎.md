---
title: Vue源码之mustache模板引擎
tags: Vue源码
categories: Vue
keywords: Vue源码
description: >-
  mustache是“胡子”的意思，因为它的嵌入标记{{ }}非常像胡子，{{ }}的语法也被vue沿用。mustache.js 是一个简单强大的
  JavaScript 模板引擎,使用它可以简化在 js 代码中的 html
  编写。它比Vue诞生的早多了，它的底层实现机理在当时是非常有创造性的、轰动性的，为后续模板引擎的发展提供了崭新的思路。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-591e6208933e9df2442ef55251ccdd9b_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-26_19-10-35.jpg"
abbrlink: 6c562092
date: 2022-07-15 10:31:54
---

# Vue 源码之 mustache 模板引擎

## mustache 库的工作机理

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-05-26_19-10-35.jpg)

## tokens

tokens 是一个 JS 的嵌套数组，是模板字符串的 JS 表示

模板字符串

```html
<h1>我买了一个{{thing}}，好{{mood}}啊</h1>
```

tokens（每个数组的第零项都是第一项的数据类型）

```js
[
  ["text", "<h1> 我买了一个"], //token
  ["name", "thing"], //token
  ["text", ",好"], //token
  ["name", "mood"], //token
  ["text", "啊</h1>"], //token
];
```

当模板字符串中有循环存在时，他将被编译为嵌套更深的 tokens

```html
<div>
  <ul>
    {{#arr}}
    <li>{{.}}</li>
    {{/arr}}
  </ul>
</div>
```

tonkes

```js
[
  ["text", "<div><ul>"],
  [
    "#",
    "arr",
    [
      ["text", "<li>"],
      ["name", "."],
      ["text", "</li>"],
    ],
  ],
  ["text", "</ul></div>"],
];
```

因此 mustache 库只需要做两件事

1. 将模板字符串编译为 tokens 形式
2. 将 tokens 结合数据，解析为 dom 字符串

## 手写一个简单的 mustache 模板引擎

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/mustache%E6%A8%A1%E6%9D%BF%E5%BC%95%E6%93%8E%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.jpg)

### 初始化文件夹

```
npm init
```

### 安装依赖包

```
npm i webpack@4 webpack-cli@3 webpack-dev-server@3
```

### 配置 webpack

```js
const path = require("path");

module.exports = {
  //模式为开发模式
  mode: "development",
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  //配置webpack_dev_server
  devServer: {
    //静态文件根目录
    contentBase: path.join(__dirname, "www"),
    //端口号
    port: 8081,
  },
};
```

### 配置入口文件

#### www/index.html

引入一个 html 模板字符串，使用定义的模板引擎将模板字符串转换为 html 字符串

```js
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>mustache</title>
</head>
<body>
  <div id="container">Hello world</div>
  <script src="/xuni/bundle.js"></script>
  <script>
    //模板字符串
    var templateStr = `
      <ul>
        {{#students}}
          <li>
            <div class="hd"> {{name}} 的基本信息</div>
            <div class="bd">
              <p>姓名：{{name}}</p>
              <p>年龄：{{age}} </p>
              <p>性别：{{sex}} </p>
              <h2>爱好</h2>
              {{#hobbies}}
                <h3>{{.}}</h3>
              {{/hobbies}}
            </div>
          </li>
        {{/students}}
      </ul>
    `
    //模板数据
    var data = {
      students: [
        {"name": "小明", "age": 12, "sex": "男", "hobbies": ["游泳", "跑步"]},
        {"name": "小红", "age": 11, "sex": "女", "hobbies": ["游泳", "跑步"]},
        {"name": "小强", "age": 13, "sex": "男", "hobbies": ["游泳", "跑步"]}
      ]
    };

		//mustache模板引擎
    var domStr = myMustache.render(templateStr, data);

		//获取元素节点
    var container = document.getElementById('container');
    //将模板字符串插入该节点中
    container.innerHTML = domStr;
  </script>
</body>
</html>
```

#### src/index.js

入口 JS 文件：在 window 属性上绑定一个 myMustache 属性，在 myMustache 属性上定义一个 render(templateStr, data)方法，该方法需要传入模板字符串和对应的数据，其中 parseTemplateToTokens()方法可以将模板字符串转换为一个 tokens 数组，renderTemplate() 方法可以将 tokens 数组中的模板那变量与对应的数据进行替换，再将替换完之后的 tokens 数组重新拼接成模板字符串返回。

```js
import parseTemplateToTokens from "./parseTemplateToTokens";
import renderTemplate from "./renderTemplate";

// @ts-ignore
window.myMustache = {
  render(templateStr, data) {
    var tokens = parseTemplateToTokens(templateStr);
    templateStr = renderTemplate(tokens, data);
    return templateStr;
  },
};
```

### class Scanner

Scanner 类：可以构造一个扫描类的实例对象，其中 scna(tag) 函数可以将字符串中为 tag 的字符跳过，输出 tag 之后的字符串；scanUtil(stopTag)函数可以将上一个 stopTag 和当前 stopTag 之间的字符串进行输出。

```js
/* Scanner类：识别'{{'和’}}‘ 两边和之间的内容并将其包裹成相应的数组 */

export default class Scanner {
  constructor(templateStr) {
    this.templateStr = templateStr;
    this.pos = 0;
    this.tail = templateStr;
  }

  scan(tag) {
    if (this.tail.indexOf(tag) == 0) {
      this.pos += tag.length;
      return this.templateStr.substr(this.pos);
    }
  }

  scanUtil(stopTag) {
    var pos_back = this.pos;
    while (
      this.pos < this.templateStr.length &&
      this.tail.indexOf(stopTag) != 0
    ) {
      this.pos++;
      this.tail = this.templateStr.substr(this.pos);
    }
    return this.templateStr.substring(pos_back, this.pos);
  }
}
```

### parseTemplateToTokens(templateStr)

在 parseTemplateToTokens()函数中实例化一个 Scanner 对象，配合 scan()和 scanUtil()可以将模板字符串转换为一个 tokens 数组，最后再调用 nestTokens(tokens)函数可以在 tokens 数组中嵌套一个 tokens 数组，实现数据的循环嵌套。

```js
/* 将传入的字符串变为tokens数组 */

import nestTokens from "./nestTokens";
import Scanner from "./Scanner";

export default function parseTemplateToTokens(templateStr) {
  var tokens = [];
  var scanner = new Scanner(templateStr);
  while (scanner.pos < templateStr.length) {
    var words = scanner.scanUtil("{{");
    if (words) {
      tokens.push(["text", words]);
    }
    scanner.scan("{{");
    var words = scanner.scanUtil("}}");
    if (words != "") {
      if (words[0] == "#") {
        tokens.push(["#", words.substring(1)]);
      } else if (words[0] == "/") {
        tokens.push(["/", words.substring(1)]);
      } else {
        tokens.push(["name", words]);
      }
    }
    scanner.scan("}}");
  }
  return nestTokens(tokens);
}
```

### nestTokens(tokens)

将传过来的二维数组折叠成更高维的数组，实现数组的循环嵌套

```js
/*将 # 与 / 之间的数组进行折叠 */
export default function nestTokens(tokens) {
  var nestTokens = [];
  var sections = [];
  var collector = nestTokens;
  for (let i = 0; i < tokens.length; i++) {
    var token = tokens[i];
    switch (token[0]) {
      case "#":
        collector.push(token);
        sections.push(token);
        collector = token[2] = [];
        break;
      case "/":
        sections.pop();
        collector =
          sections.length > 0 ? sections[sections.length - 1][2] : nestTokens;
        break;

      default:
        collector.push(token);
        break;
    }
  }
  return nestTokens;
}
```

nestTokens(tokens)函数输出的 nestTokens 数组

```js
[
    ["text","<ul>"],
    ["#","students",[
        ["text","<li><div class=\"hd\"> "],
        ["name","name"],
        [ "text"," 的基本信息</div><div class=\"bd\"><p>姓名："],
        ["name","name" ],
        ["text","</p><p>年龄："],
        ["name","age"],
        ["text"," </p><p>性别："],
        ["name", "sex"],
        ["text"," </p><h2>爱好</h2>"],
        ["#","hobbies",[
             ["text","<h3>"],
             ["name","."],
             ["text","</h3>"]
         ]
     ],
     ["text","</div></li>"]
]
```

### renderTemplate(tokens, data)

将折叠之后的 tokens 数组中的模板变量用 data 中的真实数据进行替换后，重新拼接为 html 字符串，配置 praseArray(tokens, data)函数使用递归算法，将 tokens 数组中嵌套的 tokens 数组也进行替换拼接处理。

```js
import parseArray from "./parseArray";

/* 将tokens数组转换为模板字符串 */
export default function renderTemplate(tokens, data) {
  var resultStr = "";
  for (let i = 0; i < tokens.length; i++) {
    if (tokens[i][0] == "text") {
      resultStr += tokens[i][1];
    } else if (tokens[i][0] == "name") {
      var v = tokens[i][1] == "." ? data : data[tokens[i][1]];
      resultStr += v;
    } else {
      resultStr += parseArray(tokens[i], data);
    }
  }
  return resultStr;
}
```

### praseArray(tokens, data)

配合 renderTemplate()函数使用递归算法输出拼接完成的模板字符串

```js
import renderTemplate from "./renderTemplate";

/* 处理tokens中的循环遍历 */
export default function parseArray(tokens, data) {
  var templateStr = "";
  /* 遍历数据 */
  for (let i = 0; i < data[tokens[1]].length; i++) {
    templateStr += renderTemplate(tokens[2], data[tokens[1]][i]);
  }

  return templateStr;
}
```

输出 resultStr 字符串

```html
<ul>
  <li>
    <div class="hd">小明 的基本信息</div>
    <div class="bd">
      <p>姓名：小明</p>
      <p>年龄：12</p>
      <p>性别：男</p>
      <h2>爱好</h2>
      <h3>游泳</h3>
      <h3>跑步</h3>
    </div>
  </li>
  <li>
    <div class="hd">小红 的基本信息</div>
    <div class="bd">
      <p>姓名：小红</p>
      <p>年龄：11</p>
      <p>性别：女</p>
      <h2>爱好</h2>
      <h3>游泳</h3>
      <h3>跑步</h3>
    </div>
  </li>
  <li>
    <div class="hd">小强 的基本信息</div>
    <div class="bd">
      <p>姓名：小强</p>
      <p>年龄：13</p>
      <p>性别：男</p>
      <h2>爱好</h2>
      <h3>游泳</h3>
      <h3>跑步</h3>
    </div>
  </li>
</ul>
```
