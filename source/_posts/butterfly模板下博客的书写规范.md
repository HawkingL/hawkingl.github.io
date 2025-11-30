---
title: butterfly模板下博客的书写规范
abbrlink: 46fbf4ba
date: 2021-11-16 21:39:06
keywords: butterfly主题
categories: 博客
tags: butterfly的基本配置
description: hexo插件中butterfly主题的基本配置，文章内容的书写规范
top_img: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-4cb04e1926d95725cdb3cfc3c07f692c_1440w.jpg
cover: https://hawking-img.oss-cn-hangzhou.aliyuncs.com/BlogPic/v2-591e6208933e9df2442ef55251ccdd9b_1440w.jpg
---

## 博客页面配置

### 文章页面配置

```md
---

title: 【必需】文章标题
date: 【必需】文章创建日期
updated: 【可选】文章更新日期
tags: 【可选】文章标签
categories: 【可选】文章分类
keywords: 【可选】文章关键词
description: 【可选】文章描述
top_img: 【可选】文章顶部图片
cover: 【可选】文章封面（如果没有设置 top_img，文章页顶部将显示缩略图，可设为 false/图片地址/留空）
sticky: 【可选】置顶操作，后跟数子，数字越大权重越高
comments: 【可选】显示文章评论模块（默认 true）
toc: 【可选】显示文章 TOC（默认为设置中 toc 的 enable 配置）
toc_number: 【可选】显示 toc_number（默认为设置中 toc 的 number 配置）
copyright: 【可选】显示文章版权模块（默认为设置中 post_copyright 的 enable 配置）
copyright_author: 【可选】文章版权模块的文章作者
copyright_author_href: 【可选】文章版权模块的链接文章作者
copyright_url: 【可选】文章版权模块的链接文章连结
copyright_info: 【可选】文章版权模块的文字版权声明
mathjax: 【可选】显示 mathjax（当设置 mathjax 的 per_page： false 时，才需要配置，默认 false）
katex: 【可选】显示 katex（当设置 katex 的 per_page： false 时，才需要配置，默认 false）
aplayer: 【可选】在需要的页面加载 aplayer 的 js 和 css，请参考文章下面的 配置音乐
highlight_shrink: 【可选】配置代码框是否展开（true/false）（默认为设置中 highlight_shrink 的配置）
aside: 【可选】显示侧边栏 （默认 true）

---
```

## 分割线

文章内容以“--- 博客内容 ---”进行包裹

```md
---

内容 1

---

内容 2

---
```

## 标签页

前往你的 Hexo 博客的根目录

找到这个文件： source/tags/index.md

修改文件为;

```md
--- MARKDOWN
title: 标签页面
date: 2018-01-05 00:00:00
type: "tags"
top_img: 【可选】文章顶部图片

---
```

### 分类页

前往你的 Hexo 博客的根目录

找到这个文件：source/categories/index.md

修改此文档 为：

MARKDOWN

```md
---
title: 分类
date: 2018-01-05 00:00:00
type: "categories"
top_img: 【可选】文章顶部图片
---
```

## 链接页面

前往你的 Hexo 博客的根目录

找到这个文件：source/link/index.md

```
---
title: 链接
date: 2018-06-07 22:17:49
type: "link"
---
```

### 链接的添加配置

在 Hexo 博客目录中的（如果没有 \_data 文件夹，请自行创建），在 source 下创建一个\_data 文件夹，在此文件夹下创建一个 link.yml 文件，对文件进行配置

```
- class_name: 友情链接
  class_desc: 那些人，那些事  #页面标题
  link_list:
    - name: Hexo
      link: https://hexo.io/zh-tw/
      avatar: https://d33wubrfki0l68.cloudfront.net/6657ba50e702d84afb32fe846bed54fba1a77add/827ae/logo.svg
      descr: 快速、简单强大

- class_name: 油管
  class_desc: 值得推荐
  link_list:
    - name: Youtube
      link: https://www.youtube.com/
      avatar: https://i.loli.net/2020/05/14/9ZkGg8v3azHJfM1.png
      descr: 视频网站
    - name: Weibo
      link: https://www.weibo.com/
      avatar: https://i.loli.net/2020/05/14/TLJBum386vcnI1P.png
      descr: 最大社交分享平台
    - name: Twitter
      link: https://twitter.com/
      avatar: https://i.loli.net/2020/05/14/5VyHPQqR6LWF39a.png
      descr: 社交分享平台
```
