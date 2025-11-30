---
title: Vue源码之虚拟DOM和diff算法
sticky: 1
tags: Vue源码
categories: Vue
keywords: Vue源码
descriptions: >-
  虚拟dom是当前前端最流行的两个框架（vue和react）都用到的一种技术，都说他能帮助vue和react提升渲染性能，提升用户体验。那么今天我们来详细看看虚拟dom到底是个什么鬼
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/%E5%9B%BE%E8%A7%A3snabbdom.png"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-75330af94803b3fde75afeb7fafa0b19_1440w.jpg"
abbrlink: 8e0a33d3
date: 2022-06-25 10:44:47
---

# Vue 源码之虚拟 DOM 和 diff 算法

## snabbdom 简介

snabdom 是瑞典语，原意为速度

一个精简化、模块化、功能强大、性能卓越的虚拟 DOM 库

## 使用示例

```js
import {
  init,
  classModule,
  propsModule,
  styleModule,
  eventListenersModule,
  h,
} from "snabbdom";

const patch = init([
  // 通过传入模块初始化 patch 函数
  classModule, // 开启 classes 功能
  propsModule, // 支持传入 props
  styleModule, // 支持内联样式同时支持动画
  eventListenersModule, // 添加事件监听
]);

const container = document.getElementById("container");

const vnode = h("div#container.two.classes", { on: { click: someFn } }, [
  h("span", { style: { fontWeight: "bold" } }, "This is bold"),
  " and this is just normal text",
  h("a", { props: { href: "/foo" } }, "I'll take you places!"),
]);
// 传入一个空的元素节点 - 将产生副作用（修改该节点）
patch(container, vnode);

const newVnode = h(
  "div#container.two.classes",
  { on: { click: anotherEventHandler } },
  [
    h(
      "span",
      { style: { fontWeight: "normal", fontStyle: "italic" } },
      "This is now italic type"
    ),
    " and this is still just normal text",
    h("a", { props: { href: "/bar" } }, "I'll take you places!"),
  ]
);
// 再次调用 `patch`
patch(vnode, newVnode); // 将旧节点更新为新节点
```

## 源码解析

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/%E5%9B%BE%E8%A7%A3snabbdom.png)

### 解析 diff 算法

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/diff%E7%AE%97%E6%B3%95%E6%A0%B8%E5%BF%831.jpg)

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/diff%E7%AE%97%E6%B3%95%E6%A0%B8%E5%BF%832.jpg)

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/diff%E7%AE%97%E6%B3%95%E6%A0%B8%E5%BF%833.jpg)

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/diff%E7%AE%97%E6%B3%95%E6%A0%B8%E5%BF%834.jpg)

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/diff%E7%AE%97%E6%B3%95%E6%A0%B8%E5%BF%835.png)

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/diff%E7%AE%97%E6%B3%95%E6%A0%B8%E5%BF%836.jpg)

![](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/diff%E7%AE%97%E6%B3%95%E6%A0%B8%E5%BF%837.png)

### 核心代码

入口文件

```js
import h from "./mysnabbdom/h.js";
import patch from "./mysnabbdom/patch.js";

var container = document.getElementById("container");
//生成一个虚拟DOM
const myVnode = h("ul", {}, [
  h("li", { key: "A" }, "a"),
  h("li", { key: "B" }, "b"),
  h("li", { key: "C" }, "c"),
  h("li", { key: "D" }, "d"),
  h("li", { key: "E" }, "e"),
]);

//将生成的虚拟DOM与旧的DOM进行比较之后添加到页面
patch(container, myVnode);

const vnode2 = h("ul", {}, [
  h("li", { key: "C" }, "cccc"),
  h("li", { key: "E" }, "eeee"),
  h("li", { key: "F" }, "ffff"),
]);

const btn = document.getElementById("btn");

//点击按钮时将vnode1变为vnode2
// @ts-ignore
btn.onclick = function () {
  patch(myVnode, vnode2);
};
```

构造虚拟 DOM

```js
import vnode from "./vnode";

/* 该函数传入的参数类型有三种
     · h('div', {}, '文字')
     · h('div', {}, [])
     · h('div', {}, h())
*/

export default function (sel, data, c) {
  //检查参数的个数
  if (arguments.length != 3) throw new Error("对不起，h函数必须传入三个值");
  //检查c参数的类型
  if (typeof c == "string" || typeof c == "number") {
    return vnode(sel, data, undefined, c, undefined);
  } else if (Array.isArray(c)) {
    //传入的c参数是一个数组
    let children = [];
    for (let i = 0; i < c.length; i++) {
      if (!(typeof c[i] == "object" && c[i].hasOwnProperty("sel"))) {
        throw new Error("传入的数组项中有项不是h()函数");
      } else {
        children.push(c[i]);
      }
    }
    //children收集完毕
    return vnode(sel, data, children, undefined, undefined);
  } else if (typeof c == "object" && c.hasOwnProperty("sel")) {
    //传入的c参数是一个h()函数
  } else {
    throw new Error("传入的c参数类型错误");
  }
}
```

