---
title: HTML DOM createDocumentFragment()
tags: DOM
categories: HTML
keywords: DOM
descriptions: >-
  createDocumentFragmen()方法用于创建一个虚拟节点对象，节点对象包含所有属性和方法。当你想要提取文档的一部分，改变，增加，或者删除某些内容及插入到文件末尾可以使用createDocumentFragment()方法，你也可以使用文档的文档对象来执行这些变化，但要防止文件结构被破坏
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-343984765203576c7f973dd3b9309c24_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-67030349108004a92a7b07ce246886a3_1440w.jpg"
abbrlink: bb2a52
date: 2023-03-10 23:44:03
---

# HTML DOM createDocumentFragment()

## 定义

createDocumentFragmen()方法用于创建一个虚拟节点对象，节点对象包含所有属性和方法。当你想要提取文档的一部分，改变，增加，或者删除某些内容及插入到文件末尾可以使用 createDocumentFragment()方法，你也可以使用文档的文档对象来执行这些变化，但要防止文件结构被破坏，createDocumentFragment() 方法可以更安全改变文档的结构及节点。

## 语法

`document.createDocumentFragment()`

## 参数

None

## 返回值

| 类型                  | 描述             |
| :-------------------- | :--------------- |
| DocumentFragment 对象 | 创建文档片段对象 |

## 实例

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
    <ul>
      <li>致良知</li>
      <li>知行合一</li>
    </ul>
    <script>
      var d = document.createDocumentFragment();
      //通过appendChild添加子元素是文档中存在的元素时，会将其在原来文档中删除
      d.appendChild(document.getElementsByTagName("li")[0]);
      d.childNodes[0].childNodes[0].nodeValue = "心即理";
      document.getElementsByTagName("ul")[0].appendChild(d);
    </script>
  </body>
</html>
```

## 特性

1. DocumentFragment 节点不属于文档树，继承的 parentNode 属性总是 null。它有一个很实用的特点，当请求把一个 DocumentFragment 节点插入文档树时，插入的不是 DocumentFragment 自身，而是它的所有子孙节点，即插入的是括号里的节点。这个特性使 DocumentFragment 成了占位符，暂时存放那些一次插入文档的节点。它还有利于实现文档的剪切、复制和粘贴操作。当需要添加多个 dom 元素时，如果先将这些元素添加到 DocumentFragment 中，再统一将 DocumentFragment 添加到页面，会减少页面渲染 dom 的次数，效率会明显提升。
2. 如果使用 appendChid 方法将原 dom 树中的节点添加到 DocumentFragment 中时，会删除原来的节点。

## createDocumentFragment()方法和 createElement()方法的区别：

1. 由于每一次对文档的插入都会引起重新渲染（计算元素的尺寸，显示背景，内容等），所以进行多次插入操作使得浏览器发生了很多次渲染，效率是比较低的。这是我们提倡通过减少页面的渲染来提高 DOM 操作的效率的原因。一个优化的方法是将要创建的元素写到一个字符串上，然后一次性写到 innerHTML 上，这种利用浏览器对 innerHTML 的解析确实是相比上面的多次插入快了很多。但是构造字符串灵活性上面比较差，很难符合创建各种各样的 DOM 元素的需求。利用 DocumentFragment，可以弥补这两个方法的不足。因为文档片段存在于内存中，并不在 DOM 中，所以将子元素插入到文档片段中时不会引起页面回流（对元素位置和几何上的计算），因此使用 DocumentFragment 可以起到性能优化的作用。
2. createElement 创建的元素可以使用 innerHTML，createDocumentFragment 创建的元素使用 innerHTML 并不能达到预期修改文档内容的效果，只是作为一个属性而已。两者的节点类型完全不同，createElement 创建的是元素节点，节点类型为 1，createDocumentFragment 创建的是文档碎片，节点类型是 11。并且 createDocumentFragment 创建的元素在文档中没有对应的标记，因此在页面上只能用 js 中访问到。
3. createElement 创建的元素可以重复操作，添加之后就算从文档里面移除依旧归文档所有，可以继续操作，但是 createDocumentFragment 创建的元素是一次性的，添加之后再就不能操作了（说明：这里的添加并不是添加了新创建的片段，因为上面说过，新创建的片段在文档内是没有对应的标签的，这里添加的是片段的所有子节点）。
4. 通过`createElement`新建元素必须指定元素`tagName`,因为其可用`innerHTML`添加子元素。通过`createDocumentFragment`则不必。
5. 通过 createElement 创建的元素插入文档后，还可以取到创建时的返回值，即上面例子中 createElement 还是创建的那个 div 元素，而 createDocumentFragment 创建的元素插入到文档后，就不能访问创建时的返回值了，相当于把自己创建的文档片段直接挪到文档中了。

## createDocumentFragment()方法和 createElement()方法的共同点：

1. 添加子元素后返回值都是新添加的子元素
2. 都可以通过`appendChild`添加子元素，且子元素必须是`node`类型，不能为文本。
3. **若添加的子元素是文档中存在的元素，则通过`appendChild`在为其添加子元素时，会从文档中删除之前存在的元素。**
