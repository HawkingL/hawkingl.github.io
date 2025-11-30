---
title: axios的深度理解和使用
tags: axios
categories: 前端
keywords: axios的使用
description: axios是基于xhr + promise的异步ajax请求库，浏览器端/node端都可以使用，支持请求／响应拦截器，支持请求取消，请求/响应数据转换，批量发送多个请求
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-76cbbf3febd707449a4df2f836894b7c_1440w.jpg
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/20220219210440.png
abbrlink: d9e8b7c1
date: 2022-02-19 20:29:11
---

## axios 特点

1. 基于 xhr + promise 的异步 ajax 请求库
2. 浏览器端/node 端都可以使用
3. 支持请求／响应拦截器
4. 支持请求取消
5. 请求/响应数据转换
6. 批量发送多个请求

## axios 常用语法

`axios(config)`: 通用/最本质的发任意类型请求的方式
`axios(url[, config])`: 可以只指定 url 发 get 请求
`axios.request(config)`: 等同于 axios(config)
`axios.get(url[, config])`: 发 get 请求
`axios.delete(url[, config])`: 发 delete 请求
`axios.post(url[, data, config])`: 发 post 请求
`axios.put(url[, data, config])`: 发 put 请求

`axios.defaults.xxx`: 请求的默认全局配置（method\baseURL\params\timeout…）
`axios.interceptors.request.use()`: 添加请求拦截器
`axios.interceptors.response.use()`: 添加响应拦截器

`axios.create([config])`: 创建一个新的 axios(它没有下面的功能)

`axios.Cancel()`: 用于创建取消请求的错误对象
`axios.CancelToken()`: 用于创建取消请求的 token 对象
`axios.isCancel()`: 是否是一个取消请求的错误
`axios.all(promises)`: 用于批量执行多个异步请求
`axios.spread()`: 用来指定接收所有成功数据的回调函数的方法

## 难点语法的理解和使用

### `axios.create(config)`

1. 根据指定配置创建一个新的 axios, 也就是每个新 axios 都有自己的配置
2. 新 axios 只是没有**取消请求**和**批量发请求**的方法, 其它所有语法都是一致的
3. 为什么要设计这个语法?
   (1) 需求: 项目中有部分接口需要的配置与另一部分接口需要的配置不太一样, 如何处理（比如有多个 baseURL 需要指定）
   (2) 解决: 创建 2 个新 axios, 每个都有自己特有的配置, 分别应用到不同要求的接口请求中

```
const instance = axios.create({ // instance是函数类型
	baseURL: 'http://localhost:3000'
})
// 使用instance发Ajax请求
instance({
	url: '/posts'
})
instance.get('/posts')

```

### 拦截器函数/ajax 请求/请求的回调函数的调用顺序

1. 说明: 调用`axios()`并不是立即发送 ajax 请求, 而是需要经历一个较长的流程
2. 流程: 请求拦截器 2 => 请求拦截器 1 => 发 ajax 请求 => 响应拦截器 1 => 响应拦截器 2 => 请求的回调
3. 注意: 此流程是通过 promise 串连起来的, 请求拦截器传递的是`config`, 响应拦截器传递的是`response`

```
// 添加两个请求拦截器(回调函数)
axios.interceptors.request.use(
  config => {
    console.log('request interceptor1 onResolved()') // -----------2
    return config
  },
  error => {
    console.log('request interceptor1 onRejected()')
    return Promise.reject(error)
  }
)

axios.interceptors.request.use(
  config => {
    console.log('request interceptor2 onResolved()') // -----------1
    return config
  },
  error => {
    console.log('request interceptor2 onRejected()')
    return Promise.reject(error)
  }
)

// 添加两个响应拦截器
axios.interceptors.response.use(
  resopnse => {
    console.log('response interceptor1 onResolved()') // -----------3
    return resopnse
  },
  error => {
    console.log('response interceptor1 onRejected()')
    return Promise.reject(error)
  }
)

axios.interceptors.response.use(
  resopnse => {
    console.log('response interceptor2 onResolved()') // -----------4
    return resopnse
  },
  error => {
    console.log('response interceptor2 onRejected()')
    return Promise.reject(error)
  }
)

axios.get('http://localhost:3000/posts')
  .then(response => {
    console.log('data', response.data) //data Array(4) -------------5
  })
  .catch(error => {
    cosole.log('error', error.message)
  })

// request interceptor2 onResolved()
// request interceptor1 onResolved()
// response interceptor1 onResolved()
// response interceptor2 onResolved()
// data Array(4)

```

### 取消请求

#### 1. 基本流程

1. 配置 `cancelToken` 对象
2. 缓存用于取消请求的 `cancel` 函数
3. 在后面特定时机调用 `cancel` 函数取消请求
4. 在错误回调中判断如果 `error` 是 `cancel`, 做相应处理

#### 2. 实现功能

1.用 express 先搭建一个有延迟的服务器