```
export default function vnode (sel, data, children, text, elm) {
  const key = data.key;
  return {
    sel, data, children, text, elm, key
  }
}
```

将最终的 DOM 元素添加真实的 DOM 树上

```js
import createElement from "./createElement";
import patchVnode from "./patchVnode";
import vnode from "./vnode";

export default function (oldVnode, newVnode) {
  //判断传入的第一个参数是虚拟节点还是真实DOM节点
  if (oldVnode.sel == "" || oldVnode.sel == undefined) {
    //当传入的是一个真实的DOM节点时，将真实的DOM节点转换成虚拟的DOM节点
    //注意：转换时暂时不考虑节点中有text属性的情况
    oldVnode = vnode(
      oldVnode.tagName.toLowerCase(),
      {},
      [],
      undefined,
      oldVnode
    );
  }

  //判断oldVnode和newVnode是不是在同一个节点
  if (oldVnode.sel == newVnode.sel && oldVnode.key == newVnode.key) {
    console.log("是同一个节点");
    patchVnode(oldVnode, newVnode);
  } else {
    console.log("不是同一个节点");
    //使用新传入的虚拟DOM生成真实DOM
    let newVnodeElm = createElement(newVnode);
    //将新真实DOM绑定在旧真实DOM之前
    if (oldVnode.elm.parentNode && newVnodeElm) {
      oldVnode.elm.parentNode.insertBefore(newVnodeElm, oldVnode.elm);
    }
    //删除老节点
    oldVnode.elm.parentNode.removeChild(oldVnode.elm);
  }
}
```

递归虚拟 DOM 节点

```js
import createElement from "./createElement";
import patchVnode from "./patchVnode";
import vnode from "./vnode";

export default function (oldVnode, newVnode) {
  //判断传入的第一个参数是虚拟节点还是真实DOM节点
  if (oldVnode.sel == "" || oldVnode.sel == undefined) {
    //当传入的是一个真实的DOM节点时，将真实的DOM节点转换成虚拟的DOM节点
    //注意：转换时暂时不考虑节点中有text属性的情况
    oldVnode = vnode(
      oldVnode.tagName.toLowerCase(),
      {},
      [],
      undefined,
      oldVnode
    );
  }

  //判断oldVnode和newVnode是不是在同一个节点
  if (oldVnode.sel == newVnode.sel && oldVnode.key == newVnode.key) {
    console.log("是同一个节点");
    patchVnode(oldVnode, newVnode);
  } else {
    console.log("不是同一个节点");
    //使用新传入的虚拟DOM生成真实DOM
    let newVnodeElm = createElement(newVnode);
    //将新真实DOM绑定在旧真实DOM之前
    if (oldVnode.elm.parentNode && newVnodeElm) {
      oldVnode.elm.parentNode.insertBefore(newVnodeElm, oldVnode.elm);
    }
    //删除老节点
    oldVnode.elm.parentNode.removeChild(oldVnode.elm);
  }
}
```

跳过虚拟 DOM 转换为 DOM 元素

```js
//真正创建节点，将vnode创建为DOM
export default function createElement(vnode) {
  //根据传过来的虚拟节点对象创建一个新的节点
  let domNode = document.createElement(vnode.sel);
  //判断虚拟节点是否需要递归，即虚拟节点的children属性是一个数组
  if (
    vnode.text &&
    (vnode.children == undefined || vnode.children.length == 0)
  ) {
    //将虚拟节点对象的文本内容放入新创建的节点中
    domNode.innerText = vnode.text;
  } else if (Array.isArray(vnode.children) && vnode.children.length > 0) {
    //虚拟节点属性内部还拥有子节点时
    for (let i = 0; i < vnode.children.length; i++) {
      let ch = vnode.children[i];
      let chDOM = createElement(ch);
      //给上一层创建的节点追加要给子节点
      domNode.appendChild(chDOM);
    }
  }
  //给虚拟节点的elm属性添加值
  vnode.elm = domNode;
  //返回elm
  return vnode.elm;
}
```

diff 算法

