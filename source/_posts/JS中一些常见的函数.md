---
title: JS中一些常见的函数
tags: JS
categories: JS
keywords: JS, 函数
description: 总结一下JavaScript中常见的一些函数，对这些函数的用法和使用规范做一下说明
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/5e13e133098fe155.jpg
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/R-C111.jpg
abbrlink: e0e75f8e
date: 2021-11-17 23:41:16
---

## Array.potortype

### arr.slice()

{% note info no-icon %}arr.slice(start, end)

选择从给定的 start 参数开始的元素，并在给定的 end 处结束，包括 start 但不包括 end

方法从数组中返回选定的元素，作为一个新数组。 {% endnote %}

| 参数  | 描述                                                                                                           |
| ----- | -------------------------------------------------------------------------------------------------------------- |
| start | 可选。整数，指定从哪里开始选择（第一个元素的索引为 0）。使用负数从数组的末尾进行选择。如果省略，则类似于 "0"。 |
| end   | 可选。整数，指定选择结束的位置。如果省略，将选择从开始的位置到数组末尾的所有元素。使用负数从数组末尾进行选择   |

```js
const fruits = ["Banana", "Orange", "Lemon", "Apple", "Mango"];
const citrus = fruits.slice(1, 3);
console.log(citrus);
//Orange,Lemon

var fruits = ["Banana", "Orange", "Lemon", "Apple", "Mango"];
var myBest = fruits.slice(-3, -1);
console.log(myBest);
//Lemon,Apple
```

### arr.reduce()

{% note info no-icon %} arr.reduce(function(total, currentValue, currentIndex, arr), initialValue)

主要功能是做累加处理，即接受一个函数作为累加器将数组中的元素从 左到右执行累加器，返回最终结果 {% endnote %}

| 参数         | 描述                                                                      |
| :----------- | ------------------------------------------------------------------------- |
| total        | 必需，上一次调用累加器是返回值，initial Value 有值时 total = initialValue |
| currentValue | 必需，数组当前元素的值                                                    |
| currentIndex | 可选，数组当前元素值的索引                                                |
| arr          | 可选，当前元素所属的数组对象                                              |
| initialValue | 可选，传递给函数的初始值，若没有则 initialValue = arr[currentIndex]       |

```js
var numbers = [65, 44, 12, 4];
function getSum(total, num) {
  return total + num;
}
let sum = numbers.reduce(getSum);
console.log(sum);
// 125
```

### arr.forEach()

{% note info no-icon %}arr.forEach(callback(currentValue [, index [, array]])[, thisArg])

对数组的每个元素执行一次给定的函数。 {% endnote %}

| 参数         | 描述                                                                                                                                                              |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| currentValue | 数组正在处理的当前元素                                                                                                                                            |
| index        | 数组正在处理的当前元素的索引                                                                                                                                      |
| array        | 数组本身                                                                                                                                                          |
| thisArg      | 可选，如果 thisArg 参数有值，则每次 callback 函数被调用时，this 都会指向 thisArg 参数。如果省略了 thisArg 参数，或者其值为 null 或 undefined，this 则指向全局对象 |

```js
const arraySparse = [1, 3, , 7];
let numCallbackRuns = 0;

arraySparse.forEach(function (element) {
  console.log(element);
  numCallbackRuns++;
});

console.log("numCallbackRuns: ", numCallbackRuns);

// 1
// 3
// 7
// numCallbackRuns: 3
```

### bind()、call()、apply()

#### function.bind()

{% note info no-icon %} function.bind(thisArg[, arg1[, arg2[, ...]]])

bind()方法创建一个新的函数，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。返回值是一个原函数的拷贝值，并拥有指定的 this 值和初始参数{% endnote %}

| 参数         | 描述                                                                                                                                                                                                                                                                                                                    |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| thisArg      | 调用绑定函数时作为 this 参数传递给目标函数的值。 如果使用 new 运算符构造绑定函数，则忽略该值。当使用 bind 在 setTimeout 中创建一个函数（作为回调提供）时，作为 thisArg 传递的任何原始值都将转换为 object。如果 bind 函数的参数列表为空，或者 thisArg 是 null 或 undefined，执行作用域的 this 将被视为新函数的 thisArg。 |
| arg1,arg2... | 当目标函数被调用时，被预置入绑定函数的参数列表中的参数。                                                                                                                                                                                                                                                                |

```js
this.x = 9; // 在浏览器中，this 指向全局的 "window" 对象
var module = {
  x: 81,
  getX: function () {
    return this.x;
  },
};

module.getX(); // 81

var retrieveX = module.getX;
retrieveX();
// 返回 9 - 因为函数是在全局作用域中调用的

// 创建一个新函数，把 'this' 绑定到 module 对象
// 新手可能会将全局变量 x 与 module 的属性 x 混淆
var boundGetX = retrieveX.bind(module);
boundGetX(); // 81
```

#### function.call()

{% note info no-icon %} function.call(thisArg, arg1, arg2, ...)

call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。{% endnote %}

