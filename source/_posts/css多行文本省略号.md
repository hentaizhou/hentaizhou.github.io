---
title: css多行文本省略号
date: 2019-03-21 13:03:41
categories:
 - css
tags:
 - 前端
 - css技巧

---

# CSS多行文本设置省略号



## 单行文本省略号

  如果要设置单行文本超出容器末尾显示省略号是非常简单的，通过:

```css
overflow: hidden;
text-overflow: ellipsis;
```

即可实现单行文本的省略号。

## 多行文本的省略号

​	下面就是本文主要讨论的点了，如何实现多行文本超出区域显示省略号，或者说可以设置最多显示几行。

<!-- more -->

​	实现的方式有多种，可以用css，也可以用js。 其中css也有多种方法, 但一些方法或多或少都有点hack的味道，下面我们介绍一种浏览器原生的多行文本显示的方法。

在包含文本的标签中添加如下css样式:

```css
p {
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 4;
}
```

> Tips:  1. 只有中文内容在到达最大宽度时才会自动换行
>
> 2. 如果要英文支持的话需要添加  `word-break: break-all` 文本才会换行。
> 3. 经测试  此css样式在chrome、safari浏览器上能正常生效，但在Firefox上好像并不起效果。



## 踩坑

​	单独尝试添加上面的样式可以实现多行省略号，但我在Vue以及nuxt框架下开发，几经尝试都不起效果，一直认为是不是浏览器不支持这个样式。结果倒腾许久才发现是 autoprefixer这个插件干的"好事"。

在原来的css中我们添加了`-webkit-box-orient: vertical` 这个样式。 但是经过 autoprefixer 处理之后，元素中竟然没有了！！  好吧。。。 手动加上之后就可以实现多行文本省略号的效果了。

​	那怎么解决 autoprefixer去掉 `-webkit-box-orient: vertical` 这个样式的问题呢？  很简单，在该样式样多加一条注释   `/*! autoprefixer: off */`   。

​	哦了，这下我们就能愉快的使用多行文本设置省略号了～