```js
import createElement from "./createElement"; //创造一个元素
import patchVnode from "./patchVnode"; //新旧虚拟dom的比较

//判断是否是同一个虚拟节点
function checkSameVnode(a, b) {
  return a.sel == b.sel && a.key == b.key;
}
//更新children属性
export default function updateChildren(parentElm, oldCh, newCh) {
  //旧前
  let oldStartIdx = 0;
  //新前
  let newStartIdx = 0;
  //旧后
  let oldEndIdx = oldCh.length - 1;
  //新后
  let newEndIdx = newCh.length - 1;
  //旧前节点
  let oldStartVnode = oldCh[0];
  //旧后节点
  let oldEndVnode = oldCh[oldEndIdx];
  //新前节点
  let newStartVnode = newCh[0];
  //新后节点
  let newEndVnode = newCh[newEndIdx];

  let keyMap = null;

  //开始循环遍历数组进行新旧DOM的比较
  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    //判断新旧节点中是否有undefined的节点，有就要跳过
    if (oldStartVnode == null || oldCh[oldStartIdx] == undefined) {
      oldStartVnode = oldCh[++oldStartIdx];
    } else if (oldEndVnode == null || oldCh[oldEndIdx] == undefined) {
      oldEndVnode = oldCh[--oldEndIdx];
    } else if (newStartVnode == null || newCh[newStartIdx] == undefined) {
      newStartVnode = newCh[++newStartIdx];
    } else if (newEndVnode == null || newCh[newEndIdx] == undefined) {
      newEndVnode = newCh[--newEndIdx];
    } else if (checkSameVnode(oldStartVnode, newStartVnode)) {
      //命中新前与旧前
      console.log("①将旧前和新前的指针后移");
      //将旧前节点变为新前节点
      patchVnode(oldStartVnode, newStartVnode);
      //将旧前和新前的指针后移
      oldStartVnode = oldCh[++oldStartIdx];
      newStartVnode = newCh[++newStartIdx];
    } else if (checkSameVnode(oldEndVnode, newEndVnode)) {
      //命中新后与旧后
      console.log("②将新后与旧后的指针上移");
      //将旧后节点变为新后节点
      patchVnode(oldEndVnode, newEndVnode);
      //将新后与旧后的指针上移
      oldEndVnode = oldCh[--oldEndIdx];
      newEndVnode = newCh[--newEndIdx];
    } else if (checkSameVnode(oldStartVnode, newEndVnode)) {
      //命中新后与旧前
      console.log("③移动旧前指向的节点到旧后节点之后");
      //将旧前节点变为新后节点
      patchVnode(oldStartVnode, newEndVnode);
      //移动旧前指向的节点到旧后节点之后（node.nextSibling：node节点之后紧跟的一个节点）
      parentElm.insertBefore(oldStartVnode.elm, oldEndVnode.elm.nextSibling);
      //将旧前指针下移新后指针上移
      oldStartVnode = oldCh[++oldStartIdx];
      newEndVnode = newCh[--newEndIdx];
    } else if (checkSameVnode(oldEndVnode, newStartVnode)) {
      //命中新前与旧后
      console.log("④移动旧后指向的节点到旧前节点之前");
      //将旧后节点变为新前节点
      patchVnode(oldEndVnode, newStartVnode);
      //移动旧后指向的节点到旧前节点之前
      parentElm.insertBefore(oldEndVnode.elm, oldStartVnode.elm);
      //将旧后指针上移新前指针下移
      oldEndVnode = oldCh[--oldEndIdx];
      newStartVnode = newCh[++newStartIdx];
    } else {
      /* 四种规则都没有匹配到
            在旧的节点中寻找是否有当前newStartIdx指向的节点
            如果没有直接添加
            如果有将旧节点的属性更新后添加到旧前之前，并将它在oldCh中的位置设为undefine
            处理完之后再将旧前指针下移，处理下一个新节点
      */
      //构建一个映射，在旧的节点中寻找是否有当前newStartIdx指向的节点
      if (!keyMap) {
        keyMap = {};
        //遍历 [旧前，旧后] 形成映射
        for (let i = oldStartIdx; i <= oldEndIdx; i++) {
          const key = oldCh[i].key;
          if (key != undefined) {
            keyMap[key] = i;
          }
        }
      }
      //将上面的寻找结果映射到idInOld上
      let idxInOld = keyMap[newStartVnode.key];
      if (idxInOld == undefined) {
        //idxInOld是一个新的节点
        parentElm.insertBefore(createElement(newStartVnode), oldStartVnode.elm);
      } else {
        //idxInOld是一个旧的节点需要将这个节点移动到指定位置
        const elmToMove = oldCh[idxInOld];
        patchVnode(elmToMove, newStartVnode);
        //需要先将旧节点中要移动的节点undefined，不然没办法添加相同的节点
        oldCh[idxInOld] = undefined;
        //将要移动的节点添加到旧前之前
        parentElm.insertBefore(elmToMove.elm, oldStartVnode.elm);
      }
      //将旧前指针下移处理下一个节点
      newStartVnode = newCh[++newStartIdx];
    }
  }
  /* 判断是要删除节点还是要增加节点 
        如果是新节点先循环完，删除指定的旧节点
        如果是旧节点先循环完，增加节点给旧节点
  */
  if (oldStartIdx > oldEndIdx) {
    //需要新增节点
    console.log("需要新增节点");
    //遍历需要新增的节点，并将这些节点转换为真实DOM之后添加到标志节点之前
    for (let i = newStartIdx; i <= newEndIdx; i++) {
      //insertBefore：可以识别null，将要添加的节点自动追加到最后
      parentElm.insertBefore(createElement(newCh[i]), oldCh[oldStartIdx]);
    }
  } else if (newStartIdx > newEndIdx) {
    //需要删除节点
    console.log("需要删除节点");
    //遍历 [旧前, 旧后] 之间的节点并依次删除
    for (let i = oldStartIdx; i <= oldEndIdx; i++) {
      //oldCh中有节点旧删除，没有旧跳过
      if (oldCh[i]) {
        parentElm.removeChild(oldCh[i].elm);
      }
    }
  }
}
```
