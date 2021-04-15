---
title: babel 配置 polyfill 几种方式
tags: 
  - javascript 
  - buildtool
---

corejs 升级到 version 3 之后, babel 中针对 corejs polyfill 有几种不同的配置, 主要使用@babel/preset-env 的 useBuiltIns 这个参数

1. useBuiltIns: entry with corejs: 3
这种方式,要求开发者讲一下代码显式添加到项目的入口处
```javascript
import "core-js/stable";
import "regenerator-runtime/runtime";
```
babel 会将这段代码替换为目标环境需要的特定 polyfill, 例如,在 targeting 到 chrome 72 时,它会被替换为
```javascript
import "core-js/modules/es.array.unscopables.flat";
import "core-js/modules/es.array.unscopables.flat-map";
import "core-js/modules/es.object.from-entries";
import "core-js/modules/web.immediate";
```