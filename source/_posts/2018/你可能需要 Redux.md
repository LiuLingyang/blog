title: 你可能需要 Redux
date: 2018-09-26
author: LiuLingyang
categories: Redux
---

# 前言
你可能并不需要Redux。
有点啪啪打脸的意思，但你必须权衡Redux架构给你项目带来的短期和长期的效益来决定是否应该使用Redux。如果只是一个简单的应用，Redux只会带来繁琐冗余的代码体验，这真的得不偿失。如果打算开发大型单页应用，Redux将会成为自然而然的选择。引用 Redux 的作者 Dan Abramov 的话说就是：
>Flux 架构就像眼镜：您自会知道什么时候需要它。
# Why Redux
当你在开发应用程序时，你一定会分解出很多组件进行开发，而各个组件之间想必在逻辑上多少是有关系的。那么组件之间的“通信”，就成了待解决问题。
通俗点说如果一个 model 的变化会引起另一个 model 变化，那么当 view 变化时，就可能引起对应 model 以及另一个 model 的变化，依次地，可能会引起另一个 view 的变化。直至你搞不清楚到底发生了什么。
以前我们试图用事件广播来避免这些连锁反应，但随之而来的问题是，在应用不断的扩展、变化中，广播事件也变得越来越复杂，越来越不可预料，以至于越来越难调试，越来越难追踪错误。
这当然不是我们想要的，我们希望应用的各个部分都易维护、可扩展、好调试、能预测。
于是Redux应运而生。
# 单向数据流
![](https://upload-images.jianshu.io/upload_images/13276697-c94158eb9114e4e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
单向数据流可以说是Redux的设计核心。上图是一个表示“单向数据流”理念的极简示意。单向数据流要求各组件间的数据走向永远是单向的，可预期的。你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交Actions。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够更好地了解我们的应用。
# Action
Action是把数据从应用传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说你会通过store.dispatch() 将 action 传到 store。你可以理解为这是一个修改数据的指令。
为什么要用Action？
store的数据操作封装在store内部，各组件禁止直接修改state数据，只能发布一个修改数据的意图通知store进行数据更新，这个意图就是Action。
按照约定，action 具有 type 字段来表示它的类型。type 可被定义为常量或者是从其它模块引入，最好使用字符串。除了 type 字段外，action 对象的结构完全取决于你。
为了使Action可复用，我们最好编写Action创建函数。
# Reducer
Redux中的数据处理部分，我们称之为Reducer。
Reducer 就是一个纯函数，接收旧的 state 和 action，返回新的 state。
纯函数定义：相同的输入一定会对应相同的输出。翻译过来就是只要传入Action相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。
特别注意：不要修改 state。可以使用 Object.assign() 新建了一个副本或者借助Immutable.js。
Reducer过大时可以拆分Reducer。让每个 reducer 只负责管理全局 state 中它负责的一部分。每个 reducer 的 state 参数都不同，分别对应它管理的那部分 state 数据。
# store
 Redux 应用只有一个单一的 store。当需要拆分数据处理逻辑时，你应该使用 reducer 组合，而不是创建多个 store。
你可以通过createStore()接口创建你的store，第一个参数就是之前组合好的reducer，第二个参数用于设置 state 初始状态。第三个参数是一个高阶函数，主要用于Middleware。
store主要职责：
* 提供 getState() 方法获取 state；
* 提供 dispatch(action)方法更新 state；
* 通过subscribe(listener) 注册监听器;

# Middleware
Middleware提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。 你可以利用 Middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。
Redux middleware 就像一个链表。每个 middleware 方法既能调用 next(action) 传递 action 到下一个 middleware，也可以调用 dispatch(action) 重新开始处理，或者什么都不做而仅仅终止 action 的处理进程。
创建 store 时， applyMiddleware 方法的入参定义了 middleware 链，你只需要按照规定的格式编写你自己的Middleware即可。
applyMiddleware 实现原理其实很简单，代理store.dispatch，在调用前后插入自己的逻辑。
# reducer enhancer
reducer enhancer作为一个函数，接收 reducer 作为参数并返回一个新的 reducer，这个新的 reducer 可以处理新的 action，或者维护更多的 state，亦或者将它无法处理的 action 委托给原始的 reducer 处理。
reducer enhancer可以看做是reducer的扩展，主要职责：
* 统一修正reducer处理后的数据
* 统一计算reducer处理后的计算属性
* 实现撤销重做

# Ending...
想要讲清楚Redux不是一件容易的事，文章也只是介绍了Redux的一些基本概念。
要想真正理清Redux，对应的项目经验是必不可少的。
如果你还在用low爆了的中介者模式或者难以管理的事件广播的话，强烈推荐迁移成Redux，你会发现一片不一样的天空。。。