---
title: '吃一堑长一智系列_babel基础知识'
date: 2021-07-29 10:03:29
tags: "babel"
categories: "babel"
---

#### 困扰

作为工具开发者，babel 关联问题是难绕过去的砍。
在 babel@6 时候，最常收到反馈之一就是 regeneratorRuntime is not defined
而到了 babel@7，最常收到反馈之一 Cannot find module 'core-js/library/fn/**'.
那是什么问题导致这些问题的出现呢，我觉得有一个 issue 特别能代表这一类的开发者。大家不要笑，我们内部一些基础模块也有这个问题
**总结来讲：Babel 在编译大家的代码时候，会依据大家配置的 preset or plugin 注入一些模块依赖，而这些模块依赖是大家需要在 pkg.dependencies 里面体现出来的，否则很可能出现的问题就是加载不到具体的文件或者加载错误的版本的文件。**

<!--more-->

根本的原因是什么：其实大家对
- @babel/preset-env
- @babel/plugin-transform-runtime
- @babel/runtime
- core-js
- @babel/polyfills
- babel-polyfills


#### 参考文章
- 【吃一堑长一智系列: 99% 开发者没弄明白的 babel 知识](https://zhuanlan.zhihu.com/p/361874935)