{% note info no-icon %} call() call() 允许为不同的对象分配和调用属于一个对象的函数/方法。

call() 提供新的 this 值给当前调用的函数/方法。你可以使用 call 来实现继承：写一个方法，然后让另外一个新的对象来继承它（而不是在新对象中再写一次这个方法）。{% endnote %}

| 参数         | 描述                                                                                                                                                                                         |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| thisArg      | 可选的。在 function 函数运行时使用的 this 值。请注意，this 可能不是该方法看到的实际值：如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象，原始值会被包装。 |
| arg1,arg2... | 传给 thisArg 函数的参数                                                                                                                                                                      |

```js
function Product(name, price) {
  this.name = name;
  this.price = price;
}

function Food(name, price) {
  Product.call(this, name, price);
  this.category = "food";
}

function Toy(name, price) {
  Product.call(this, name, price);
  this.category = "toy";
}

var cheese = new Food("feta", 5);
var fun = new Toy("robot", 40);
```

#### func.apply()

{% note info no-icon %}func.apply(thisArg, [argsArray])

方法调用一个具有给定 this 值的函数，以及以一个数组（或[类数组对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Indexed_collections#working_with_array-like_objects)）的形式提供的参数。返回值是调用有指定 this 值和参数的函数的结果。{% endnote %}

| 参数      | 描述                                                                                                                                                                                                                   |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| thisArg   | 必选的。在 func 函数运行时使用的 this 值。请注意，this 可能不是该方法看到的实际值：如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象，原始值会被包装。                               |
| argsArray | 可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 func 函数。如果该参数的值为 null 或 undefined，则表示不需要传入任何参数。从 ECMAScript 5 开始可以使用类数组对象。 浏览器兼容性 请参阅本文底部内容。 |

```js
var array = ["a", "b"];
var elements = [0, 1, 2];
array.push.apply(array, elements);
console.info(array); // ["a", "b", 0, 1, 2]
```

#### bind()、call()、apply()的区别

| 函数    | 区别                                                       |
| ------- | ---------------------------------------------------------- |
| call()  | 立即调用前面的数，参数是一个一个的传入的                   |
| apply() | 立即调用前面的数，参数是以数组的形式传入的                 |
| bind()  | 不会立即调用，而会生成一个新的函数，参数是一个一个的传入的 |

## Object.prototype

### Object.prototype.valueOf()

{% note info no-icon %} valueOf()是 object 的原型方法，他定义在 Object.prototype 对象上，所有实例对象搜会继承 valuueOf()方法 。valueOf()返回指定对象的原始值{% endnote %}

不同类型对象的 valueOf()方法的返回值

| **对象** | **返回值**                                               |
| :------- | :------------------------------------------------------- |
| Array    | 返回数组对象本身。                                       |
| Boolean  | 布尔值。                                                 |
| Date     | 存储的时间是从 1970 年 1 月 1 日午夜开始计的毫秒数 UTC。 |
| Function | 函数本身。                                               |
| Number   | 数字值。                                                 |
| Object   | 对象本身。这是默认情况。                                 |
| String   | 字符串值。                                               |
|          | Math 和 Error 对象没有 valueOf 方法。                    |

```js
// Array：返回数组对象本身
var array = ["ABC", true, 12, -5];
console.log(array.valueOf() === array); // true

// Date：当前时间距1970年1月1日午夜的毫秒数
var date = new Date(2013, 7, 18, 23, 11, 59, 230);
console.log(date.valueOf()); // 1376838719230

// Number：返回数字值
var num = 15.2654;
console.log(num.valueOf()); // 15.2654

// 布尔：返回布尔值true或false
var bool = true;
console.log(bool.valueOf() === bool); // true

// new一个Boolean对象
var newBool = new Boolean(true);
// valueOf()返回的是true，两者的值相等
console.log(newBool.valueOf() == newBool); // true
// 但是不全等，两者类型不相等，前者是boolean类型，后者是object类型
console.log(newBool.valueOf() === newBool); // false

// Function：返回函数本身
function foo() {}
console.log(foo.valueOf() === foo); // true
var foo2 = new Function("x", "y", "return x + y;");
console.log(foo2.valueOf());
/*
ƒ anonymous(x,y
) {
return x + y;
}
*/

// Object：返回对象本身
var obj = { name: "张三", age: 18 };
console.log(obj.valueOf() === obj); // true

// String：返回字符串值
var str = "http://www.xyz.com";
console.log(str.valueOf() === str); // true

// new一个字符串对象
var str2 = new String("http://www.xyz.com");
// 两者的值相等，但不全等，因为类型不同，前者为string类型，后者为object类型
console.log(str2.valueOf() === str2); // false
```

### Object.prototype.toString()

{% note info no-icon %} toString()方法返回一个表示该对象的字符串。{% endnote %}

```js
function Dog(name) {
  this.name = name;
}

const dog1 = new Dog("Gabby");

Dog.prototype.toString = function dogToString() {
  return `${this.name}`;
};

console.log(dog1.toString());
// expected output: "Gabby"
```
