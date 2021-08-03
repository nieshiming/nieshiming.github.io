---
title: levi_css总结
date: 2021-07-27 18:31:36
tags: 
    - css
    - "八股文"
categories: "css"
---

#### flex 
##### flex-basis
flex-basis: 项目占据主轴的宽度， 在分配多余空间之前会判断这个属性大小， 默认值auto
- auto: 即项目本身的大小，取决于item自身的宽度或高度
- 0%： 不暂居主轴宽度，item的宽度和高度无意义
**当主轴为水平方向的时候，当设置了 flex-basis，项目的宽度设置值会失效，flex-basis 需要跟 flex-grow 和 flex-shrink 配合使用才能发挥效果。**

##### flex-grow 
flex-grow: 用于瓜分”父容器“剩余空间 。定义项目的放大比例， 默认值0，即存在剩余空间，也不会放大
- 如果所有项目的 flex-grow 属性都为 1，则它们将等分剩余空间。(如果有的话)
- 如果一个项目的 flex-grow 属性为 2，其他项目都为 1，则前者占据的剩余空间将比其他项多一倍。


<!--more-->

##### flex-shrink
定义了项目的缩小比例， 默认值1。 即项目空间不足，item将弱小。 负值对该属性无效
flex-shrink: 计算元素宽度（包含标准盒模型、怪异盒模型）总和超出父容器部分，然后按照 盒宽度 -  (实际内容宽度) * shrink /  总的(实际内容宽度) * shrink， 计算缩放后的实际宽度

##### 简写
 { flex: flex-grow | flex-shrink | flex-basis}
 
 ```css
    #container {
        display: flex;
        width: 1000px;
    }

    /* 瓜分父元素剩余空间 */
    /* 实际宽度  300 + (1000 - 500) * 2 / 3 = 633.33 */
    .left {
        width: 300px;
        height: 200px;
        flex-grow: 2;
    }

    /* 实际宽度  200 + (1000 - 500) * 1 / 3 = 366.66 */
    .right {
        width: 200px;
        height: 200px;
        flex-grow: 1;
    }


    /* flex-shrink 测评 */
    /* 
        分配比例为： 超出宽度 400 + 300 = 700 - 500 = 200
        left和right 分配比例为 400*2:300*1= 8/3
        left = 400 - 200*8/11 = 254.54 
        right = 300 - 200*3/11 = 245.45

        计算缩放比例：用width，要考虑到盒模型，标准和模型和怪异盒模型
        超出宽度 = 盒子宽度+padding+border - 外部容器宽度
    */
    #container {
        display: flex;
        width: 500px;
    }

    .left {
        width: 400px;
        height: 200px;
        flex-shrink: 2;
    }

    .right {
        width: 300px;
        height: 200px;
        flex-shrink: 1;
    }

 ```

