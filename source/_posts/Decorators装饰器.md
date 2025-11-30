---
title: Decorators装饰器
tags: 装饰器
categories: TS
keywords: Decorators装饰器
description: >-
  随着TypeScript和ES6里引入了类，在一些场景下我们需要额外的特性来支持标注或修改类及其成员。
  装饰器（Decorators）为我们在类的声明及成员上通过元编程语法添加标注提供了一种方式。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-67030349108004a92a7b07ce246886a3_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-343984765203576c7f973dd3b9309c24_1440w.jpg"
abbrlink: f26548b8
date: 2022-8-25 23:45:43
---

> 随着 TypeScript 和 ES6 里引入了类，在一些场景下我们需要额外的特性来支持标注或修改类及其成员。 装饰器（Decorators）为我们在类的声明及成员上通过元编程语法添加标注提供了一种方式。

## 1.装饰器

装饰器是一种特殊的类型声明，他可以附加在类、方法、属性、参数上

定义一个装饰器函数，加上装饰器的类相当于对该类调用了相应的装饰器函数，类似于类的前置操作

装饰器的类型有四种：类装饰器、属性装饰器、方法装饰器、参数装饰器

## 2.使用装饰器

### 启用装饰器

创建 tsconfig.json 配置文件

`tsc --init`

在 TS 的配置文件中开启 experimentalDecorators 选项

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/15image.png)

### demo

```TypeScript
//类装饰器：其中target参数为类
const decClass: ClassDecorator = (target: any) => {
  console.log('类装饰器', target);
  target.prototype.name = '小李';
}
//属性装饰器：其中target为属性所在类的原型对象，key为类的属性
const decProperty: PropertyDecorator = (target: any, key: string | symbol) => {
  console.log('属性装饰器', target, key);
}
//方法装饰器：其中target为属性所在类的原型对象，key方法名，description为描述对象
const decMethod: MethodDecorator = (target: any, key: string | symbol, description: any) => {
  console.log('方法装饰器', target, key, description);
}
//参数装饰器：其中target为属性所在类的原型对象，key方法名，index为参数的所在位置
const decParameter: ParameterDecorator = (target: any, key: string | symbol, index: any) => {
  console.log('参数装饰器', target, key, index);
}

@decClass
class XiaoLi{
  @decProperty
  name: string;
  constructor() {
    console.log('XioaLi');
    this.name = '小李';
  }
  @decMethod
  getName(name: string, @decParameter age: number) {

  }
}

const xiaoli: any = new XiaoLi();
console.log("🚀 ~ file: 装饰器.ts ~ line 19 ~ xiaoli", xiaoli.name)

```
