---
title: Nestjs
tags: Nestjs
categories: 小程序
keywords: 微信小程序中npm的使用
descriptions: >-
  由于小程序只识别miniprogram_npm文件中的包，所有需要将下载好的npm包转换成小程序能够识别的包，找到工具中的构建npm点击，微信开发者工具会自动将下载到node_modules中的包转移到miniprogram_npm中，并将起转换成小程序能够识别的包。
top_img: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-f3d09f2930406c4c186d8fcf4e4d612d_1440w.jpg"
cover: "https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-3d7a580c5009e9e61e3cab628d497316_1440w.jpg"
abbrlink: a92e5ed7
date: 2023-02-24 23:20:58
---

## 设计模式

### IOC 控制反转

Inversion of Control 字面意思是[控制反转](https://so.csdn.net/so/search?q=%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC&spm=1001.2101.3001.7020)，具体定义是高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。

### IO 依赖的注入

依赖注入（Dependency Injection）其实和 IoC 是同根生，这两个原本就是一个东西，只不过由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以 2004 年大师级人物 Martin Fowler 又给出了一个新的名字：“依赖注入”。 类 A 依赖类 B 的常规表现是在 A 中使用 B 的 instance。

```TypeScript
class A {
    name: string
    constructor(name: string) {
        this.name = name
    }
}


class B {
    age:number
    entity:A
    constructor (age:number) {
        this.age = age;
        this.entity = new A('小满')
    }
}

const c = new B(18)

c.entity.name
```

**我们可以看到，B  中代码的实现是需要依赖  A  的，两者的代码耦合度非常高。当两者之间的业务逻辑复杂程度增加的情况下，维护成本与代码可读性都会随着增加，并且很难再多引入额外的模块进行功能拓展。**

```TypeScript
class A {
    name: string
    constructor(name: string) {
        this.name = name
    }
}


class C {
    name: string
    constructor(name: string) {
        this.name = name
    }
}
//中间件用于解耦
class Container {
    modeuls: any
    constructor() {
        this.modeuls = {}
    }
    provide(key: string, modeuls: any) {
        this.modeuls[key] = modeuls
    }
    get(key) {
        return this.modeuls[key]
    }
}

const mo = new Container()
mo.provide('a', new A('小满1'))
mo.provide('c', new C('小满2'))

class B {
    a: any
    c: any
    constructor(container: Container) {
        this.a = container.get('a')
        this.c = container.get('c')
    }
}

new B(mo)
```

其实就是写了一个中间件，来收集依赖，主要是为了解耦，减少维护成本

## 创建 nestjs 项目

安装 nestjs

`npm i -g @nestjs/cli`

创建项目

`nest new 项目名称`

启动项目

`npm run start:dev`

## 目录介绍

### main.js

入口文件类似于 vue 的 main.js，通过 NestFactory.create(AppModule)创建一个 app 就是类似绑定一个根组件 App.vue

app.listen(300) 监听一个端口

### Controller.ts 控制器

类似于 Vue 的路由，private readonly appService:AppServeice 这一行代码就是依赖注入不需要实例化，appService 它内部会自己实例化的我们主要需要放上去就可以了

### app.service.ts

这个文件主要实现业务逻辑

## nest cli 常用命令

nest --help 可以查看 nestjs 所有的命令

1.生成 chontroller.ts

`nest g co user`

2.生成 modulee.ts

`nest g mo user`

3.生成 servuce.ts

`nest g s user`

3.以上步骤一个一个生成的太慢了我们可以直接使用一个命令生成 CURD

`nest g resource 名称`

> 第一次使用这个命令的时候，除了生成文件之外还会自动使用  `npm`  帮我们更新资源，安装一些额外的插件，后续再次使用就不会更新了。

## RESTful 风格设计

### 传统接口

http://localhost:8080/api/get_list?id=1

http://localhost:8080/api/delete_list?id=1

http://localhost:8080/api/update_list?id=1

> 通过在 url 上拼接不同的字段，去访问不同的端口就，例如 get_list 就是获取文章，而 delete_list 就是删除文章，updata_list 就是更新文章。

### RESTful 接口

http://localhost:8080/api/get_list/1 查询 删除 更新

> RESTful 风格的接口只需一个 url 就会完成 增删改差 他是通过不同的请求方式来区分的

查询 GET
提交 POST
更新 PUT PATCH
删除 DELETE

### RESTful 版本控制  

一共有三种我们一般用第一种 更加语义化

|                         |                                   |
| ----------------------- | --------------------------------- |
| `URI Versioning`        | 版本将在请求的 URI 中传递（默认） |
| `Header Versioning`     | 自定义请求标头将指定版本          |
| `Media Type Versioning` | 请求的`Accept`标头将指定版本      |

```TypeScript
import { NestFactory } from '@nestjs/core';
import { VersioningType } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableVersioning({
    type: VersioningType.URI,
  })
  await app.listen(3000);
}
bootstrap();
```

然后在 user.controller 配置版本

Controller 变成一个对象 通过 version 配置版本

```TypeScript
import { Controller, Get, Post, Body, Patch, Param, Delete, Version } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller({
  path:"user",
  version:'1'
})
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }

  @Get()
  // @Version('1')
  findAll() {
    return this.userService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(+id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.userService.update(+id, updateUserDto);
  }
```

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/1image.png)

