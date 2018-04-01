---
layout:     post
title:      "CSS 进化史"
subtitle:   "CSS Evolution"
date:       2018-03-16 12:00:00 
author:     "Hayden"
header-img: "img/in-post/css-evolution/banner.png"
header-mask: 0.3
catalog:    true
tags:
    - 前端开发
    - CSS
---




### CSS 进化过程： CSS,CSS-PREPROCESSOR,BEM,CSS Modules,Styled Components

CSS是一个弱类型语言，但也是前端非常重要的一部分，所以才衍生出了这么多有趣以及有用的工具。在一些大型项目中，CSS的管理以及规范就成了至关重要以及必不可少的一个环节，本文的目的是让大家在目前前端日益强大的同时，能够了解CSS的一些变迁以及如何选择工具来完全的Handle CSS



### 桀骜不驯的CSS


当一个Team在开发项目的时候，当功能和业务模块日益增多的时候，这个时候CSS开始变得混乱，因为每个开发者都有自己的风格，所以导致项目日益壮大的同时CSS留下的祸根也在悄然成长。所以「一致性」就成为了首要解决的问题。




### CSS 预处理器(preprocessor)

目前用的较多的预处理预处理有 **Stylus Sass Less** 基本上都有自己的语法，功能也较强大，可以使用一些变量来传参，这里就拿**stylus**做为例子吧

这里是stylus的官网，里面有一些具体的语法，东西也不多可以先过一遍 [stylus](http://stylus-lang.com/docs/mixins.html)

首先可以先创建几个文件分别是

```css
mixins.styl
base.styl
```
首先来介绍 **mixins**

主要是用于一些复用的样式片段，然后像JS一样传参

比如自定义一个圆角的兼容写法
```css
border-radius(n)
  -webkit-border-radius n
  -moz-border-radius n
  border-radius n
```
然后使用的时候

```css
.hello
    border-radius(5px)
```

是不是方便了很多呢
其实现在webpack里面的**postcss**插件可以实现自动加浏览器前缀，但这不是本文的讨论范畴，有兴趣的自行查之

接下来就是 **base.styl**

顾名思义这个地方是用来放一些通用以及基础的一些样式， 比如主题色啊，主字体字号，以及什么的 

```css 
$primary-color = #000 // 定义主题颜色 
$danger-color = #000
$warn-color = #000
$warn-btn-color = $warn-color
$success-btn-color = #000
$font-size-primary 14px
$font-size-s $font-size-primary * 0.7

```
调用

```css
    .main-btn
        color danger-color
```

这样做的好处是方便管理项目，统一都用变量来定义一些主题色以及主字号，也不容易出错。

### BEM规范

BEM的意思就是块（block）、元素（element）、修饰符（modifier），全名是 Block Element Modifier，前端这个大家庭真是为了CSS操碎了心呢，首先用一句来概况为啥需要BEM：

还是「一致性」问题，每个人的风格都不太一样，但是放在页面内也就都能跑起来，导致非常混乱，换一个人就很难维护，它的出现就是规范团队的写法，大家都按照这个风格来写，日后就不用MMP了。

**Block**

所有东西都划分为一个模块，header是block  header里面的input也是一个block，block可以之间可以相互嵌套

```html
    <!-- page是一个block -->
    <div class="page"> 
        <!-- page-header是一个block -->
        <div class="page-header"></div>
    </div>
```
**-**中划线同英语里面连字符用法一样，用于多个单词描述

**Element**

顾名思义，Block下面的元素，一个Block下面的所有Element无论层级如何都需要摊开属于一个Block，不然就会造成太多嵌套
```html
    <!-- page是一个block -->
    <div class="page"> 
        <!-- page__title是page下的Element -->
        <div class="page__title"></div>
        <!-- page-header是一个block -->
        <div class="page-header">
            <!-- page-header__main是page-header下的元素 -->
            <div class="page-header__main"></div>
        </div>
    </div>

```
看到这里，有些童鞋可能会有些疑问，为什么title和header同是page之下的元素，header成了独立的块，title却没有？
因为title下面没有层级的嵌套，因为**BEM 最多只有 B+E+M 三级**

**__** 双下划线表示块与元素的连接

**Modifier**

修饰符，比如 .active .disabled

**下面来总结一下用法以及约定**

**__**双下划线代表B与E的连接 比如 **page__title**

**_**单下划线代表B与M或E与M的连接 比如 **page_active page__title_active**

**-**中划线用于多个单词组合的描述**page-header**

**BEM 最多只有 B+E+M 三级**

**BEM 最多只有 B+E+M 三级**

**BEM 最多只有 B+E+M 三级**

不然就会出来很多级的嵌套而造成结构以及样式紊乱

还有，BEM并不严格的与DOM挂钩，所以元素层级嵌套的多少层也没关系。命名约定是为了更好的识别顶层组件，如**page**

```html
     <!-- page是一个block -->
    <div class="page"> 
        <div class="page-content">
            <div class="page-content__body">
                <div class="page-content__item"></div>
            </div>
            <div class="page-content__body"></div>
        </div>
    </div>
```
虽然在HTML结构里page-content__body和page-content__item是父子级关系，还是那句话BEM并不严格与DOM挂钩，所以就可以采用上面那种写法


### CSS modules

最后来到CSS modules

CSS modules的做法类似于 vue style scope的原理，给那些加了scope的块加上hash，从而实现作用域隔离，使样式文件只作用域当前文件。

因为vue-loader默认自带CSS module，所以这里就的demo就拿vue来演示

```html

<template>
    // vue将拿到的module对象命名为$style注入组件计算属性
    <div :class="[$style.hello]"></div>
</template>

<style module> 
    .hello {
        color: #000;
    }
</style>
```

经过编译之后就会变成

```html
    <div class="b7RHp1qw89123Iuq_1"></div>
```
彻彻底底的局部作用域，大家井水不犯河水

如果需要定义一些全局的样式则可以使用 **:global(.classname)**的语法，从而不会编译成Hash字符串

```css
:global(.title){ 
        color: #666
    }
```

看到这里，大家可能会有个疑问：编译出来一串这样的东西出了问题那岂不是大海捞针？

那当然是选择配置啊 ！！！

```js
module: {
  rules: [
    {
      test: '\.vue$',
      loader: 'vue-loader',
      options: {
        cssModules: {
          localIdentName: '[path][name]---[local]---[hash:base64:5]',
          camelCase: true
        }
      }
    }
  ]
}
```
这样编译出来的class就会变成： 路径 + 文件名 + class名 + hash


### 最后

通过这些年的发展以及变迁，CSS已经逐渐变得工程化，模块化，而且还有这么多强大的预处理器，很多以前的坑如今都可以轻松越过去，本文的初衷是让大家能找到一个适合自己项目的方式Coding，并不是一篇纯粹的教程文，其中有有很多具体的细节以及用法需要自己去看官方文档。

要不就当成科普文看？   逃      :)