---
title: 怎么在 `Hexo` 中创建一篇新的文章
tags: hexo create-post
date: 2019-05-17 17:36:10
---

## 怎么在 `Hexo` 中创建一篇新的文章

### 创建一篇新的文章

创建一篇新的文章需要使用 `hexo` 命令， `hexo` 命令有两种安装方式

#### 通过 `npm` 全局安装
   1. `npm instal -g hexo`
   2. `hexo new [layout] <title>`

Arguments:
```
  layout  Post layout. Use post, page, draft or whatever you want.
  title   Post title. Wrap it with quotations to escape.
```

#### 将 `hexo` 安装到当前目录下

通常我们在项目中执行 `npm install` 后 `hexo` 就已经安装到了`./node_modules/` 中，我们可以通过路径 `./node_modules/hexo/bin/hexo` 来使用, 为了方便使用，我们可以将命令写到 `package.json` 中
```
  "scripts": {
    "hexo": "./node_modules/hexo/bin/hexo",
    "create" : "./node_modules/hexo/bin/hexo new"
  },
```
这是可以使用 `npm run create -- [layout] <title>` 来创建新的文章

创建完成后会发现 \<title\>.md 创建在 source/_\<layout\> 目录下面