### Code 码规范

- 200 OK

- 304 Not Modified 协商缓存了

- 400 Bad Request 参数错误

- 401 Unauthorized token 错误

- 403 Forbidden referer origin 验证失败

- 404 Not Found 接口不存在

- 500 Internal Server Error 服务端错误

- 502 Bad Gateway 上游接口有问题或者服务器问题

## nest 控制器

> 通过装饰器获取前端传过来的值

@Request()
@Response()
@Next()
@Session()
@Param(key?: string)  
@Body(key?: string)
@Query(key?: string)
@Headers(name?: string)

### 获取 get 请求传参

可以使用 Request 装饰器 或者 Query 装饰器 跟 express 完全一样

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/image.png)

也可以使用 Query 直接获取 不需要在通过 req.query 了

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/2image.png)

```TypeScript
import { Controller, Get, Post, Body, Patch, Param, Delete, Version, Request, Query } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) { }
  @Get()
  find(@Query() query) {
  console.log(query)
  return { code: 200 }
  }
}
```

### post 获取参数

可以使用 Request 装饰器 或者 Body 装饰器 跟 express 完全一样

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/3image.png)

或者直接使用 Body 装饰器

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/4image.png)

也可以直接读取 key

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/5image.png)

```TypeScript
import { Controller, Get, Post, Body, Patch, Param, Delete, Version, Request, Query } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) { }

  @Get()
  find(@Query() query) {
    console.log(query)
    return { code: 200 }
  }

  @Post()
  create (@Body() body) {

    console.log(body)

    return {
       code:200
    }
  }
}
```

### 动态路由

可以使用 Request 装饰器 或者 Param 装饰器 跟 express 完全一样

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/6image.png)

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/7image.png)

```TypeScript
import { Controller, Get, Post, Body, Patch, Param, Delete, Version, Request, Query } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) { }

  @Get()
  find(@Query() query) {
    console.log(query)
    return { code: 200 }
  }

  @Post()
  create (@Body('name') body) {

    console.log(body)

    return {
       code:200
    }
  }

  @Get(':id')
  findId (@Param() param) {

    console.log(param)

    return {
       code:200
    }
  }
}
```

### 读取 header 信息

我在调试工具随便加了一个 cookie

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/8image.png)

```TypeScript
import { Controller, Get, Post, Body, Patch, Param, Delete, Version, Request, Query, Ip, Header, Headers } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) { }

  @Get()
  find(@Query() query) {
    console.log(query)
    return { code: 200 }
  }

  @Post()
  create (@Body('name') body) {

    console.log(body)

    return {
       code:200
    }
  }

  @Get(':id')
  findId (@Headers() header) {

    console.log(header)

    return {
       code:200
    }
  }
}
```

### 状态码

使用 HttpCode 装饰器 控制接口返回的状态码

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/9image.png)

```TypeScript
import { Controller, Get, Post, Body, Patch, Param, Delete, Version, Request, Query, Ip, Header, Headers, HttpCode } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('user')
export class UserController {
  constructor(private readonly userService: UserService) { }

  @Get()
  find(@Query() query) {
    console.log(query)
    return { code: 200 }
  }

  @Post()
  create (@Body('name') body) {

    console.log(body)

    return {
       code:200
    }
  }

  @Get(':id')
  @HttpCode(500)
  findId (@Headers() header) {
    return {
       code:500
    }
  }
}
```

