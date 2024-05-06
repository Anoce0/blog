---
title: angular 中的 ngc 命令
date: 2021-06-08 19:21:15
tags:
    - javascript
    - angular
    - buildtool
---


## 关于 ngc

ngc ,全称是 [Angular Template Compiler](https://stackoverflow.com/a/51893626) ,来源于包 `@angular/compiler-cli`.

## ngc 与 tsc

它在 angular 官网缺乏相关的文档, 有人在 angular repo 中提了个 [issue](https://github.com/angular/angular/issues/29623) 来讨论这个问题.
同时有个 [PR](https://github.com/iRealNirmal/angular/commit/748db836c696ab5ac66e1cefb85192740ec8db32?short_path=0a65f1d#diff-0a65f1d37c6a41d9b8c6e91e484e9e58bb815f22e9b3425432c086c1b904eefa) 正在在给 angular 添加 `ngc` 的 document  
摘录如下:  

> While most of the time you interact with the Angular Compiler indirectly using Angular CLI, when debugging certain issues, you might find it useful to invoke the Angular Compiler directly. You can use the ngc command provided by the @angular/compiler-cli npm package to call the compiler from the command line.
> The ngc command is just a wrapper around TypeScript's tsc compiler command and is primarily configured via the tsconfig.json configuration options documented in the previous sections.
> In addition to the configuration file, you can also use tsc command line options to configure ngc.  

这里说 `ngc` 是 `tsc` 的一个 Wrapper, 我的实际测试发现这两个命令编译出来的包大小并不想等等, ngc 会更大一点.

tsc and ngc have different purposes and it's not about selecting one over the other.

tsc is a TypeScript compiler, and you need it to generate JavaScript if your app is written in TypeScript.

ngc is an Angular-specific compiler. It doesn't turn the TypeScript code into JavaScript. It does a "finishing touch" to make your app bundles ready for rendering by the browser. In particular, it turns your components templates into inline JavaScript. If you do a prod build with Ahead of Time (AoT) compilation, the ngc does its part before the bundles are built. In dev mode we use Just-in-Time compilation: the templates are not precompiled, the ngc compiler is included into the bundles, and it compiles the templates after the browser loaded your bundles.

see [Angular: ngc or tsc?](https://stackoverflow.com/a/50111982)

## ngc 与 ng build

stackoverflow 有人提了这个[问题](https://stackoverflow.com/questions/44642696/whats-the-relationship-and-difference-between-ng-build-and-ngc)
简单解释:
Angular 提供了两种编译工具,JIT(Just in Time) and AOT (Ahead of Time compile) .
`ng` == JIT
`ngc` == AOT
