---
title: Vue源码之AST抽象语法树
tags: Vue源码
categories: Vue
keywords: Vue源码
description: >-
  本文章我们会亲自手写实现抽象语法树，让你真正的理解 AST抽象语法树。开始手写 AST抽象语法树
  之前，我们先要了解3个常见的算法思想：指针思想、递归缓存和栈。然后利用这些算法思想，实现 AST抽象语法树。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-06-21_18-03-27.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-591e6208933e9df2442ef55251ccdd9b_1440w.jpg"
abbrlink: aa923dbf
date: 2022-07-6 10:38:05
---

# AST 抽象语法树

## 抽象语法树和虚拟节点的关系

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/Snipaste_2022-06-21_18-03-27.jpg)

简而言之，抽象语法树就是将你写的 html 语法拆分成固定格式的对象，该对象可以被渲染函数识别。

## 测试环境的搭建

### 安装依赖

npm i webpack@5 -D

npm i webpack-cli@4 -D

npm i webpack-dev-server@4 -D

### 新建对应文件夹

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/%E6%8A%BD%E8%B1%A1%E8%AF%AD%E6%B3%95%E6%A0%91%E7%9A%84%E6%96%87%E4%BB%B6%E5%A4%B9%E6%A0%B7%E5%BC%8F.jpg)

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

## 源码解析

index.js

JS 的入口文件，将模板字符串传入抽象语法树的处理函数中

```js
import parse from "./parse";

var templateStr = `<div class='ast' id=1>
    <h3>AST抽象语法树</h3>
    <ul>
      <li>递归</li>
      <li>栈</li>
      <li>指针</li>
    </ul>
  </div>`;

//抽象语法树的入口函数
parse(templateStr);
```

parse.js

AST 抽象语法树的入口函数，可以将目标字符串转换为渲染函数可以识别的一个对象

```js
import parseAttrsString from "./parseAttrsString";

export default function (templateStr) {
  //指针，从左到右遍历字符串
  var pointer = 0;
  //存放每次遍历完之后的剩余字符串
  var result = templateStr;
  //用于匹配剩余字符串中开头标签的正则表达式
  var startRegExp = /^\<([a-z]+[1-6]?)(\s[^\>]+)?\>/;
  //用于匹配剩余字符串中结束标签的正则表达式
  var endRegExp = /^\<\/([a-z]+[1-6]?)\>/;
  //用于匹配剩余字符串开头和结束标签之间内容部分的正则表达式
  var wordRegExp = /^([^\<]+)\<\/[a-z]+[1-6]?\>/;
  //辅助栈：用于存放匹配到的标签的栈
  var stack1 = [];
  //内容栈：用于存放匹配到的内容的栈
  var stack2 = [{ children: [] }];

  //循环遍历这个模板字符串
  while (pointer < templateStr.length - 1) {
    //将将模板字符串进行截取，并将截取的字符串存到剩余字符串result中
    result = templateStr.substring(pointer);
    //判断剩余字符串开头是开始标签、结束标签还是具体内容
    if (startRegExp.test(result)) {
      //当匹配到开始标签时
      //捕获开始标签
      let startTag = result.match(startRegExp)[1];
      //捕获开始标签的属性
      let attrsStr = result.match(startRegExp)[2];
      //将开始标签推入stack1栈中
      stack1.push(startTag);
      //向stack2栈中推入一个指定格式的对象
      stack2.push({
        tag: startTag,
        attrs: parseAttrsString(attrsStr),
        type: 1,
        children: [],
      });
      //将指针后移，使其通过整个开始标签
      pointer += startTag.length + 2 + (attrsStr ? attrsStr.length : 0);
    } else if (wordRegExp.test(result)) {
      //当匹配到标签中的内容时
      //捕获结束标签
      let word = result.match(wordRegExp)[1];
      //当匹配到的标签内容不为空时
      if (!/^\s+$/.test(word)) {
        //向stack2栈中的最后一项的children属性中添加一个内容对象
        // @ts-ignore
        stack2[stack2.length - 1].children.push({ text: word, type: 3 });
      }
      //将指针后移
      pointer += word.length;
    } else if (endRegExp.test(result)) {
      //当匹配到结束标签时
      //捕获结束比标签
      let endTag = result.match(endRegExp)[1];
      //判断捕获的标签是否和stack1栈中最后一个元素相同
      if (endTag == stack1[stack1.length - 1]) {
        //如果相同就将标签和内容出栈，即删除stack1和stack2中的最后一个元素
        stack1.pop();
        let content = stack2.pop();
        //将stack2中删除的元素添加到当前stack2中最后一个元素的children属性中
        // @ts-ignore
        stack2[stack2.length - 1].children.push(content);
      } else {
        //如果不相同就说明标签书写有误
        throw new Error("标签没有闭合！！！");
      }
      //处理完结束标签后将指针后移
      pointer += endTag.length + 3;
    } else {
      //匹配到结束标签和下一个开始标签之间的内容时
      pointer++;
    }
  }
  return stack2[0].children[0];
}
```

parseAttrsString.js

parse()函数的辅助函数，用于处理开始标签中属性部分的字符串

```js
/* 处理开始标签中的属性部分 */
export default function (attrsStr) {
  //如果开始标签中没有属性内容就直接返回一个空数组
  if (attrsStr == undefined) return [];
  //标志变量，辅助判断空格所在的位置
  var isYinhao = false;
  //指针变量
  var point = 0;
  //将每次截取的剩余字符串存放到该变量
  var result = [];
  //循环遍历传入的开始标签的属性
  for (let i = 0; i < attrsStr.length; i++) {
    let char = attrsStr[i];
    /* 
      当isYinhao为真时遇到空格不用对该字符串进行截取
      当isYinhao为假时遇到空格进行截取
    */
    if (char == '"') {
      isYinhao = !isYinhao;
    } else if (char == " " && !isYinhao) {
      if (!/^\s*$/.test(attrsStr.substring(point, i))) {
        result.push(attrsStr.substring(point, i).trim());
        point = i;
      }
    }
  }
  //将最后剩余的部分也进行截取
  result.push(attrsStr.substring(point).trim());

  //将拆分的字符串进行二次截取，变成键值对的形式
  result = result.map((item) => {
    const o = item.match(/^(.+)=(.+)$/);
    return {
      name: o[1],
      value: o[2],
    };
  });

  return result;
}
```