[知乎-flex布局进阶](https://zhuanlan.zhihu.com/p/25303493)
[flex分配空间面试题](https://juejin.cn/post/6844904066078736391)

#### meta标签作用
标签提供关于 HTML 文档的元数据。元数据不会显示在页面上，但是对于机器是可读的。  
meta的必需属性是content，当然并不是说meta标签里一定要有content，而是当有http-equiv或name属性的时候，一定要有content属性对其进行说明。例如
```html
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

##### http-equiv  模拟http标头字段
http-equiv属性是添加http头部内容，对一些自定义的，或者需要额外添加的http头部内容，需要发送到浏览器中

##### charset 解析编码
告知浏览器文档使用的解析编码
```html
    <meta charset="UTF-8">
```

##### name/content
name属性与content属性结合使用, name用来表示元数据的类型，表示当前<meta>标签的具体作用；content属性用来提供值
name属性如下：
- author: 当前页的作者名
- description: 当前页的说明
- keyword: 描述网站内容的关键词,以逗号隔开，用于SEO搜索
- viewreport: 它提供有关视口初始大小的提示，仅供移动设备使用

#### 重绘&&重排
页面在生成过程中至少会渲染一次，并且在用户访问过程中，可能还会不断渲染      
**重排一定会重绘，重绘不一定会重排**

##### 页面生成过程
- 从服务器拉取html、css、js资源
- html被html解析器解析成DOM树
- css被css解析器解析cssom树
- 结合DOM树和css树生成渲染树(render tree)
- 生成布局flow，将渲染树的所有节点进行平面合成<耗时, 重绘少了这一步>
- 根据render tree和 节点，绘制在屏幕上<耗时>
<img src="https://user-gold-cdn.xitu.io/2018/12/16/167b642d014afaf1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1">

##### 重排：简称回流
当DOM变化，影响了元素的几何位置、删除元素、添加元素，浏览器需要重新生成元素几何位置，将其放置在正确位置的过程叫做重排   
触发条件： 重新生成布局、重新排列元素， 需要重新生成布局
- 窗口大小改变
- 添加、删除元素
- 改变元素位置、改变元素尺寸
- 内容变化(例如：用户在input中输入内容)

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e68537afa1a40a098d932fe2a6c7a24~tplv-k3u1fbpfcp-zoom-1.image">

##### 重绘
当元素外观发生改变，未改变布局(几何位置)，重新把元素外观绘制的过程，叫做重绘  
重绘后，dom树未改变，cssomtre改变，跳过布局阶段，直接绘制
- 元素的某些外观改变，style_change

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e68537afa1a40a098d932fe2a6c7a24~tplv-k3u1fbpfcp-zoom-1.image">


#### 什么是BFC
BFCC: 块级格式化上下文，BFC决定了元素如何对齐内容进行定位，以及与其他元素关系和相互作用，简单来说BFC就是一个密闭空间，空间内的元素的不会影响到空间外元素

##### 形成BFC条件
- float 不为none
- overflow 不为visible
- position不为static 和 relative
- display: 为flex， table-cell, inline-block等

##### 使用BFC场景
- 防止margin塌陷
- 自适应两栏布局
- 清除浮动

##### BFC原理
- 在BFC内部盒子垂直的从顶部一个一个的排序
- BFC的区域不会与浮动的盒子产生交集，而是紧贴浮动边缘；（**重点**）
- 计算BFC的高度时，自然也会检测浮动盒子的高度，它是一个独立的渲染区域
- 盒子垂直方向的距离由margin决定，属于同一个BFC的两个盒子的margin会发生重叠
- 在BFC中，每一个盒子的左外边缘（margin-left）会触碰到容器的左边缘， 同理： 每个盒子的右边缘(margin-right)会碰到容器的右边缘

#### 实现吸底效果，底部footer在内容没有铺满可视区，始终固定在顶部，当内容超出可视区高度，跟随内容后面展示
```css
    html,
    body {
        height: 100%;
        position: relative;
    }

    .wrapper {
        position: relative;
        min-height: 100%;
    }

    .header {
        height: 100px;
        background-color: red;
    }

    .content {
        padding-bottom: 80px;
        background: blue;
        height: 400px;
    }

    .footer {
        width: 100%;
        height: 80px;
        left: 0;
        right: 0;
        bottom: 0;
        background: green;
        position: absolute;
    }


    /* 方法二， 利用css计算 calc  */
    html,
    body {
        height: 100%;
        position: relative;
    }

    .wrapper {
        position: relative;
        /* 这步骤关键 设置高度100%， 不能是min-height */
        height: 100%;
    }

    .header {
        height: 100px;
        background-color: red;
    }

    .content {
        background: blue;
        /* 计算父级高度， 减去header&&footer高度 */
        min-height: calc(100% - 160px);
    }

    .footer {
        height: 80px;
        background: green;
    }
```

#### 浏览器页面有哪三层构成，分别是什么，作用是什么?
构成：结构层、表示层、行为层 
分别是：HTML、CSS、JavaScript 
作用：HTML实现页面结构，CSS完成页面的表现与风格，JavaScript实现一些客户端的功能与业务。

#### 解释一下CSS的盒子模型？
标准的盒模型：**border--sizing: content-box** 盒子总宽度/高度=width/height（内容区宽度/高度）+ padding + border 
IE盒模型(怪异盒模型)：**border-sizing: border-box** 盒子总宽度/高度 = width/height(内容区高度/宽度 + 2 * border + 2 * padding)

#### 请你说说CSS选择器的类型有哪些，并举几个例子说明其用法？
首先说主要都有哪些选择器
- 标签选择器（如：body,div,p,ul,li）
- 类选择器（如：class="head",class="head_logo"）
- ID选择器（如：id="name",id="name_txt"）
- 组合选择器（如：.head .head_logo,注意两选择器用空格键分开）
- 继承选择器（如：div p,注意两选择器用空格键分开）
- 伪类选择器（如：就是链接样式,a元素的伪类，4种不同的状态：link、visited、active、hover。）


#### 要动态改变层中内容可以使用的方法 && innerHtml/ innerText区别
- innerHmtl: 当前标签内全部内容，包括html标签
- innerText： 当前标签内除非html标签内容
```javascript
    const wrapper = document.getElementById('wrapper');
    wrapper.innerHTML = `<p style="color:red">111111</p>`; // div内容  111 且颜色是红色
    wrapper.innerText = `<p style="color:red">111111</p>`; // div内容  <p style="color:red">111111</p>
```

#### 列出display的值并说明他们的作用？
- block：块级元素/显示
- inline：行内元素
- inline-block：行内块级元素
- none：取消样式
- normal：默认样式
- flex：弹性盒模型

#### 请列举几种清除浮动的方法(至少两种)?
- 父级div定义 height 
- 结尾处加空div标签 clear:both 
- 父级div定义 伪类:after {content:” ”,height:0; display:block; overflow:hidden; cleat:both;}

##### 你有哪些性能优化的方法？
- 使用CDN
- 可缓存的AJAX 
- 减少HTTP请求次数 
- 把JS放到底部
- 把CSS放到顶部
- 为文件头指定Expires
- 压缩相关图片等资源
- 少用全局变量、缓存DOM节点查找的结果。减少IO读取操作。

#### 说说你对页面中使用定位(position)的理解？
- **static:**可以认为静态的，默认元素都是静态的定位，对象遵循常规流。此时4个定位偏移属性不会被应用，也就是使用left，right，bottom，top将不会生效。
- **relative:** 相对定位，对象遵循常规流，并且参照自身在常规流中的位置通过top，right，bottom，left这4个定位偏移属性进行偏移时不会影响常规流中的任何元素。
- **absolute:**绝对定位，对象脱离常规流，此时偏移属性参照的是离自身最近的定位祖先元素，如果没有定位的祖先元素，则一直回溯到body元素。盒子的偏移位置不影响常规流中的任何元素，其margin不与其他任何margin折叠。
- **fixed**固定定位，与absolute一致，但偏移定位是以窗口为参考。当出现滚动条时，对象不会随着滚动。

#### 如何解决多个元素重叠问题？
使用z-index属性可以设置元素的层叠顺序 

#### position的值relative和absolute的定位原点是？
- relative（相对定位）：定位原点是元素本身所在位置； 
- absolute（绝对定位）：定位原点是离自己这一级元素最近的一级position设置为absolute或者relative的父元素的左上角为原点的

#### `::before` 和 `:after`中双冒号和单冒号有什么区别？
- 单冒号(:)用于CSS3伪类，双冒号(::)用于CSS3伪元素。
- 伪元素由双冒号和伪元素名称组成。双冒号是在css3规范中引入的，用于区分伪类和伪元素。但是伪类兼容现存样式；

伪类弥补css选择器的不足；eg: :nthc-child, :focus等等    
伪元素创建虚拟的内容容器    
可以同时使用多个伪类，但是只能同时使用一个伪元素
```css
    p:nth-child(:lang(fe)):first-child::after {
        color: red
    }

```

##### 参考文章
- [css3伪类伪元素区别](https://www.cnblogs.com/ihardcoder/p/5294927.html)

#### CSS3有哪些新特性？
- 新增各种`css`选择器 
- 圆角 `border-radius`
- 多列布局
- 阴影和反射
- 文字特效`text-shadow`
- 线性渐变
- 旋转`transform`

#### display:inline-block 什么时候不会显示间隙？(携程
- 移除空格
- 使用`margin`负值
- 使用 `font-size:0`

#### 垂直居中的几种方式 **
- flex 布局
```css
    div {
        display: flex;
        align-items: center      
        justify-content: center;
    }
```

- 百分比集合transform
```css
    div {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }

```

- margin移动： 必须确定了元素的宽高度(宽高度的一半)
```css
    #t1 {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 100px;
    height: 200px;
    margin-left: -50px;
    margin-top: -100px;
    }
```

- 绝对定位和0
```css
    #t1 {
    position: absolute;
    width: 20%;
    height: 32%;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
    }
