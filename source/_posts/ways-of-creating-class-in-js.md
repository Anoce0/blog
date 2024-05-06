---
title: Javascript 创建对象的几种方式
date: 2021-05-24 23:08:07
tags:
    - javascript
    - basic
---

在经典的面向对象语言中，对象是指数据和在这些数据上进行的操作的集合。与 C++ 和 Java 不同，JavaScript 是一种基于原型的编程语言，并没有 class 语句，而是把函数用作类, 定义对象的方式与其他语言有很多不同.

### 1. 通过方法创造并返回对象

例如,我们希望得到一个以下的数据和操作的集合:

```javascript
function makePerson(first, last) {
    return {
        first: first,
        last: last,
    };
}
function personFullName(person) {
    return person.first + ' ' + person.last;
}
function personFullNameReversed(person) {
    return person.last + ', ' + person.first;
}
var s = makePerson('Simon', 'Willison');
personFullName(s); // "Simon Willison"
personFullNameReversed(s); // "Willison, Simon"
```

上述方法定义到一个方法中可以变成一下这个样子

```javascript
function makePerson(first, last) {
    return {
        first: first,
        last: last,
        fullName: function () {
            return this.first + ' ' + this.last;
        },
        fullNameReversed: function () {
            return this.last + ', ' + this.first;
        },
    };
}
let person = makePerson('Ocean', 'Wang');
person.fullName(); // Ocean Wang
person.fullNameReversed(); // Wang Ocean
```

### 2. 使用 this 与 new 关键字

上述写法使用了 `this` 关键字, JS 中的 `this` 关键字与 Java 中的 `this` 永远指向当前实例不同, JS 中的 `this` 是由当前的调用环境决定的.  
当使用在函数中时，this 指代当前的对象，也就是调用了函数的对象。如果在一个对象上使用点或者方括号来访问属性或方法，这个对象就成了 this。如果并没有使用“点”运算符调用某个对象，那么 this 将指向全局对象（global object）。这是一个经常出错的地方。例如：

```javascript
let person = makePerson('Ocean', 'Wang');
let fullName = person.fullName;
fullName(); // undefined undefined
```

通过利用 `this` 指代当前对象的特点, 我们可以将上述类定义成以下这个样子

```javascript
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = function () {
        return this.first + ' ' + this.last;
    };
    this.fullNameReversed = function () {
        return this.last + ', ' + this.first;
    };
}
```

我们注意到,这个函数是没有返回值的, 如果我们试图调用这个函数, 实际返回的是 `undefined`

```javascript
Person(); // undefined
```

那如果我们将 `this` 返回会怎么样呢? 就像我们刚刚说的,this 指代当前的对象，在下面的调用中, this 实际指向的是全局对象（global object）, 实际上下面代码做的事情是 将 last,first, fullName 绑定到了全局对象中,并且返回了全局对象.

```javascript
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = function () {
        return this.first + ' ' + this.last;
    };
    this.fullNameReversed = function () {
        return this.last + ', ' + this.first;
    };
    return this;
}
Person(); // undefined
window.fullName; // function
```

好在为了解决这种问题, javascript 提供了 `new` 关键字,帮我们正确的从函数初始化一个对象,正确绑定并返回 `this`

```javascript
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = function () {
        return this.first + ' ' + this.last;
    };
    this.fullNameReversed = function () {
        return this.last + ', ' + this.first;
    };
}
new Person('Ocean', 'Wang'); // person object
```

`new` 关键字在运行是会生成一个新的对象, 并且将 `this` 绑定到这个对象, 然后返回它, 可以用一下的代码来模拟:

```javascript
function trivialNew(constructor, ...args) {
    var o = {}; // 创建一个对象
    constructor.apply(o, args);
    return o;
}
var persion = trivialNew(Person, 'Ocean', 'Wang');
```

> 这并不是 new 的完整实现，因为它没有创建原型（prototype）链。想举例说明 new 的实现有些困难，因为你不会经常用到这个，但是适当了解一下还是很有用的。

### 3. 使用 (prototype) 原型链

我们回过头来看这个定义类和初始化对象的方法,我们知道在 javascript 中, 定义 `function` 实际是生成了一个新的 `function` 对象, 所以每次 `new Person()` 时都会构造两个新的对象而这些对象的功能都是类似的, 有办法复用吗? 比如这样

```javascript
function personFullName() {
    return this.first + ' ' + this.last;
}
function personFullNameReversed() {
    return this.last + ', ' + this.first;
}
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = personFullName;
    this.fullNameReversed = personFullNameReversed;
}
```

这种写法的好处是，我们只需要创建一次方法函数，在构造函数中引用它们, 那是否还有更好的方法呢？答案是肯定的。

```javascript
function Person(first, last) {
    this.first = first;
    this.last = last;
}
Person.prototype.fullName = function () {
    return this.first + ' ' + this.last;
};
Person.prototype.fullNameReversed = function () {
    return this.last + ', ' + this.first;
};
```

Person.prototype 是一个可以被 Person 的所有实例共享的对象。它是一个名叫原型链（prototype chain）的查询链的一部分：当你试图访问 Person 某个实例（例如上个例子中的 s）一个没有定义的属性时，解释器会首先检查这个 Person.prototype 来判断是否存在这样一个属性。所以，任何分配给 Person.prototype 的东西对通过 this 对象构造的实例都是可用的。

这个特性功能十分强大，JavaScript 允许你在程序中的任何时候修改原型（prototype）中的一些东西，也就是说你可以在运行时(runtime)给已存在的对象添加额外的方法：

```javascript
s = new Person('Simon', 'Willison');
s.firstNameCaps(); // TypeError on line 1: s.firstNameCaps is not a function

Person.prototype.firstNameCaps = function () {
    return this.first.toUpperCase();
};
s.firstNameCaps(); // SIMON
```

#### 扩展

以上我们简单聊了下 Javascript 创建对象的方法,和其中的原理, 由于 JS 是基于原型链的特性,JS 的继承机制也与其他面向对象的语言有很多不同,具体可以参考这个文档: [MDN - 继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

> Reference:
>
> - _[MDN - A_re-introduction_to_JavaScript#自定义对象](hhttps://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E8%87%AA%E5%AE%9A%E4%B9%89%E5%AF%B9%E8%B1%A1)_
