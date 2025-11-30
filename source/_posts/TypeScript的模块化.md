---
title: TypeScript的模块化
sticky: 1
tags: 模块化
categories: TS
keywords: TS模块化
descriptions: >-
  模块在其自身的作用域里执行，而不是在全局作用域里；这意味着定义在一个模块里的变量，函数，类等等在模块外部是不可见的，除非你明确地使用`export`形式之一导出它们。
  相反，如果想使用其它模块导出的变量，函数，类，接口等的时候，你必须要导入它们，可以使用`import`形式之一。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-3d7a580c5009e9e61e3cab628d497316_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-75330af94803b3fde75afeb7fafa0b19_1440w.jpg"
abbrlink: 6a971e7a
date: 2022-11-25 00:14:36
---

# TypeScript 中的模块

模块在其自身的作用域里执行，而不是在全局作用域里；这意味着定义在一个模块里的变量，函数，类等等在模块外部是不可见的，除非你明确地使用`export`形式之一导出它们。 相反，如果想使用其它模块导出的变量，函数，类，接口等的时候，你必须要导入它们，可以使用`import`形式之一。

模块是自声明的；两个模块之间的关系是通过在文件级别上使用 imports 和 exports 建立的。

模块使用模块加载器去导入其它的模块。 在运行时，模块加载器的作用是在执行此模块代码前去查找并执行这个模块的所有依赖。 大家最熟知的 JavaScript 模块加载器是服务于 Node.js 的[CommonJS](https://en.wikipedia.org/wiki/CommonJS)和服务于 Web 应用的[Require.js](http://requirejs.org/)。

## 导出

任何声明（比如变量，函数，类，类型别名或接口）都能够通过添加`export`关键字来导出。

```TypeScript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}

export const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

导出语句很便利，因为我们可能需要对导出的部分重命名，所以上面的例子可以这样改写：

```TypeScript
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

## 重新导出

我们经常会去扩展其它模块，并且只导出那个模块的部分内容。 重新导出功能并不会在当前模块导入那个模块或定义一个新的局部变量。

**`export * from “module”`**

```TypeScript
export class ParseIntBasedZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && parseInt(s).toString() === s;
    }
}

// 导出原先的验证器但做了重命名
export {ZipCodeValidator as RegExpBasedZipCodeValidator} from "./ZipCodeValidator";

export * from "./StringValidator"; // exports interface StringValidator
export * from "./LettersOnlyValidator"; // exports class LettersOnlyValidator
export * from "./ZipCodeValidator";  // exports class ZipCodeValidator
```

## 导入

```TypeScript
import { ZipCodeValidator } from "./ZipCodeValidator";

let myValidator = new ZipCodeValidator();

//重命名
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```

将整个模块导入到一个变量，并通过它来访问模块的导出部分

```TypeScript
import * as validator from "./ZipCodeValidator";

let myValidator = new validator.ZipCodeValidator();
```

## 默认导出

每个模块都可以有一个`default`导出。 默认导出使用`default`关键字标记；并且一个模块只能够有一个`default`导出。 需要使用一种特殊的导入形式来导入`default`导出。

```TypeScript
//JQuery.d.ts
declare let $: JQuery;
export default $;

//APP.ts
import $ from "JQuery";

$("button.continue").html( "Next Step..." );
```

类和函数声明可以直接被标记为默认导出。 标记为默认导出的类和函数的名字是可以省略的。

```TypeScript
//ZipCodeValidator.ts
export default class ZipCodeValidator {
    static numberRegexp = /^[0-9]+$/;
    isAcceptable(s: string) {
        return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
    }
}

//Test.ts
import validator from "./ZipCodeValidator";

let myValidator = new validator();
```

`default`导出也可以是一个值

## export =  和  import = require()

`export =`语法定义一个模块的导出对象。 它可以是类，接口，命名空间，函数或枚举。

若要导入一个使用了`export =`的模块时，必须使用 TypeScript 提供的特定语法`import module = require("module")`。

### export = 导出

```TypeScript
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

### import module = require("module")导入

```TypeScript
import zip = require("./ZipCodeValidator");

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validator = new zip();

// Show whether each string passed each validator
strings.forEach(s => {
  console.log(`"${ s }" - ${ validator.isAcceptable(s) ? "matches" : "does not match" }`);
});
```

## 外部模块

在 Node.js 里大部分工作是通过加载一个或多个模块实现的。 我们可以使用顶级的`export`声明来为每个模块都定义一个`.d.ts`文件，但最好还是写在一个大的`.d.ts`文件里。 我们使用与构造一个外部命名空间相似的方法，但是这里使用`module`关键字并且把名字用引号括起来，方便之后`import`。

```TypeScript
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export let sep: string;
}
```

现在我们可以`/// <reference>` `node.d.ts`并且使用`import url = require("url");`或`import * as URL from "url"`加载模块。

