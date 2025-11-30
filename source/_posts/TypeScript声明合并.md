---
title: TypeScript声明合并
tags: TS
categories: TypeScript声明合并
keywords: TypeScript声明合并
description: >-
  对本文件来讲，“声明合并”是指编译器将针对同一个名字的两个独立声明合并为单一声明。 合并后的声明同时拥有原先两个声明的特性。
  任何数量的声明都可被合并；不局限于两个声明。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/iTab-l3po5l.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/iTab-pkz5r9.png"
abbrlink: 611eac30
date: 2022-08-22 23:42:27
---

> 对本文件来讲，“声明合并”是指编译器将针对同一个名字的两个独立声明合并为单一声明。 合并后的声明同时拥有原先两个声明的特性。 任何数量的声明都可被合并；不局限于两个声明。

## 合并接口

接口的非函数的成员应该是唯一的。 如果它们不是唯一的，那么它们必须是相同的类型。 如果两个接口中同时声明了同名的非函数成员且它们的类型不同，则编译器会报错。

```TypeScript
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

let box: Box = {height: 5, width: 6, scale: 10};
```

对于函数成员，每个同名函数声明都会被当成这个函数的一个重载。 同时需要注意，当接口`A`与后来的接口`A`合并时，后面的接口具有更高的优先级。

```TypeScript
interface Cloner {
    clone(animal: Animal): Animal;
}

interface Cloner {
    clone(animal: Sheep): Sheep;
}

//合并后
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
}
```

## 合并命名空间

对于命名空间里值的合并，如果当前已经存在给定名字的命名空间，那么后来的命名空间的导出成员会被加到已经存在的那个模块里。

```TypeScript
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}

//合并后
namespace Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
}
```

除了这些合并外，你还需要了解非导出成员是如何处理的。 非导出成员仅在其原有的（合并前的）命名空间内可见。这就是说合并之后，从其它命名空间合并进来的成员无法访问非导出成员。

```TypeScript
namespace Animal {
    let haveMuscles = true;

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

namespace Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // <-- error, haveMuscles is not visible here
    }
}
```

## 合并命名空间和类

合并规则与上面`合并命名空间`小节里讲的规则一致，我们必须导出`AlbumLabel`类，好让合并的类能访问。 合并结果是一个类并带有一个内部类。 你也可以使用命名空间为类增加一些静态属性。

```TypeScript
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}
```

除了内部类的模式，你在 JavaScript 里，创建一个函数稍后扩展它增加一些属性也是很常见的。 TypeScript 使用声明合并来达到这个目的并保证类型安全。

```TypeScript
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

alert(buildLabel("Sam Smith"));
```

相似的，命名空间可以用来扩展枚举型：

```TypeScript
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```

## 模块扩展

虽然 JavaScript 不支持合并，但你可以为导入的对象打补丁以更新它们

```TypeScript
// observable.js
export class Observable<T> {
    // ... implementation left as an exercise for the reader ...
}

// map.js
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}
```

它也可以很好地工作在 TypeScript 中， 但编译器对  `Observable.prototype.map`一无所知。 你可以使用扩展模块来将它告诉编译器：

```TypeScript
// observable.ts stays the same
// map.ts
import { Observable } from "./observable";
declare module "./observable" {
    interface Observable<T> {
        map<U>(f: (x: T) => U): Observable<U>;
    }
}
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}


// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number>;
o.map(x => x.toFixed());
```

## 全局扩展

```TypeScript
// observable.ts
export class Observable<T> {
    // ... still no implementation ...
}

declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}

Array.prototype.toObservable = function () {
    // ...
}
```
