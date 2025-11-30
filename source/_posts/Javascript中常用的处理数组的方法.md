---
title: Javascript中常用的处理数组的方法
tags: Array.prototype
categories: JavaScript
descriptions: >-
  Javascript中常用的处理数组的方法Array.prototype.some()，Array .splice()，Array.filter()
  ，Array .map()
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-75330af94803b3fde75afeb7fafa0b19_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-780d2703ad47bb32a7fc7c218933d9a0_1440w.jpg"
abbrlink: 19d6c420
date: 2021-04-1 13:08:49
---

# Javascript 中常用的处理数组的方法

## Array.prototype.some()

### 描述

`some()` 为数组中的每一个元素执行一次 `callback` 函数，直到找到一个使得 callback 返回一个“真值”（即可转换为布尔值 true 的值）。如果找到了这样一个值，`some()` 将会立即返回 `true`。否则，`some()` 返回 `false`。`callback` 只会在那些”有值“的索引上被调用，不会在那些被删除或从来未被赋值的索引上调用。

### 语法

```
arr.some(callback(element[, index[, array]])[, thisArg])
```

### 参数

- `callback`用来测试每个元素的函数，接受三个参数：
- `element`数组中正在处理的元素。
- `index` 可选数组中正在处理的元素的索引值。
- `array`可选`some()`被调用的数组。
- `thisArg`可选执行 `callback` 时使用的 `this` 值。

### 返回值

数组中有至少一个元素通过回调函数的测试就会返回**`true`**；所有元素都没有通过回调函数的测试返回值才会为 false。

### 示例

```
function isBiggerThan10(element, index, array) {
  return element > 10;
}

[2, 5, 8, 1, 4].some(isBiggerThan10);  // false
[12, 5, 8, 1, 4].some(isBiggerThan10); // true
```

## Array .splice()

### 描述

splice() 方法向/从数组添加/删除项目，并返回删除的项目。

**注释：**splice() 方法会改变原始数组。

### 语法

```js
array.splice(index, howmany, item1, ....., itemX)
```

### 参数

| 参数                | 描述                                                                        |
| ------------------- | --------------------------------------------------------------------------- |
| _index_             | 必需。整数，指定在什么位置添加/删除项目，使用负值指定从数组末尾开始的位置。 |
| _howmany_           | 可选。要删除的项目数。如果设置为 0，则不会删除任何项目。                    |
| _item1, ..., itemX_ | 可选。要添加到数组中的新项目。                                              |

## Array.filter()

### 描述

filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。

**注意：** filter() 不会对空数组进行检测。

**注意：** filter() 不会改变原始数组。

### 语法

```js
array.filter(function(currentValue,index,arr), thisValue)
```

### 参数

| 参数           | 描述                                                                                                            |
| :------------- | --------------------------------------------------------------------------------------------------------------- |
| _currentValue_ | 必须。当前元素的值                                                                                              |
| _index_        | 可选。当前元素的索引值                                                                                          |
| _arr_          | 可选。当前元素属于的数组对象                                                                                    |
| _thisValue_    | 可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。 如果省略了 thisValue ，"this" 的值为 "undefined" |

## Array .map()

### 描述

map() 方法返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。

map() 方法按照原始数组元素顺序依次处理元素。

**注意：** map() 不会对空数组进行检测。

**注意：** map() 不会改变原始数组。

### 语法

```
array.map(function(currentValue,index,arr), thisValue)
```

| 参数           | 描述                                                                                                                                            |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| _currentValue_ | 必须。当前元素的值                                                                                                                              |
| _index_        | 可选。当前元素的索引值                                                                                                                          |
| _arr_          | 可选。当前元素属于的数组对象                                                                                                                    |
| _thisValue_    | 可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。 如果省略了 thisValue，或者传入 null、undefined，那么回调函数的 this 为全局对象。 |
