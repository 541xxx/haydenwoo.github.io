---
layout:     post
title:      "深入浅出apply、call"
subtitle:   "apply and call"
date:       2017-11-03 12:00:00 
author:     "Hayden"
header-img: "img/in-post/post-eleme-pwa/eleme-at-io.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 前端开发
    - JavaScript
---

> 网上关于这几个属性的文章真是数不胜数，以至于我每次使用都已经养成了习惯去查阅相关博客内容，几乎每次要用的时候对这几个东西的意识都会有点朦胧，所以还是整理一下吧

## apply、call

JavaScript函数存在「定义时上下文」和「运行时上下文」以及「上下文是可以改变的」这样的概念。

在javascript中，这两个属性都是为了改变函数内部this指向问题而存在的。

引用知乎上一个大牛的回答

在javascript OOP(面向对象) 中，我们总是会这样定义

```js
function cat () {
}
cat.prototype = {
  food: 'fish',
  say: function() {
    alert('I love' + this.food);
  }
}

var blackCat = new cat();
blackCat.say();

```

但是如果我们有一个对象whiteDog= {food : "bone"} ,我们不想对它重新定义 say 方法，那么我们可以通过 call 或 apply 用 cat 的 say 方法：


```js
  var whiteDog = {
    food: 'bone';
  }

  blackCat.say.call(whiteDog)
```

当一个object下没有某个方法，但是其他的有，我们就可以通过apply 和 call用其他对象的方法来操作，说说前段时间犯的一个小错误

```js
var childs = document.querySelectorAll('.item'); 
console.log(childs.length) // 假设长度是是3

childs.forEach(function(item) { //直接报错 TypeError: b.forEach is not a function
});

```
为什么会报错呢？当时太年轻还不知道有伪数组这一说，殊不知上面获取的看似是数组的玩意儿只有数组里面的length这一个属性，其他的比如push、 pop、shift 一概没有然后就有了下面的写法

```js
childs = Array.prototype.slice.call(childs); // 让childs可以用Array里的的所有方法
```
这样再forEach就不会报错了

**apply 和 call 的区别**

两者的作用完全一样，只是接受参数的方式不太一样。例如，有一个函数

```js
function func(a, b. c) {
  alert(a + b + c);
}
```
**call**
> fun.call(thisArg, arg1, arg2, ...) 

```js
var obj = {};
func.call(obj, 1,2,3);
```
**apply**
> fun.apply(thisArg, [argsArray])

```js
var obj = {};
func.apply(obj,[1,2,3]);
```
得出的结果都是一样的，都是6，看完了基本使用再来总结一下区别，apply只能接受两个参数，其中第二个参数必须是一个数组或者类数组，call则可以接受多个参数，这就是两个方法很重要的一个区别。

**最后来说说怎么用call来实现对象继承**
```js
function parents () {
  this.age = 22;
  this.func = function () {
    console.log(this.age);
  }
}

function child() {
  parents.call(this);
  this.func();
}
child(); // 22
```

child通过call方法，继承了parents的age变量和func方法，同时还可以扩展自己其他的方法。