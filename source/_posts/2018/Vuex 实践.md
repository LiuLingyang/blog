title: Vuex 实践
date: 2018-01-11
author: LiuLingyang
categories: Vue
---

# 前言

2017年对于Vue注定是不平凡的一年。凭借着自身简介、轻量、快速等特点，Vue俨然成为最火的前端MVVM开发框架。随着Vue2.0的release，越来越多的项目开始采用Vue作为他们的前端框架。而作为Vue生态中最重要的一环，Vuex渐渐进入大家的视野。

# 数据管理模式

在正式开始介绍Vuex之前，有必要介绍一下数据管理模式的前世今生。

当你在开发应用程序时，你一定会分解出很多组件进行开发，而各个组件之间想必在逻辑上多少是有关系的。那么组件之间的“通信”，就成了待解决问题。以前我们试图用事件广播来做，但随之而来的问题是，在应用不断的扩展、变化中，事件变得越来越复杂，越来越不可预料，以至于越来越难调试，越来越难追踪错误。这当然不是我们想要的，我们希望应用的各个部分都易维护、可扩展、好调试、能预测。于是数据管理模式应运而生。

![image](http://upload-images.jianshu.io/upload_images/13276697-ade5772825adcf0a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  图1

图1是最简单的组件关系，b是a的子组件，而c是b的子组件。在我们不引入任何数据管理模式之前，c组件要拿到a组件的数据只能由a先传给b，在由b传给c。如果组件树变得复杂，可想而知这将是一场灾难。看似严谨的父子结构其实严格限制了数据的流动方式。

![image](http://upload-images.jianshu.io/upload_images/13276697-94ea04d5ddad1c1a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  图2

图2是最简单的数据管理模式，所有数据将统一交给全局store来管理。a和c组件现在直接修改store里的数据，并且通过mapState从store中抓取自己感兴趣的数据到自己的组件中。而b组件，如果它对a、c组件的数据毫无兴趣，则可以做到完全解耦。

![image](http://upload-images.jianshu.io/upload_images/13276697-374b973e1101f3d3?imageMogr2/auto-orient/strip)

  图3

随着数据管理的进一步发展和演变，有一种叫单向数据流的东西冒了出来。图3就是一个表示“单向数据流”理念的极简示意。单向数据流要求各组件间的数据走向永远是单向的，可预期的。你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交Actions。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够更好地了解我们的应用。

这次我们的主人公Vuex可以说是单向数据流的最佳实践者。

## Vuex是什么？

Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。Vuex背后的基本思想，就是前面所说的单向数据流。图4就是Vuex实现单向数据流的示意图。

![image](http://upload-images.jianshu.io/upload_images/13276697-aabd7a9c819cdbe1?imageMogr2/auto-orient/strip)

  图4

接下来，我们将会更深入地探讨一些核心概念。让我们先从Store概念开始。

## Store

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态 (state) 。

安装 Vuex 之后，让我们来创建一个最简单的 store。创建过程直截了当——仅需要提供一个初始 state 对象和一些 mutation：

![image](http://upload-images.jianshu.io/upload_images/13276697-88987b3719dad698?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在，你可以通过 store.state 来获取状态对象，以及通过 store.commit 方法触发状态变更：

![image](http://upload-images.jianshu.io/upload_images/13276697-2cab0905bc826369?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutation。

## State

Vuex 使用单一状态树——是的，用一个对象就包含了全部的应用层级状态。至此它便作为一个“唯一数据源 (SSOT)”而存在。单一状态树让我们能够直接地定位任一特定的状态片段，在调试的过程中也能轻易地取得整个当前应用状态的快照。

那么我们如何在 Vue 组件中展示状态呢？由于 Vuex 的状态存储是响应式的，从 store 实例中读取状态最简单的方法就是在计算属性中返回某个状态：

![image](http://upload-images.jianshu.io/upload_images/13276697-eadfef0c06f06aa6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Getter

有时候我们需要从 store 中的 state 中派生出一些状态，例如对列表进行过滤并计数：

![image](http://upload-images.jianshu.io/upload_images/13276697-492cc3d42023b7b8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果有多个组件需要用到此属性，我们要么复制这个函数，或者抽取到一个共享函数然后在多处导入它——无论哪种方式都不是很理想。

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

![image](http://upload-images.jianshu.io/upload_images/13276697-dc45b1a528c34f5c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Getter 会暴露为 store.getters 对象：

![image](http://upload-images.jianshu.io/upload_images/13276697-88971c42880736f9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Mutation

Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数(handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数，payload作为第二个参数（额外的参数）：

![image](http://upload-images.jianshu.io/upload_images/13276697-e8b95350bd88ffe4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一条重要的原则就是要记住 mutation 必须是同步函数。

## Action

Action 类似于 mutation，不同在于：

    1.Action 提交的是 mutation，而不是直接变更状态。 

    2.Action 可以包含任意异步操作。

让我们来注册一个简单的 action：

![image](http://upload-images.jianshu.io/upload_images/13276697-4b23f98f12b9b7ae?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。

我们可以在 action 内部执行异步操作：

![image](http://upload-images.jianshu.io/upload_images/13276697-e50c2eb59936deb1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 今天的目标

![image](http://upload-images.jianshu.io/upload_images/13276697-e2d3ca07eb8312ff?imageMogr2/auto-orient/strip)

这是一个记忆小游戏，来自leftstick的vue-memory-game，github地址：[https://github.com/leftstick/vue-memory-game](https://github.com/leftstick/vue-memory-game)。今天我们就这个小游戏来详细讲解下vuex的实现。

## 组件分解

![image](http://upload-images.jianshu.io/upload_images/13276697-e8b0d5e8ba09621c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们根据分解图，先把要实现的组件挨个儿列出来：

1.  Game, 最外层的游戏面板
2.  Dashboard, 上面的logo，游戏进度，最佳战绩的容器
3.  Logo，左上角的logo
4.  MatchInfo, 正中上方的游戏进度组件
5.  Score, 右上角的最佳战绩组件
6.  Chessboard, 正中大棋盘
7.  Card, 中间那十六个棋牌
8.  PlayStatus, 最下方的游戏状态信息栏

## 目录结构

![image](http://upload-images.jianshu.io/upload_images/13276697-1571af221c8661a9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

让我们聚焦红框中关于Vuex的这一部分。接下来我们将结合Card.vue和Chessboard.vue这两个核心组件进行。

store/index.js

![image](http://upload-images.jianshu.io/upload_images/13276697-81de916d03bbe53c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

store的创建并没有什么复杂的逻辑，而state就是这个项目需要维护的整个数据。初始化cards的结构我简要的整合了一下：

![image](http://upload-images.jianshu.io/upload_images/13276697-526b7edad9b6ad08?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你可以选择初始化store的时候直接带进去，也可以像作者一样从入口处reset一遍数据，这里就不详细展开了。

Chessboard.vue

![image](http://upload-images.jianshu.io/upload_images/13276697-39e5fff4e1c20cf1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](http://upload-images.jianshu.io/upload_images/13276697-99765d410a01d8df?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Card.vue

![image](http://upload-images.jianshu.io/upload_images/13276697-ed6ad752f456dafc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我在源码上增加了一些注释，方便大家理解。

mapGetter和mapActions只是一些辅助函数，可以减少一些this.$store和dispatch action之类比较重复的代码，这些辅助函数可以帮助vuex应用的更加简便。

相比与这些可有可无的噱头，我们更关心卡片被点击的时候发生了什么？this.flipCard()后数据流向了哪里？

数据流动

当卡片被点击时，vue component 实则 dispatch出了一个action，而我们的数据，作为载荷（参数），也只能随波逐流了。

他首先来到了第一站：actions，然而actions却说你不需要什么异步处理，可以去下一站了。（如果没有异步操作，其实可以少走一站）

![image](http://upload-images.jianshu.io/upload_images/13276697-cfb3f0e7940642cd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 actions处理函数

数据没有灰心，很快他就来到了mutaions。

mutaions欣喜若狂：你在这等着，待我处理好了你再上路。

![image](http://upload-images.jianshu.io/upload_images/13276697-070fb12b6e9473e5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 mutations处理函数

不一会，数据就改头换面了，但如何告诉vue components我发生变化了呢？迷茫的数据找到了getter。getter哈哈大笑：你在我这的状态存储是响应式的，只要vue components在计算属性中应用了你，他们就能第一时间知道你的状态变更哦。

![image](http://upload-images.jianshu.io/upload_images/13276697-fd410e388dd6700f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  getter生成计算属性（如果没有派生出其他状态，可以不用getter）

![image](http://upload-images.jianshu.io/upload_images/13276697-ea97acce2a1d6b45?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 components状态更新

# 结尾

通过这个记忆小游戏，希望大家对Vuex有一个基本的理解。如果你熟悉Redux的话，掌握Vuex应该不难，因为Vuex和Redux基本思想是一致的，实现方式也是大同小异。但Vuex只适用于Vue，没有Redux那么泛用罢了。关于Vue和Vuex的问题，欢迎交流~
