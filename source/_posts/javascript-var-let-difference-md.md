---
title: Javascript 中的变量声明 - let, const, var
date: 2021-03-17 23:08:07
tags: [javascript, basic]
---

### 开始

Javascript 支持的变量声明方式其实有四种 `var` `let` `const` `隐式创建`

### 一 . 隐式创建变量

当赋值给未声明的变量, 则执行赋值后, 该变量会被隐式地创建为全局变量（它将成为全局对象的属性）。
```
function x() {
  y = 1;   // 在严格模式（strict mode）下会抛出 ReferenceError 异常
}
x();
console.log(y); // 打印 "1"
```

### 二. 使用 var 创建变量
var 是 ES6 发布之前仅存的一种变量声明方式，用 var 声明变量有一下几个特征

#### 1. 作用域

var 的作用域只有两种 函数级别作用域 和 全局作用域 (let const 可以声明块作用域，这个我们在后面会谈到)

 ```
// 全局作用域
var a  = 1;

// 块作用域不会生效
if (true) {
  var b = true; // 使用 "var" 而不是 "let"
}
alert(b); // true，变量在 if 结束后仍存在

// 函数作用域
function sayHi() {
  if (true) {
    var c = "Hello";
  }
  alert(c); // 能正常工作
  alert(a); // 正常工作
}
sayHi();
alert(c); // Error: phrase is not defined
 ```

#### 2. 重复声明

var 可以被声明多次, 新的声明会被忽略

```
var user = "Pete";

var user = "John"; // 这个 "var" 无效（因为变量已经声明过了), 这里就是简单的赋值操作
// ……不会触发错误

alert(user); // John
```

#### 3. 声明提升 

“var” 声明的变量，可以在其声明语句前被使用
当函数开始的时候，就会处理 var 声明（脚本启动对应全局变量）。
换言之，var 声明的变量会在函数开头被定义，与它在代码中定义的位置无关（这里不考虑定义在嵌套函数中的情况）。

例如
```
function sayHi() {
  a = "Hello";
  alert(a);
  var a;
}
sayHi();
```
与下面这种写法本质是一样的
```
function sayHi() {
  var a;
  a = "Hello";
  alert(a);
}
sayHi();
```
甚至于这种都是一样的 (由于 var 只有函数级的作用域，所以代码块是会被忽略的）)
```
function sayHi() {
  a = "Hello"; 
  if (false) {
    var a;
  }
  alert(a);
}
sayHi();
```
#### 4. 全局声明

var 在全局声明是会绑定成全局属性

```
var a = 1;
this.a // 1
```

> ### IIFE
> 在 ES6 发布之前，Javascript 只有 var 这一种声明方式，并且这种声明方式没有块级作用域，开发者发明了一种模仿块级作用域的方法： “立即调用函数表达式”（immediately-invoked function expressions，IIFE）。
> 如今，在有 let, const 声明变量的方式， IIFE 已经不再被使用了，但是我们经常还是可以再一些老的 code 中看到它 
>
> ```
> (function() {
>  var message = "Hello";
>  alert(message); // Hello
> })();
> alert(message) // Uncaught ReferenceError: message is not defined
> ```
> -------


### 三. 使用 let 创建变量
let 是 ES6 中被引进的一种变量定义方式，它与 var 声明变量主要有这几个区别
 - 增加了块级作用域
 - 消除变量提升作用（暂存死区TDZ(Temporal Dead Zone)）
 - 在一个作用域中不可重复声明
------
#### 1. 块级作用域

```
let a = 0;
if(true){
    let a = 1;
    console.log(a) // 1
}
console.log() // 2
```

#### 2. 暂存死区  

不同于 var, 所有的变量声明会被提升到作用域的开头位置，在声明前引用是合法的，let 的声明只有在执行到声明那一刻才会正式生效。  

在程序的控制流程在新的作用域进行实例化时，在此作用域中用let、const声明的变量会在该作用域中先创建，但这个时候还没有进行词法绑定，没有进行对声明语句的赋值运算，所以是不能访问的，访问会抛出错误。所以在这运行流程一进入作用域创建变量，到变量开始可被访问的一段时间，就称为TDZ。
```
console.log(a) // undefined
var a = 0;

console.log(b) // undefined, 这个时候是隐式声明， 在严格模式（strict mode）下会抛出 ReferenceError 异常
b = 0; 

console.log(b) // Uncaught ReferenceError：a is not defined
let b = 0
```

这个暂存死区，放到词法作用域/静态作用域中会产生一个有趣的现象

```
function test(){
let foo = 33;
if (foo) {
    let foo = (foo + 55); // Uncaught ReferenceError
}
}
test();
```
这是因为在进入 if 代码块之后，程序识别到一个新的 `foo` 已经被创建了，而且在被正式赋值之前，这个 `foo` 是不可被引用的， `(foo + 55)` 中 `foo` 并不能引用到外层词法作用域中的 `let foo = 33;`

#### 3. 非全局属性

let不会在全局声明时（在最顶部的范围）创建 window/this 对象的属性。

```
var a = 1;
console.log(window.a); // 1
let b = 2; 
console.log(window.b); // undefined
```

### 四. 使用 const 创建变量

`const` 创建变量与 `let` 基本是一致的，只是 `const` 不能被重新赋值, 而且必须在创建的同时给它初始值.  

```  
const a = 1;
a = 2; // Uncaught TypeError: Assignment to constant variable.
cosnt b; // Uncaught SyntaxError: Missing initializer in const declaration
```

这里要注意的是，不能被赋值的是 `const` 声明的变量这个引用本身，但是对引用的引用是不受影响的：
```
const foo = {
    bar: 1
}
foo.bar = 2 // ok
foo = 2; // Uncaught TypeError

const arr = [1]
arr[0] = 2 // ok
arr = [2] // Uncaught TypeError
```

> 参考:  
> - *[旧时的 var](https://zh.javascript.info/var)*  
> - *[MDN Web Doc - let](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)*  
> - *[MDN Web Doc - var](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/var)*