---
title: 怎么在 Hexo 中创建一篇新的文章
tags: [hexo]
date: 2019-05-17 17:36:10
---

### 安装 hexo 命令

创建一篇新的文章需要使用 `hexo` 命令， `hexo` 命令有两种安装方式

#### 通过 `npm` 全局安装

   1. `npm instal -g hexo`
   2. `hexo new [layout] <title>`

Arguments:

```none
  layout  Post layout. Use post, page, draft or whatever you want.
  title   Post title. Wrap it with quotations to escape.
```

#### 将 `hexo` 安装到当前目录下

通常我们在项目中执行 `npm install` 后 `hexo` 就已经安装到了`./node_modules/` 中，我们可以通过路径 `./node_modules/hexo/bin/hexo` 来使用, 为了方便使用，我们可以将命令写到 `package.json` 中

```javascript
  "scripts": {
    "hexo": "./node_modules/hexo/bin/hexo",
    "create" : "./node_modules/hexo/bin/hexo new"
  },
```

### 创建新的文章

通常如果要发布一个新的文章，可以通过如下步骤

1. 使用 `hexo new draft <title>` 来生成一个 draft
2. 编辑 draft, 通过 `hexo server --draft` 来预览
3. 编辑完成之后使用 `hexo publish post <title>` 来将 draft 发布到 post 下
4. 使用 `hexo clean && hexo deplpy` 来发布最新的页面 (通常这个新的页面可能需要等几分钟才能生效)

详细的说明可以可以在[这里](https://hexo.io/docs/writing)看到具体的官方文档

### 关于 markdown 语法

- 语法速查 - [Markdown 指南中文版](https://www.markdown.xyz/basic-syntax)