```

- [css中的vw](https://blog.csdn.net/qq_49353940/article/details/113757685)

#### float 和position一起用是什么效果 **
> float: 是元素脱离文档流，浮动在文档流上， 可以用padding和margin进行定位
> postion： 元素相对自身或父元素定位，<relative: 相对自身左上角(0, 0), static: 默认属性， absolted: 相对父元素设置relative(0, 0)位置> 2者同时使用，并不冲突, 当设置 absolute和fixed 会是元素脱离文档流， 可以使用top,left定位

经实战发现，无论postion和float谁前谁后，都是postion在起到作用
- 当postion设置absolute、fixed. 元素已脱离文档流，float不起到作用
- 当position设置 relative/static， 元素扔处于文档流中， float将起到作用


#### 实现一个垂直居中的div，左右边距10px，高度是宽度的一半,里面内容也是垂直居中
```css
    .container {
    width: calc(100vw - 20px);
    height: calc(50vw - 10px);

    margin: 0 auto;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid red;

    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    }
```

#### scrollWidth/scrollHeight/offsetWidth/offsetHeight/clientWidht/clientHeight
##### clientWidth / clientHeight
该属性指的是元素可视区内宽度和高度，即padding+content,如果元素没有滚动条，那么就是元素设置的宽度和高度，如果出来滚动条
那么滚动条会遮盖元素宽高，该属性就是元素宽高减去滚动条的宽高

- [scrollWidth/offsetWidth/clientWidth区别](https://blog.csdn.net/qq_37288477/article/details/87890456)


##### offsetWidth / offsetHeigt
该属性和其内部的内容是否超出元素大小无关，只和本来设定的border+padding以及width和height有关(可视区域)

##### scrollWidth / scrollHeight
这两个属性指的是当元素内部的内容超出其宽度和高度的时候，元素内部内容的实际宽度和高度， width/height+padding     
**当元素内容没有超出设置的宽度和高度时候， clientWidth/clientHieght ==== scrollWidth/scrollHeight**

##### scrollTop / scrollLeft
设置向上向下滚动距离

##### offsetTop / offsetLeft
距离上一层元素 postion设置不为static距离， 如果上一层没有，则向上上层获取，直接body元素