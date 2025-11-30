---
title: axios的配置和基本使用方法
tags: 前端
categories: axios
keywords: axios
description: axios是一个基于Promise的HTTP客户端，可以用在浏览器和node.js中，向服务器发送AJAX请求进行数据交换，本篇文章将讲解axios的基本配置和使用。
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/OIP-C.jpg
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/axioslogo1440x600.jpg
abbrlink: 6087338d
date: 2021-11-22 07:48:37
---

## Installing

{% note success no-icon %}首先将 axios 安装到全局，或用 script 标签直接导入 {% endnote %}

Using npm:

```js
npm install axios
```

Using bower:

```js
bower install axios
```

Using yarn:

```js
yarn add axios
```

Using jsDelivr CDN:

```js
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

国内 CDN：

```js
<script src="https://cdn.bootcdn.net/ajax/libs/axios/0.21.1/axios.js"></script>
```

Using unpkg CDN:

```js
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

## 基本使用

### 使用 JSON Server 配置一个可被访问的服务器

{% note default simple %} 在全局安装 JSON Server {% endnote %}

```js
npm install -g json-server
```

{% note default simple %} 在根目录下创建一个 db.json 文件存放服务器数据{% endnote %}

```js
{
  "posts": [
    {
        "id": 1,
        "title": "json-server",
        "author": "typicode"
    }
  ],
  "comments": [
    {
        "id": 1,
        "body": "some comment",
        "postId": 1
    }
  ],
  "profile": { "name": "typicode" }
}
```

{% note default simple %} 在终端中启动服务器 {% endnote %}

```js
json-server --watch db.json
```

{% note default simple %} 默认路由 {% endnote %}

多路路由

```
GET    /posts
GET    /posts/1
POST   /posts
PUT    /posts/1
PATCH  /posts/1
DELETE /posts/1
```

单路路由

```
GET    /profile
POST   /profile
PUT    /profile
PATCH  /profile
```

### axios 的配置

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>axios的基本使用</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/axios/0.21.1/axios.min.js"></script>
    <style>
      div {
        padding: 20px 80px;
        text-align: center;
      }

      button {
        font-size: large;
        border-radius: 5px;
        border: 0px;
        padding: 5px 10px;
      }

      .btn1 {
        background-color: #ff7959;
      }

      .btn2 {
        background-color: #8a76b5;
      }

      .btn3 {
        background-color: #ef4e00;
      }

      .btn4 {
        background-color: #e39001;
      }
    </style>
  </head>

  <body>
    <div>
      <h2>基本使用</h2>
      <button class="btn1">发送GET请求</button>
      <button class="btn2">发送POST请求</button>
      <button class="btn3">发送PUT请求</button>
      <button class="btn4">发送DELETE请求</button>
    </div>
    <script>
      const btn = document.querySelectorAll("button");

      //GET获取服务器的信息
      btn[0].addEventListener("click", function () {
        //发送Ajax请求
        axios({
          //设置请求方式
          method: "GET",
          //设置请求的地址(最后面的数字代表数组中id为2的对象)
          url: "http://localhost:3000/posts/2",
          //axios会返回一个promise对象，将返回的promise对象进行处理
        }).then((response) => {
          //打印响应体
          console.log(response);
        });
      });

      //POST给服务器添加数据
      btn[1].addEventListener("click", function () {
        //发送ajax请求
        axios({
          //设置请求方式
          method: "POST",
          //将要添加的data数据放到posts数组中
          url: "http://localhost:3000/posts",
          //设置要添加的数据
          data: {
            title: "爱你所爱，行你所行，听从你心你，无问西东",
            author: "Mr'L",
          },
        }).then((response) => {
          console.log(response);
        });
      });

      //PUT更新(修改)服务器的数据
      btn[2].addEventListener("click", function () {
        //发送ajax请求
        axios({
          //设置请求方式
          method: "PUT",
          //将posts数组中id=3的数据进行修改
          url: "http://localhost:3000/posts/3",
          //设置修改的数据
          data: {
            title:
              "浮世三千，吾爱有三，日月于卿，日为朝，月为暮，吾原与卿朝朝暮暮",
            author: "Mr'L",
          },
        }).then((response) => {
          console.log(response);
        });
      });

      //DELETE删除服务器中的某条数据
      btn[3].addEventListener("click", function () {
        //发送ajax请求
        axios({
          //设置请求方式
          method: "delete",
          //设置要删除的数据（将posts中id=4的数组对象删除）
          url: "http://localhost:3000/posts/4",
        }).then((response) => {
          console.log(response);
        });
      });
    </script>
  </body>
</html>
```
