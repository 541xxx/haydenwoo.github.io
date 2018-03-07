---
layout:     post
title:      "模拟实现call && apply"
subtitle:   "simulate call and apply"
date:       2018-03-06 12:00:00 
author:     "Hayden"
header-img: "img/in-post/post-call-apply/ghetto-streets-on-acid.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 前端开发
    - JavaScript
---

> 随眼一看很多公司出的题目都会有「如何改变this上下文」此类，当然答案大家都已知晓，但不知其内部实现原理，故模拟之

 其实这是一篇转载，原文地址是 [https://github.com/mqyqingfeng/Blog/issues/11](https://github.com/mqyqingfeng/Blog/issues/11) <br>

 安利一下这个博客，涵盖了大多数的js基础以及实现原理，会让你发现不止是面试的时候有用哟~


## call 
***
一句话介绍call
> call() 方法在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。

举个例子：

```js
var foo = {
    value: 1
};

function bar () {
    console.log(this.value);
}

bar.call(foo); // 1
```
注意两点：
1. call 改变了 this 的指向，指向到 foo
2. bar 函数执行了

**模拟实现第一步**

***
那么我们该怎么模拟实现这两个效果呢？
试想当调用 call 的时候，把 foo 对象改造成如下：

```js
var foo  = {
    value: 1,
    bar: function() {
        console.log(this.value);
    }
}

foo.bar(); // 1
```
这个时候**this**就指向了**foo**，是不是很简单呢？

但是这样却给 foo 对象本身添加了一个属性，这可不行呐！

不过也不用担心，我们用 delete 再删除它不就好了~

所以我们模拟的步骤可以分为：

1. 将函数设为对象的属性
2. 执行该函数
3. 删除该函数

以上个例子为例，就是：

```js
// 第一步
foo.fn = bar
// 第二步
foo.fn()
// 第三步
delete foo.fn
```

fn 是对象的属性名，反正最后也要删除它，所以起成什么都无所谓。

根据这个思路，我们可以尝试着去写第一版的 call2 函数：

```js
// 第一版
Function.prototype.call2 = function(context) {
    // 首先要获取调用call的函数，用this可以获取
    context.fn = this;
    context.fn();
    delete context.fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

bar.call2(foo); // 1
```

**模拟实现第二步**

***
从一开始也讲了，call 函数还能给定参数执行函数。举个例子：

```js
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call(foo, 'kevin', 18);
// kevin
// 18
// 1
```

注意：传入的参数并不确定，这可咋办？

不急，我们可以从 Arguments 对象中取值，取出第二个到最后一个参数，然后放到一个数组里。

比如这样：

```js
// 以上个例子为例，此时的arguments为：
// arguments = {
//      0: foo,
//      1: 'kevin',
//      2: 18,
//      length: 3
// }
// 因为arguments是类数组对象，所以可以用for循环
var args = [];
for(var i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']');
}

    // 执行后 args为 ["arguments[1]", "arguments[2]", "arguments[3]"]
```

不定长的参数问题解决了，我们接着要把这个参数数组放到要执行的函数的参数里面去。

也许有人想到用 ES6 的方法，不过 call 是 ES3 的方法，我们为了模拟实现一个 ES3 的方法，要用到ES6的方法，好像……，嗯，也可以啦。但是我们这次用 eval 方法拼成一个函数，类似于这样：

```js
 eval('context.fn(' + args +')')
```
这里说一下eval的用法  [eval()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eva)

> eval() 函数可计算某个字符串，并执行其中的的 JavaScript 代码，简而言之就是可以执行字符串里的方法 比如 eval('(function() {console.log(1)})()')     // 1

这里 args 会自动调用 Array.toString() 这个方法。

所以我们的第二版克服了两个大问题，代码如下：

```js
// 第二版
Function.prototype.call2 = function(context) {
    context.fn = this;
    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    eval('context.fn(' + args +')');
    delete context.fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call2(foo, 'kevin', 18); 
// kevin
// 18
// 1
```

**模拟实现第三步**

***
模拟代码已经完成 80%，还有两个小点要注意：

**1.this 参数可以传 null，当为 null 的时候，视为指向 window**

举个例子：

```js
var value = 1;

function bar() {
    console.log(this.value);
}

bar.call(null); // 1 这里用的是原生的call
```
**2.函数是可以有返回值的**

举个例子：

```js
var obj = {
    value: 1
}

function bar(name, age) {
    return {
        value: this.value,
        name: name,
        age: age
    }
}

console.log(bar.call(obj, 'kevin', 18));
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }
```
不过都很好解决，让我们直接看第三版也就是最后一版的代码：

```js
// 第三版
Function.prototype.call2 = function (context) {
    var context = context || window;
    context.fn = this;

    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }

    var result = eval('context.fn(' + args +')');

    delete context.fn
    return result;
}

// 测试一下
var value = 2;

var obj = {
    value: 1
}

function bar(name, age) {
    console.log(this.value);
    return {
        value: this.value,
        name: name,
        age: age
    }
}

bar.call2(null); // 2

console.log(bar.call2(obj, 'kevin', 18));
// 1
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }

```

到此，我们完成了**call**的模拟实现

最后再来用我们精(yu)简(fa)的(tang)ES6来实现一下

```js
Function.prototype.call2 = function(context, ...args) {
    var context = context || window;
    context.fn = this;
    var result = context.fn(...args);
    delete context.fn;
    return result;
}
```

## apply的模拟实现

apply 的实现跟 call 类似，在这里直接给代码，代码来自于知乎 @郑航的实现：

```js
Function.prototype.apply = function (context, arr) {
    var context = Object(context) || window; // 和上面代码比较这种写法比较严格
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    }
    else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
}
```