```
const express = require('express')
const cors = require('cors')

const app = express()

// 使用cors, 允许跨域
app.use(cors())
// 能解析urlencode格式的post请求体参数
app.use(express.urlencoded())
// 能解析json格式的请求体参数
app.use(express.json())

app.get('/products1', (req, res) => {

  setTimeout(() => {
    res.send([
      {id: 1, name: 'product1.1'},
      {id: 2, name: 'product1.2'},
      {id: 3, name: 'product1.3'}
    ])
  }, 1000 + Math.random()*2000);

})

app.get('/products2', (req, res) => {

  setTimeout(() => {
    res.send([{
        id: 1,
        name: 'product2.1'
      },
      {
        id: 2,
        name: 'product2.2'
      },
      {
        id: 3,
        name: 'product2.3'
      }
    ])
  }, 1000 + Math.random() * 2000);

})

app.listen(4000, () => {
  console.log('server app start on port 4000')
})


```

2.点击按钮, 取消某个正在请求中的请求

```
let cancel // 用于保存取消请求的函数
function getProducts1() {
  axios({
    url: 'http://localhost:4000/products1',
    cancelToken: new axios.CancelToken(function executor(c){ // c是用于取消当前请求的函数
      // 保存取消函数，用于之后可能需要取消当前请求
      cancel = c;
    })
  }).then(
    response => {
      cancel = null
      console.log('请求1成功了', response.data)
    },
    error => {
      cancel = null
      console.log('请求1失败了', error.message, error) // 请求1失败了 强制取消请求 Cancel {message: "强制取消请求"}
    }
  )

}

function getProducts2() {
  axios({
      url: 'http://localhost:4000/products2'
  }).then(
    response => {
      console.log('请求2成功了', response.data)
    }
  )
}

function cancelReq() {
  // alert('取消请求')
  // 执行取消请求的函数
  if (typeof cancel === 'function'){
    cancel('强制取消请求')
  } else {
    console.log('没有可取消的请求')
  }
}

```

3.在请求一个接口前, 取消前面一个未完成的请求

```
let cancel // 用于保存取消请求的函数
function getProducts1() {
  // 在准备发请求前，取消未完成的请求
  if (typeof cancel === 'function'){
    cancel('取消请求')
  }
  axios({
    url: 'http://localhost:4000/products1',
    cancelToken: new axios.CancelToken(function executor(c){ // c是用于取消当前请求的函数
      // 保存取消函数，用于之后可能需要取消当前请求
      cancel = c;
    })
  }).then(
    response => {

      cancel = null
      console.log('请求1成功了', response.data)
    },
    error => {
      if (axios.isCancel(error)){
        console.log('请求1取消的错误', error.message)
      }else{ // 请求出错了
        cancel = null
        console.log('请求1失败了', error.message, error) // 请求1失败了 强制取消请求 Cancel {message: "强制取消请求"}
      }
    }
  )

}

function getProducts2() {
// 在准备发请求前，取消未完成的请求
  if (typeof cancel === 'function'){
    cancel('取消请求')
  }
  axios({
      url: 'http://localhost:4000/products2',
      cancelToken: new axios.CancelToken(function executor(c){
      cancel = c;
    })
  }).then(
    response => {
      cancel = null
      console.log('请求2成功了', response.data)
    },
    error => {
      if (axios.isCancel(error)){
        console.log('请求2取消的错误', error.message)
      }else{
        cancel = null
        console.log('请求2失败了', error.message, error)
      }
    }
  )
}
function cancelReq() {
  // alert('取消请求')
  // 执行取消请求的函数
  if (typeof cancel === 'function'){
    cancel('强制取消请求')
  } else {
    console.log('没有可取消的请求')
  }
}

```

#### 使用拦截器改进代码，减少重复代码

```
// 添加请求拦截器
axios.interceptors.request.use((config) => { // 只写一个成功的回调
  // 在准备发请求前，取消未完成的请求
  if (typeof cancel === 'function'){
    cancel('取消请求')
  }
  // 添加一个cancelToken的配置
  config.cancelToken = new axios.CancelToken(function executor(c){ // c是用于取消当前请求的函数
      // 保存取消函数，用于之后可能需要取消当前请求
      cancel = c;
    })
    return config
})

// 添加响应拦截器
axios.interceptors.response.use(
  response => { // 成功的回调
    cancel = null
    return response
  },
  error => { // 失败的回调
    if (axios.isCancel(error)){ // 请求取消的错误
        console.log('请求取消的错误', error.message)
        // 中断promise链
        return new Promise(() => {})
    }else{ // 请求出错了
      cancel = null
      // 将错误向下传递
      // throw error
      return Promise.reject(error)
    }
  }
)

let cancel // 用于保存取消请求的函数
function getProducts1() {
  axios({
    url: 'http://localhost:4000/products1'
  }).then(
    response => {
      console.log('请求1成功了', response.data)
    },
    error => { // 只用处理请求失败的情况，取消请求的错误不用处理
      console.log('请求1失败了', error.message, error)
    }
  )
}

function getProducts2() {
  axios({
      url: 'http://localhost:4000/products2'
  }).then(
    response => {
      console.log('请求2成功了', response.data)
    },
    error => {
      console.log('请求2失败了', error.message, error)
    }
  )
}
function cancelReq() {
  if (typeof cancel === 'function'){
    cancel('强制取消请求')
  } else {
    console.log('没有可取消的请求')
  }
}

```
