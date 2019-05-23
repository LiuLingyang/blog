title: 老调重弹 webpack
date: 2018-06-14
author: LiuLingyang
categories: Webpack
---

# 前言
本来是准备只总结下热更新的，但发现webpack4升级后还是有不少改动，索性放在一起总结一下。
按照惯例，先上链接：[https://github.com/LiuLingyang/framework](https://github.com/LiuLingyang/framework)。（注：该脚手架基于 webpack4 ）

# 多入口多出口
在大家熟知的脚手架中，大部分可能都是单页面应用，即单入口单出口。当你的网站足够简单，SPA确实提高了用户体验。但对于企业级应用而言，SPA极易造成单个页面体积过大，它可能加载了很多其他页面的资源。SPA的另一个问题就是不利于SEO。针对这些问题，多页面 + 局部SPA 成为了最优的选择。

多页面大多数情况有多个html模板文件，并且要求引入各自的JS文件配置，即多入口多出口。如何生成多个页面？只要以链式的方法，调用html-webpack-plugin插件，每次调用都要指定filename也就是生成页面的名字。调用一次生成一个页面，调用两次生成两个页面,以此类推。

如何引入各自的JS文件配置？比如说我们希望index.html引入的是index.js，test.html引入的是test.js如何操作呢？
我们需要在html-webpack-plugin中再配置一个参数，chunks，支持数组，数组里面填写的是引入的js，也就是entry里面配置的key，要引入哪个js就配置entry中的哪个key。

 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-10a11c392da5f96e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 开发环境？生产环境？
ebpack4中提供了更加快捷的mode模式来区分开发环境和生产环境。mode有两个值：development和production，默认值是 production。mode是我们为减小生产环境构建体积以及节约开发环境的构建时间提供的一种优化方案，提供对应的构建参数项的默认开启或关闭，降低配置成本。

 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-2094665814acff8f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-4c1d7d5b3726a079?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

development模式下，将侧重于功能调试和优化开发体验。
production模式下，将侧重于模块体积优化和线上部署。注意：这里不再需要配置UglifyJSPlugin插件，而是会自动启用uglifyjs对代码进行压缩。

# webpack4其他改动
webpack4官方移除了commonchunk插件，改用了optimization属性进行更加灵活的配置，这也应该是从V3升级到V4的代码修改过程中最为复杂的一部分。

由于webpack4以后对css模块支持的逐步完善和commonchunk插件的移除，在处理css文件提取的计算方式上也做了些调整，之前我们首选使用的extract-text-webpack-plugin也完成了其历史使命，将让位于mini-css-extract-plugin。

剩下的问题大部分都是因为当前的包与webpack4不兼容。解决办法：升级呗。

这里就不贴出详细代码了，看源码吧。

# 模块热替换(Hot Module Replacement)
HMR是webpack最令人兴奋的特性之一，当你对代码进行修改并保存后，webpack将对代码重新打包，并将新的模块发送到浏览器端，浏览器通过新的模块替换老的模块，这样在不刷新浏览器的前提下就能够对应用进行更新。HMR是一个非常值得去深入研究的东西，它绝不是目前我们看到的大多数技术文章说的配置一个hot参数这么简单，有兴趣的小伙伴可以去看看它的实现原理，目前为止我也只看过一点点。

实现模块热替换的方式有两种，一种是采用中间件的方式，webpack-dev-middleware和webpack-hot-middleware配合使用，实现热替换功能；另外一种是使用webpack-dev-server插件，配置一些必要的参数，也可以实现热替换功能。需要注意的是两种方式都只用于开发环境。打包后的文件都是放到内存中的，所以我们在项目中是看不到它打包以后生成的文件的。本例采用第二种方式实现。

两种方式都需要配置的项我就放在最前面说好了：
1. webpack的plugins添加webpack.HotModuleReplacementPlugin()，这是一个必要的热替换插件。你可能在某些地方可能会看到webpack-dev-server添加了hot: true以后，它会自动帮我们加这个插件。相信我，还是会报错的，所以还是老老实实的手动添加到plugins里面吧。

2. 我们通过 HotModuleReplacementPlugin 启用了模块热替换，则它的接口将被暴露在module.hot属性下面。我们需要把整个项目的要被webpack编译的文件都设置为接受热更新，而最简单的方式就是在入口文件的地方添加：
 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-205f22821d59a879?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 中间件
中间件的方式配置起来还是比较简单的。

 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-685718606099b7ba?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就这几行核心代码，我就直接贴出来了。

中间件是可以和自己的后端服务整合的，相对与webpack-dev-server，它拥有更好的灵活性和自由度。

Koa实现热更新的中间件似乎还没有包可以直接引，但大神们早已为我们铺好了路。Kao架构可以参考：https://www.cnblogs.com/liuyt/p/7217024.html

## webpack-dev-server
webpack-dev-server实际上相当于启用了一个express的Http服务器+调用webpack-dev-middleware。它的作用主要是用来伺服资源文件。这个Http服务器和client使用了websocket通讯协议，原始文件作出改动后，webpack-dev-server会用webpack实时的编译，再用webpack-dev-middleware将webpack编译后文件输出到内存中。

webpack-dev-server支持两种配置方式（iframe mode && inline mode）。

iframe模式不需要配置任何东西，只需要在你启动的项目的端口号后面加上/webpack-dev-server/即可，比如：http://localhost:8080/webpack-dev-server/。
它的原理是在页面中嵌入了一个iframe标签来实现热更新。因为这种模式配置比较简单，这里就不展开介绍这种模式了。有兴趣的小伙伴可以自行研究。

inline模式不需要改变url，但配置起来比iframe模式繁琐的多。

inline模式又分两种方式启动，Node方式和非Node方式。

#非Node方式
非Node方式有关webpack-dev-server的配置都在webpack.config.js的devServer参数里。一定要指定output.publicPath，如果不指定会导致HMR无法工作。

 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-387aa53641c0c109?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

contentBase就和output.path是一样的作用，如果不配置这个参数就会打包到项目的根路径下。这里再强调一下为了加快打包进程打包后的文件是放到内存中的，所以我们是看不到实际文件的。

hot:true&&inline:true就是开启inline模式和热更新，没什么好说的。

proxy：当您有一个单独的后端开发服务器，并且想要在同一个域上发送API请求时，则代理这些url。设置代理后contentBase需要修改为绝对路径。本例中将被修改为http://localhost:3001/。

命令行启动：webpack-dev-server --hot –inline。

## Node方式
如果使用Node.js方式，那么即使你配置了devServer也会被忽略，真正起作用的应该是Node.js的dev.js文件，这个文件放在build目录下面。
此时启动项目：node build/dev.js。

build/dev.js
 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-dcf1a45bf94c1db9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然这还不够，还需要在entry属性里添加webpack-dev-server/client?http://«path»:«port»/ 和 webpack/hot/dev-server。
 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-1e035f357e5263bf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好的，选择一个你喜欢的方式启动起来吧，如果能在控制台看到以下的信息，代表热更新已经成功启动了。
 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-8175752ec6f67443?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## react-hot-loader
你可以已经注意到了，在entry属性中我还添加了react-hot-loader/patch。

webpack-dev-server的热更新对于保存react状态是无法做到的，所以才有了react-hot-loader这个东西，这个不是必须配置的插件。
不过如果你想要更新时可以保存state，这是必须的。

配置分三步走：

1..babelrc中添加react-hot-loader/babel：
 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-1dd57addaef0f156?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.entry参数添加react-hot-loader/patch；

3.修改跟组件接收热更新：
 ![Alt pic](http://upload-images.jianshu.io/upload_images/13276697-7ea145c9d4e00bbd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

经过一顿神操作后，相信你已经弄清楚了热更新到底是怎么回事。如果你的项目还没有开始应用，抓紧尝试下吧。