```TypeScript
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");
```

## 模块声明通配符

某些模块加载器如[SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/overview.md#plugin-syntax)  和[AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md)支持导入非 JavaScript 内容。 它们通常会使用一个前缀或后缀来表示特殊的加载语法。 模块声明通配符可以用来表示这些情况。

```TypeScript
declare module "*!text" {
    const content: string;
    export default content;
}
// Some do it the other way around.
declare module "json!*" {
    const value: any;
    export default value;
}
```

现在你可以就导入匹配`"*!text"`或`"json!*"`的内容了。

```TypeScript
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```

## 创建模块结构指导

1. 如果仅导出单个  `class`  或  `function`，使用  `export default`

```TypeScript
export default class SomeType {
  constructor() { ... }
}
```

```TypeScript
export default function getThing() { return 'thing'; }
```

```TypeScript
import t from "./MyClass";
import f from "./MyFunc";
let x = new t();
console.log(f());
```

2. 如果要导出多个对象，把它们放在顶层里导出

```Plain Text
export class SomeType { /* ... */ }
export function someFunc() { /* ... */ }
```

```TypeScript
import { SomeType, SomeFunc } from "./MyThings";
let x = new SomeType();
let y = someFunc();
```

3. 使用命名空间导入模式当你要导出大量内容的时候

```TypeScript
export class Dog { ... }
export class Cat { ... }
export class Tree { ... }
export class Flower { ... }
```

```TypeScript
import * as myLargeModule from "./MyLargeModule.ts";
let x = new myLargeModule.Dog();
```

## 使用重新导出进行扩展

你可能经常需要去扩展一个模块的功能。 JS 里常用的一个模式是 JQuery 那样去扩展原对象。 如我们之前提到的，模块不会像全局命名空间对象那样去*合并*。 推荐的方案是*不要*去改变原来的对象，而是导出一个新的实体来提供新的功能。

```TypeScript
export class Calculator {
    private current = 0;
    private memory = 0;
    private operator: string;

    protected processDigit(digit: string, currentValue: number) {
        if (digit >= "0" && digit <= "9") {
            return currentValue * 10 + (digit.charCodeAt(0) - "0".charCodeAt(0));
        }
    }

    protected processOperator(operator: string) {
        if (["+", "-", "*", "/"].indexOf(operator) >= 0) {
            return operator;
        }
    }

    protected evaluateOperator(operator: string, left: number, right: number): number {
        switch (this.operator) {
            case "+": return left + right;
            case "-": return left - right;
            case "*": return left * right;
            case "/": return left / right;
        }
    }

    private evaluate() {
        if (this.operator) {
            this.memory = this.evaluateOperator(this.operator, this.memory, this.current);
        }
        else {
            this.memory = this.current;
        }
        this.current = 0;
    }

    public handelChar(char: string) {
        if (char === "=") {
            this.evaluate();
            return;
        }
        else {
            let value = this.processDigit(char, this.current);
            if (value !== undefined) {
                this.current = value;
                return;
            }
            else {
                let value = this.processOperator(char);
                if (value !== undefined) {
                    this.evaluate();
                    this.operator = value;
                    return;
                }
            }
        }
        throw new Error(`Unsupported input: '${char}'`);
    }

    public getResult() {
        return this.memory;
    }
}

export function test(c: Calculator, input: string) {
    for (let i = 0; i < input.length; i++) {
        c.handelChar(input[i]);
    }

    console.log(`result of '${input}' is '${c.getResult()}'`);
}
```

```TypeScript
import { Calculator, test } from "./Calculator";


let c = new Calculator();
test(c, "1+2*33/11="); // prints 9
```

现在扩展它，添加支持输入其它进制（十进制以外），让我们来创建`ProgrammerCalculator.ts`。

```TypeScript
import { Calculator } from "./Calculator";

class ProgrammerCalculator extends Calculator {
    static digits = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"];

    constructor(public base: number) {
        super();
        if (base <= 0 || base > ProgrammerCalculator.digits.length) {
            throw new Error("base has to be within 0 to 16 inclusive.");
        }
    }

    protected processDigit(digit: string, currentValue: number) {
        if (ProgrammerCalculator.digits.indexOf(digit) >= 0) {
            return currentValue * this.base + ProgrammerCalculator.digits.indexOf(digit);
        }
    }
}

// Export the new extended calculator as Calculator
export { ProgrammerCalculator as Calculator };

// Also, export the helper function
export { test } from "./Calculator";
```

```TypeScript
import { Calculator, test } from "./ProgrammerCalculator";

let c = new Calculator(2);
test(c, "001+010="); // prints 3
```
