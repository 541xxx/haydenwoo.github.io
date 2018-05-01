---
layout:     post
title:      "实现Prototype继承的几种方式"
subtitle:   "prototype and class"
date:       2018-04-30 12:00:00 
author:     "Hayden"
header-img: "img/in-post/post-eleme-pwa/eleme-at-io.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 前端开发
    - JavaScript
---

> 继承是Js里面非常重要的一个概念，几乎所有的前端同学都会用继承，但是你真的了解吗？以下就是我如何一步一步深入的了解以及学习继承的片段。
> 本文适合稍微了解原型链的同学，本文注实战，不知道Prototype以及原型链的同学请自行查资料学之~

### 构造函数实现

```js
   /**
     * 借助构造函数实现继承
    */
    function Parent(a) {
        this.name = 'parent';
    }
    function Child() {
        Parent.call(this); 
        this.type = 'child';
    }
    console.log( new Child()); // {name: 'parent', type: child}
```
上面的代码非常精简，但一点也不精炼

上述的子元素仅仅是继承了父元素的**name**属性而已，而忽略了其父元素重要的<code>prototype</code>**属性，也就是说**Parent**原型链上的东西并没有被**Child**所继承。

For example
```js
  Parent.prototype.age = 3;
  console.log(Parent.prototype)    // {age: '3',constructor: ƒ}constructor:ƒ Child1()__proto__:Objec}
  console.log(Child.prototype)    // {constructor: ƒ}constructor:ƒ Child1()__proto__:Object}
```

所以这是一次不完全的继承

### 原型链实现继承

```js
    /**
     * 借助原型链实现继承
    */
    function Parent(a) {
        this.name = 'parent';
        this.arr = [1, 2, 3];
    }
    function Child() {
        this.type = 'child';
    }
    Child.prototype = new Parent();
    console.log(new Child); // {name: 'parent', type:'child', arr: [1, 2, 3], ...} 
```
又是一个看起来没问题系列，其实这样做会引发一个新问题，废话不多说直接上代码

```js
    // 接着以上实例
    var instance1 = new Child();
    var instance2 = new Child();
    instance1.arr.push(4);
    console.log(instance1, instance2) // [1, 2, 3, 4]  [1, 2, 3, 4]
```

造成这个问题的原因是，它两共用了原型链中的原型对象，加上js中的数组和对象是引用类型，instance1改的是引用对象中的值，所以instance2的输出的值也随之改变了。

那么改如何解决这个问题呢？

### 组合继承

```js
    /**
     * 组合继承
    */
    function Parent(a) {
        this.name = 'parent';
        this.arr = [1, 2, 3];
    }
    function Child() {
        Parent.call(this);
        this.type = 'child';
    }
    Child.prototype = new Parent();
    var instance1 = new Child();
    var instance2 = new Child();
    instance1.arr.push(4);
    console.log(instance1, instance2) // [1, 2, 3, 4]  [1, 2, 3]
```
这样应该没啥问题了吧？ 对，没啥问题，但还是不够完美，比如

这里先插入一条知识点<code>instance</code>

**The instanceof operator tests whether the prototype property of a constructor appears anywhere in the prototype chain of an object.**

以上是MDN的原话，简而言之就是检测构造函数的<code>prototype</code>是否存在于目标对象的原型链的任何位置，下面就是检测**instance1**的构造函数是否存在于**Parent**原型链上

```js
    console.log(instance1 instanceof Child, instance1 instanceof Parent);  // true true
    console.log(instance1.constructor);  // f Parent() {}
```
返回都是<code>true</code>，而且**instance1**的<code>constructor</code>指向的是**parent**，这样就存在一个问题了，我该怎么区分**instance1**是**Child**的实例还是**Parent**的直接实例呢？

以下是优化

```js
    /**
     * 组合继承优化
    */
    function Parent(a) {
        this.name = 'parent';
        this.arr = [1, 2, 3];
    }
    function Child() {
        Parent.call(this);
        this.type = 'child';
    }
    Child.prototype = new Parent();
    Child.prototype.constructor = Child;
    var instance1 = new Child();
    console.log(instance1.constructor); // f Child() {}
```

### ES6实现继承

ES6引入了 Class（类）这个概念，作为对象的模板，通过class关键字，可以定义类，这种写法使对象原型的写法更加清晰，也可以理解为语法糖。

```js
    /**
     * ES6实现继承
    */
    class Parent {
      constructor() {  
        this.name = 'parent';
        this.arr = [1, 2, 3];
      }
    }

    class Child extends Parent { 
      constructor(props) {
        super(props);
        this.type = 'child';
      }
    }
    
    
```
这样就轻易的实现了继承，而且也避免了很多问题，是不是很简洁明了？以上的代码都可以用**ES5**实现，这里就不过多解释**ES6**的语法了，以免脱离了本文的初衷。

### 最后

文章写到这里就结束了，有些人可能不以为然，一个这么基础的东西也要写一篇博文，但我不这么认为，因为在实际工作中很多东西仅仅只是「会用」，回想一下自己最近有没有遇到「发现问题问题了，但就是找不到原因」的时候，如果有的话，请把红宝书上的灰擦掉，啃之。