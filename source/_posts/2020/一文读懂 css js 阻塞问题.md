title: 一文读懂 css js 阻塞问题
date: 2020-04-24
author: LiuLingyang
categories: 基础

---
# 前言
先抛出几个问题：
* css 加载会不会阻塞 js 的加载？（不会）
* css 加载会不会阻塞 js 的执行？（会）
* css 加载会不会阻塞 DOM 的解析？（不会）
* css 加载会不会阻塞 DOM 的渲染？（会）
* js 加载会不会阻塞 DOM 的解析？（会）
* js 加载会不会阻塞 DOM 的渲染？（会）
* js 执行会不会阻塞 DOM 的解析？（会）
* js 执行会不会阻塞 DOM 的渲染？（会）

可以看出 js 是全阻塞的，这也是为什么 js 要放尾部的原因。
至于 css 放头部则是为了避免页面一开始样式，而后出现样式导致页面闪动情况，这样用户体验就比较差了。

# 浏览器的主要进程和职责
![](https://upload-images.jianshu.io/upload_images/13276697-efa7667456530d8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关于 css，js 的阻塞问题，都跟浏览器的渲染进程有关。而渲染进程又是多线程的。理清各个线程的职责及相互之间的合作关系，就能窥探其原理。
* JS 引擎线程（单线程）：负责解析 Javascript 脚本，运行代码
* GUI 渲染线程：负责渲染浏览器界面，解析 HTML，CSS，构建 DOM Tree，Style Tree 和 Render Tree，布局和绘制等

> 注意：GUI 渲染线程与 JS 引擎线程是互斥的，当 JS 引擎执行时 GUI 线程会被挂起，所以当 JS 加载和执行时，会阻塞住 DOM 的解析和渲染，导致白屏时间很长

# 浏览器渲染流程
![](https://upload-images.jianshu.io/upload_images/13276697-fd05e089969cdbf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.解析 HTML 生成 DOM Tree
2.解析 CSS 生成 Style Tree
3.将 DOM Tree 与 Style Tree 合并在一起生成 Render Tree
4.遍历 Render Tree 开始布局，计算每个节点的位置大小信息（Layout）
5.绘制 Render Tree，绘制页面的像素信息（Painting），显示到屏幕上（Display）
> DOM Tree 和 Style Tree 是并行构建的，所以 CSS 加载不会阻塞 DOM 的解析

> 由于 Render Tree 是依赖于 DOM Tree 和 Style Tree 的，因此，css 加载会阻塞 Dom 的渲染

> GUI 渲染线程与 JS 引擎线程是互斥的，加载解析 css 时，JS 引擎会被挂起，所以 css 会阻塞 js 的执行

# 回流和重绘

> 回流必将引起重绘，重绘不一定会引起回流

回流(Reflow)：当 Render Tree 中部分或全部元素的尺寸、结构、或某些属性发生改变时,浏览器重新渲染部分或全部文档的过程称为回流
重绘(Repaint)：当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility 等）,浏览器会将新样式赋予给元素并重新绘制它,这个过程称为重绘

>回流比重绘的代价要更高

# 资源加载优先级
想要提升页面的加载速度，除了关注 css，js 阻塞的问题外，了解资源加载的优先级同样重要。
我们知道资源加载本身不存在互相阻塞的问题，但浏览器依然会按照资源默认的优先级确定加载顺序：
1.html 、 css 、 font 这三种类型的资源优先级最高
2.然后 script 、 xhr 请求
3.接着是图片、语音、视频

然而，有些资源我们知道很重要，想优先加载；有些资源无关紧要，想延后加载，那么如何手动控制浏览器加载优先级呢？
主要有4种指令：
* preload 预加载（提升优先级）：通知浏览器接下来马上就会用到的资源，并尽快开始加载资源
* prefetch 预获取（最低优先级）：通知浏览器这是稍后可能需要用到的东西，可以延迟加载（在带宽空闲(idle)时再加载）
* asnyc 异步获取（降低优先级）：资源可以异步加载，加载完即可执行（乱序执行）
* defer 异步获取（降低优先级）：资源可以异步加载，但需要按照资源加载顺序执行（按序执行）
