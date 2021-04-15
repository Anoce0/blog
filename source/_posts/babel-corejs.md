---
title: babel 配置 polyfill 几种方式
tags:
  - javascript
  - buildtool
date: 2021-04-15 20:27:06
---


corejs 升级到 version 3 之后, babel 中针对 corejs polyfill 有几种不同的配置, 主要使用@babel/preset-env 的 useBuiltIns 这个参数

## 1. useBuiltIns: entry with corejs: 3

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

## 2. useBuiltIns: usage with corejs: 3

这种方式,不需要开发者添加 `core-js` `regenerator-runtime` 到项目入口, babel 会自动的将目标文件需要的 polyfills 添加到项目中
例如,如果我们有如下代码

```javascript
const set = new Set([1, 2, 3]);
[1, 2, 3].includes(2);
```

在 ie11 中会被被自动替换成

```javascript
import "core-js/modules/es.array.includes";
import "core-js/modules/es.array.iterator";
import "core-js/modules/es.object.to-string";
import "core-js/modules/es.set";

const set = new Set([1, 2, 3]);
[1, 2, 3].includes(2);
```

它跟 `entry` 选项的主要区别在于

1.  `usage` 不需要在项目开头显式地添加 `import "core-js/stable";`
2.  `usage` 会在文件的作用域内添加被使用到的 polyfill, 而 `entry` 会在项目入口处添加
3.  `entry` 会根据目标环境(例如 ie11) 来添加所有的 polyfill, 而 `usage` 会检查被使用到的 polyfill

## 3. @babel/plugin-transform-runtime plugin

这个 plugin 可以使用 babel 的 注入 helper 来减少 codesize,通过是用这个 plugin, 可以代替使用 @babel/preset-env 的 useBuiltIns 的功能, 即 `"useBuiltIns": false,`,这个插件与使用 useBuiltIns 主要有两个差别

1. 使用 `useBuiltIns` 的 polyfill 会污染全局变量, 而 `plugin-transform-runtime` 不会, [这里](https://babeljs.io/docs/en/babel-plugin-transform-runtime#technical-details)可以找他关于相关的技术细节
2. 使用 `plugin-transform-runtime` 可以显著降低输出的文件体积, 所以一般建议在开发 sdk 是使用这个配置

> 参考:
>
> - _[Confused about useBuiltIns option of @babel/preset-env (using Browserslist Integration)](https://stackoverflow.com/q/52625979)_
> - _[core-js@3, babel and a look into the future](https://github.com/zloirock/core-js/blob/master/docs/2019-03-19-core-js-3-babel-and-a-look-into-the-future.md)_
