title: vue 生态圈
date: 2017-09-02
author: LiuLingyang
categories: Vue
---

## 前言

公司社区上关于Vue的文章挺少的（少的可怜），不禁为Vue愤愤不平，此文应运而生。

但笔者水平有限，也写不了什么特别高深的东西，只能简单介绍下Vue生态圈，如有不对之处，还望指正。

## Vue.js

Vue.js是一款极简的 mvvm 框架，如果让我用一个词来形容它，就是“轻巧”。如果用一句话来描述它，它能够集众多优秀逐流的前端框架之大成，但同时保持简单易用。为什么这么说，因为Vue.js通过简洁的API提供高效的数据绑定和灵活的组件系统。在前端纷繁复杂的生态中，Vue.js却一直受到一定程度的关注，而其本身也在高速发展中，不论是生态、社区、资源、插件等等都在日趋壮大。如果您还未曾了解Vue.js的话，建议您阅读[http://cn.vuejs.org/v2/guide/](http://cn.vuejs.org/v2/guide/)，这里有Vue.js正确的食用方法。如果您想在此文中知晓Vue.js核心的话，可能要让您失望了。本文不会介绍Vue.js的语法，模板、组件、API等等，这是一篇介绍Vue.js周边或者说Vue.js生态的文章（当然这要求你对Vue.js有一定程度的了解）。

## Vuex

Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。灵感来自Flux和Redux,但简化的概念和实现是一个专门为`Vue.js`应用设计的状态管理架构。如果您的应用程序足够简单，建议您不要使用Vuex。但是，如果您需要构建是一个中大型单页应用，您很可能会考虑如何更好地在组件外部管理状态，Vuex 将会成为自然而然的选择。

言归正传，什么是状态管理模式？讲讲我自己的理解吧。当你在开发应用程序时，你一定会分解出很多组件进行开发，而各个组件之间想必在逻辑上多少是有关系的。那么组件之间的“通信”，就成了待解决问题。以前我们试图用事件广播来做，但随之而来的问题是，在应用不断的扩展、变化中，事件变得越来越复杂，越来越不可预料，以至于越来越难调试，越来越难追踪错误。这当然不是我们想要的，我们希望应用的各个部分都易维护、可扩展、好调试、能预测。于是，状态管理模式冒了出来。

![image](http://upload-images.jianshu.io/upload_images/13276697-dc8081d4ab35469b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

几个重要的概念：

*   state：驱动应用的数据源；
*   actions：响应在用户操作行为导致的状态变化；
*   mutations：引发状态改变的所有方法的集合；
*   store对象：store对象是Vuex.Store的实例。在store内有分为state对象和mutations对象。

简单点说，本来需要共享状态的更新是需要组件之间通讯的，而现在有了Vuex，所有组件就都和store通讯了。问题就自然解决了。

## vue-router

都说Vue牛逼，那一定也有一套自己路由的实现，接下来让我们来看看vue-router。

vue-router是Vue.js官方的路由插件，它和vue.js是深度集成的，适合用于构建单页面应用。vue的单页面应用是基于路由和组件的，路由用于设定访问路径，并将路径和组件映射起来。传统的页面应用，是用一些超链接来实现页面切换和跳转的。在vue-router单页面应用中，则是路径之间的切换，也就是组件的切换。

vue-router的用法也是异常简单：

**HTML**

```html
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入`to`属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个`<a>`标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

**JS**

<pre data-type="javascript">
0. 如果使用模块化机制编程，導入Vue和VueRouter，要调用 Vue.use(VueRouter)

1. 定义（路由）组件
// 可以从其他文件 import 进来
const Foo = { template: 'foo' }
const Bar = { template: 'bar' }

2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是通过 Vue.extend() 创建的组件构造器，或者，只是一个组件配置对象。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

3. 创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
})

4. 创建和挂载根实例
// 记得要通过 router 配置参数注入路由，从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')
</pre>

现在，你已经完成了整个应用的路由配置，到浏览器上看看效果吧！

## vue-devtools

vue-devtools是基于google chrome浏览器的一款调试vue.js应用的开发者浏览器扩展。一张图看懂它是什么：

![image](http://upload-images.jianshu.io/upload_images/13276697-69008de1aea50cab?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意：初次安装好vue-devtools以后，需要关闭chrome devtool再开，才能看见vue的标签（通常在最后）。如果你正在使用我提供的例子，或者同样也是在浏览器访问自己本机写的html，需要在vue-devtools的设置里面勾选“允许访问文件URL”（如图）。

![image](http://upload-images.jianshu.io/upload_images/13276697-377be978a9d309c4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## webpack

把webpack放在这里讲似乎不太合适，毕竟这并不是Vue独有的东西。而且webpack的大名说不定比Vue本身还响亮。看近期Github上各大主流的项目，无一例外都已经是基于webpack来开发。你可以不打算将其用在你的项目上，但没有理由不去掌握它。篇幅有限，不展开描述，一句话概括webpack的主要用途：把所有浏览器端需要发布的资源做相应的准备，完成资源的合并和打包。

## vue-cli

作为Vue的脚手架，vue-cli无疑是出色的。它可以帮你快速的上手vue构建的工程，而无需再花多余的时间去熟悉vue工程的文件系统。

使用它的方法也很简单：

1.  npm install -g vue-cli      //全局安装vue-cli
2.  vue init webpack projectName  //生成项目名为projectName的模板，这里的项目名projectName随你自己写
3.  cd projectName                              
4.  npm install             //初始化安装依赖
5.  npm run dev            //启动工程

在浏览器打开http://localhost:8080，则可以看到欢迎页了：

![image](http://upload-images.jianshu.io/upload_images/13276697-319302c2973f3f2f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再回头看看项目结构，是不是一目了然:

![image](http://upload-images.jianshu.io/upload_images/13276697-40879b0df04378e0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## iView

其实这是Vue.js的一个ui库，我一直不明白为什么作者不直接把它叫做ivue或者vue-ui，似乎这样的名字才更明了吧，或许好名字都早已被人抢先注册了~

言归正传，iView本身还是异常强大的，附iView官方文档：[https://www.iviewui.com/](https://www.iviewui.com/)。


## React？

what？React怎么成Vue生态圈里的东西了？别激动，这不是有个问号吗？其实我只是想讲讲和React的区别罢了，瞧把你激动的。

相似：

其实都是model driven思想的严格践行者，以及通过component拆分完整整个系统的分治。

不同：

1.react基本上已经有一套遵循Flux的完整开发方案（基本上也就这一套大家默认的方式），而vue虽然有配合vuex使用，但是还有其他很多组织方式来解决，所以并不算是有固定模式，相对灵活很多，当然这个你可以看作是优势，也可以看作是不足；

2.react社区还是要比vue大很多；

3.react在view层侵入性还是要比vue大很多的；

4.首次渲染性能，对于大量数据来说react还是比vue有优势；

5.对于component的写法，react偏向于all in js，语法学习上需要下一些功夫，而vue配合vue-loader，其实在很大程度上让你不会觉得陌生--这不就是web component么。

是时候结束本文了，如果你对Vue依旧没有兴趣，不是你的锅，也不是Vue的锅，文章写的太烂，望海涵~