## nest session

session  是服务器 为每个用户的浏览器创建的一个会话对象 这个 session 会记录到 浏览器的 cookie 用来区分用户

我们使用的是 nestjs 默认框架 express 他也支持 express 的插件 所以我们就可以安装 express 的 session

`npm i express-session --save`

安装声明依赖

`npm i @types/express-session -D`

然后在 main.ts 引入 通过 app.use 注册 session

`import * as session from 'express-session'`

`app.use(session())`

### 参数配置详情

|                                                |                                                                                                 |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| secret                                         | 生成服务端 session 签名 可以理解为加密                                                          |
| name                                           | 生成客户端 cookie 的名字 默认  connect.sid                                                      |
| cookie                                         | 设置返回到前端 key 的属性，默认值为{ path: ‘/’, httpOnly: true, secure: false, maxAge: null }。 |
| rolling                                        | 在每次请求时强行设置 cookie，这将重置 cookie 过期时间(默认:false)                               |

### nestjs 的配置

```TypeScript
import { NestFactory } from '@nestjs/core';
import { VersioningType } from '@nestjs/common';
import { AppModule } from './app.module';
import * as session from 'express-session'
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableVersioning({
    type: VersioningType.URI
  })
  app.use(session({ secret: "XiaoMan", name: "xm.session", rolling: true, cookie: { maxAge: null } }))
  await app.listen(3000);
}
bootstrap();
```

### 验证码案例

#### 后端

1.创建 nestjs 项目作为后端

`nest new nestjs后端`

2.生成 CURD

`nest g recoure yzh`

3.安转 expresss-session 及其 TS 声明，并设置 cookie

`npm i express-session --save`

`npm i @types/express-session -D`

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/10image.png)

4.安装 svgCaptcha 生成验证码

`npm i svg-captcha --save`

5.创建接口

业务接口

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/11image.png)

控制接口

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/12image.png)

#### 前端

1.创建一个 vue3 项目

`npm init vue@latest`

2.下载 elementPuls

`npm i element-puls --save`

```TypeScript
// main.ts
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

const app = createApp(App)

app.use(ElementPlus)
app.mount('#app')
```

3.使用 elementPuls 生成一个登录表单

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/13image.png)

4.登录页面逻辑代码的编写

```Vue
<template>
  <el-radio-group v-model="labelPosition" label="label position">
    <el-radio-button label="left">Left</el-radio-button>
    <el-radio-button label="right">Right</el-radio-button>
    <el-radio-button label="top">Top</el-radio-button>
  </el-radio-group>
  <div style="margin: 20px" />
  <el-form
    :label-position="labelPosition"
    label-width="100px"
    :model="formLabelAlign"
    style="max-width: 460px"
  >
    <el-form-item label="账户">
      <el-input v-model="formLabelAlign.name" />
    </el-form-item>
    <el-form-item label="密码">
      <el-input v-model="formLabelAlign.password" />
    </el-form-item>
    <el-form-item label="验证码">
      <div style="display: flex">
        <el-input v-model="formLabelAlign.code"></el-input>
        <img @click='resetCode' :src='codeUrl'>
      </div>
    </el-form-item>
    <el-form-item>
      <el-button @click="submit">登录</el-button>
    </el-form-item>
  </el-form>
</template>

<script lang="ts" setup>
import { reactive, ref } from 'vue'
//请求的地址
const codeUrl = ref<string>('/api/yzh/code')
//点击图片实现刷新验证码
const resetCode = () => codeUrl.value = codeUrl.value + '?' + Math.random();

//表单内容格式
const labelPosition = ref('top')

//收集表单信息
const formLabelAlign = reactive({
  name: '',
  password: '',
  code: '',
})

//提交表单
const submit = () => {
  fetch('api/yzh/create', {
    method: "post",
    body: JSON.stringify(formLabelAlign),
    headers: {
      'content-type': 'application/json'
    }
  }).then(res => res.json()).then(res => {
    console.log(res);

  })
}
</script>
```

5.配置代理

![image.png](https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/14image.png)
