title: React 同构实践
date: 2019-05-23
author: LiuLingyang
categories: React
---

#  前言
目前单页面应用（SPA）很是流行，同时也带了一些问题，如SEO不友好，首屏加载慢等，为了解决上述问题，所谓的服务端渲染就粉墨登场了。
React 作为一个 SPA 应用开发框架同时也支持服务端渲染，本文将详细介绍如何搭建一个 React 服务端渲染的项目。
如果你倾向于开箱即用的体验，可以尝试更高层次的解决方案 [Next.js](https://nextjs.org/)。

在你阅读之前，你需要具备以下技术能力：
> * React 全家桶
> * Webpack
> * ES6
> * Promise
> * Express
# 同构
这里我们把”服务端渲染“一词替换成”同构“。其实，这两个词的背景和所表达的意义大体相同，但又有一定差别：服务端渲染主要侧重架构层面的设计，而同构更侧重代码复用。同构的核心是“同一套代码”，这是脱离于两端角度的另一个维度。

随着 Node.js 的异军突起，前后端开发有了归一化编程语言的基础土壤，同构的概念也因此得以更广泛的传播，React 率先引领了这种潮流。同构实际上是客户端渲染和服务器端渲染的一个整合。我们把页面的展示内容和交互写在一起，让代码执行两次。在服务器端执行一次，用于实现服务器端渲染，在客户端再执行一次，用于接管页面交互。
## 同构的优势
* 更好的性能
* SEO 优化
* 同一套代码，可维护性更强
* 更好的用户体验

## 同构的劣势
* 服务端处理逻辑增多，增加了复杂性
* 增加了服务端 TTFB（Time To First Byte）时间，降低了服务端返回的速度

# 认识 renderToString 和 hydrate
为了实现服务端渲染，打造同构应用，React 也实现了相应的 API。依赖 React 提供的 ReactDOMServer 对象可以实现服务端渲染。ReactDOMServer 对象主要提供 renderToString 方法。
``` js
const ReactDOMServer = require('react-dom/server');
ReactDOMServer.renderToString(element);
```
该方法接受一个 React element，并将此 element 转化为 HTML 字符串，通过浏览器返回。因此，在服务端将页面拼接字符串插入 HTML 文档中并返回给浏览器，完成初步服务端渲染的目的。
为了客户端在渲染组件时，最大限度地保留在服务端使用 renderToString 生成的内容结构，ReactDom 相应的在客户端提供了一个新的 API：hydrate。
``` js
import ReactDOM from 'react-dom';
ReactDOM.hydrate(App, document.getElementById('app'));
```
前菜已经上齐了，下一节开始上正餐。
# 客户端路由与服务器端路由的差异
实现 React 的 SSR 架构，我们需要让相同的 React 代码在客户端和服务器端各执行一次。大家注意，这里说的相同的 React 代码，指的是我们写的各种组件代码，所以在同构中，只有组件的代码是可以公用的，而路由这样的代码是没有办法公用的，大家思考下这是为什么呢？其实原因很简单，在服务器端需要通过请求路径，找到路由组件，而在客户端需通过浏览器中的网址，找到路由组件，是完全不同的两套机制，所以这部分代码是肯定无法公用。我们来看看在 SSR 中，前后端路由的实现代码。
## 客户端路由（entry-client.js）：
``` js
const createApp = (Component) => {
  // 获取服务端初始化的state，创建store
  const initialState = window.__INITIAL_STATE__;
  const store = createStore(initialState);

  const App = () => {
    return (
      <Provider store={store}>
        <BrowserRouter>
          <Component />
        </BrowserRouter>
      </Provider>
    );
  };
  return <App />;
};

ReactDOM.hydrate(createApp(Root), document.getElementById('app'));
```
客户端路由代码非常简单，大家一定很熟悉，BrowserRouter 会自动从浏览器地址中，匹配对应的路由组件显示出来。
## 服务端路由（entry-server.js）：
``` js
const createApp = (context, url, store) => {
  const App = () => {
    return (
      <Provider store={store}>
        <StaticRouter context={context} location={url}>
          <Root />
        </StaticRouter>
      </Provider>
    );
  };
  return <App />;
};

export {
  createApp,
  createStore,
  router
};
```
服务器端路由代码相对要复杂一点，需要你把 location（当前请求路径）传递给 StaticRouter 组件，这样 StaticRouter 才能根据路径分析出当前所需要的组件是谁。（PS：StaticRouter 是 React-Router 针对服务器端渲染专门提供的一个路由组件。）

# Webpack 配置
对于一个 React 应用来说，路由一般是整个程序的执行入口。在 SSR 中，服务器端的路由和客户端的路由不一样，也就意味着服务器端的入口代码和客户端的入口代码是不同的。换句话说，Entry 的配置肯定是不同的。所以我们需要打包出两份代码，一份由服务端执行渲染html，一份由浏览器执行，两份代码里大部分代码都可以在服务端和客户端执行（不然怎么能叫同构应用呢）。
## 目录结构
![](https://upload-images.jianshu.io/upload_images/13276697-fd85a3da3c6ec8a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
``` html
|---build   - 工程构建目录，包含了，开发，测试以及上线中所用到的构建脚本及插件
    |---plugins   - 构建中所用到的自定义插件
    |---webpack.base.config.js    - 通用 webpack 配置
    |---webpack.dev.config.js     - 客户端开发环境 webpack 配置
    |---webpack.prod.config.js    - 客户端生产环境 webpack 配置
    |---webpack.dll.config.js     - 客户端 webpack dll 配置
    |---webpack.server.config.js  - 服务端 webpack 配置
    |---webpack.test.config.js    - 客户端测试环境 webpack 配置
|---config  - 构建配置文件，包含静态资源路径，接口代理地址等
```
## 服务端配置
这么多 webpack 配置文件里，其实只有 webpack.server.config.js 是给服务端用的，因为服务端配置无需区分开发环境还是生成环境。
服务端运行于 node 中，不支持 babel，不支持样式，同时也不支持一些浏览器全局对象如 window、document，对于 babel 使用 babel-loader 进行转换，对于样式使用插件提取出来，服务端只运行 js 生成 html 片段，再根据客户端清单将 css 插入到 head 中。
``` js
'use strict';
const utils = require('./utils');
const webpack = require('webpack');
const config = require('../config');
const nodeExternals = require('webpack-node-externals');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const SSRServerPlugin = require('./plugins/server-plugin');

module.exports = {
  mode: 'production',
  entry: {
    app: utils.resolve('src/entry-server.js')
  },
  output: {
    path: config.build.assetsRoot,
    filename: 'entry-server.js',
    libraryTarget: 'commonjs2'
  },
  target: 'node', // 指定node运行环境
  devtool: config.server.devtool,
  externals: [
    nodeExternals({
      whitelist: [ /\.(css|sass)$/ ] // 忽略css，让webpack处理
    })
  ],
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              babelrc: false,
              presets: ['@babel/preset-env', '@babel/preset-react'],
              plugins: [
                'dynamic-import-node',
                '@loadable/babel-plugin'
              ]
            }
          }
        ],
        exclude: /node_modules/
      },
      ...utils.styleLoaders({
        sourceMap: config.build.productionSourceMap,
        extract: true,
        usePostCSS: true
      })
    ]
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.server.env
    }),
    new MiniCssExtractPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // 这是将服务器的整个输出
    // 构建为单个 JSON 文件的插件
    new SSRServerPlugin({
      filename: 'server-bundle.json'
    })
  ]
};
```
## 客户端配置
客户端配置是有必要区分开发环境和生产环境的（我们先忽略测试环境配置），开发环境我们增加热更新、source-map、eslint 等功能，而生产环境我们将侧重 webpack 分包及样式提取上。
### 开发环境配置
``` js
'use strict';
const utils = require('./utils');
const webpack = require('webpack');
const config = require('../config');
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base.config');

// add hot-reload related code to entry chunks
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = [
    'react-hot-loader/patch',
    'webpack-hot-middleware/client'
  ].concat(baseWebpackConfig.entry[name]);
});

const createLintingRule = () => ({
  test: /\.(js|jsx)$/,
  loader: 'eslint-loader',
  enforce: 'pre',
  include: [utils.resolve('src')],
  options: {
    formatter: require('eslint-friendly-formatter'),
    emitWarning: !config.dev.showEslintErrorsInOverlay
  }
});

module.exports = merge(baseWebpackConfig, {
  mode: 'development',
  output: {
    path: config.dev.assetsRoot,
    filename: utils.assetsPath('js/[name].js'),
    publicPath: config.dev.assetsPublicPath
  },
  module: {
    rules: [
      createLintingRule(),
      ...utils.styleLoaders({ sourceMap: config.dev.cssSourceMap, usePostCSS: true })
    ]
  },
  devtool: config.dev.devtool,
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin()
  ]
});
```
### 生产环境配置
``` js
'use strict';
const utils = require('./utils');
const webpack = require('webpack');
const config = require('../config');
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base.config');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');

module.exports = merge(baseWebpackConfig, {
  mode: 'production',
  output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    publicPath: config.build.assetsPublicPath,
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true,
      usePostCSS: true
    })
  },
  devtool: config.build.devtool,
  optimization: {
    runtimeChunk: {
      name: 'manifest'
    },
    splitChunks: {
      cacheGroups: {
        chunks: 'initial',
        vendor: {
          test: /([\\/]node_modules[\\/])/,
          name: 'vendor',
          chunks: 'all'
        },
        'async-vendors': {
          test: /[\\/]node_modules[\\/]/,
          minChunks: 3,
          chunks: 'async',
          name: 'async-vendors'
        }
      }
    },
    minimizer: [
      new UglifyJsPlugin({
        sourceMap: true
      }),
      new OptimizeCSSAssetsPlugin({
        cssProcessorOptions: {
          map: { inline: false }
        }
      })
    ]
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': config.build.env
    }),
    new MiniCssExtractPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // keep module.id stable when vendor modules does not change
    new webpack.HashedModuleIdsPlugin(),
    // enable scope hoisting
    new webpack.optimize.ModuleConcatenationPlugin()
  ]
});
```
# Node 服务
在 SSR 架构中，一般 Node 只是一个中间层，用来做 React 代码的服务器端渲染，而 Node 需要的数据通常由 API 服务器单独提供。
这样做一是为了工程解耦，二也是为了规避 Node 服务器的一些计算性能问题。

服务器端渲染时，直接请求 API 服务器的接口获取数据没有任何问题。但是在客户端，就有可能存在跨域的问题了，所以，这个时候，我们需要在服务器端搭建 Proxy 代理功能，客户端不直接请求 API 服务器，而是请求 Node 服务器，经过代理转发，拿到 API 服务器的数据。

这里你可以通过 express-http-proxy 这样的工具帮助你快速搭建 Proxy 代理功能，但是记得配置的时候，要让代理服务器不仅仅帮你转发请求，还要把 cookie 携带上，这样才不会有权限校验上的一些问题。

在 server/index.js 中我们使用 express 启动 Node 服务，处理任何 get 请求。从服务端打包后的 js 中获取根组件，读取的 index.html 模板，调用 renderToString 传入根组件，将返回的 html 字符串替换掉模板中占位符，返回到客户端。
## server/index.js
``` js
const config = require('../config');
const opn = require('opn');
const chalk = require('chalk');
const fs = require('fs');
const express = require('express');
const ServerRenderer = require('./renderer');
const app = express();

// 静态资源映射到dist路径下
app.use(express.static('dist'));

const isProd = process.env.NODE_ENV === 'production';
let renderer;
let readyPromise;
let template = fs.readFileSync('./index.html', 'utf-8');
if (isProd) {
  let bundle = require('../dist/server-bundle.json');
  let stats = require('../dist/loadable-stats.json');
  renderer = new ServerRenderer(bundle, template, stats);
} else {
  readyPromise = require('./dev-server')(app, (bundle, stats) => {
    renderer = new ServerRenderer(bundle, template, stats);
  });
}

const render = (req, res) => {
  console.log(chalk.cyan('visit url: ' + req.url));

  renderer.renderToString(req).then(({ error, html }) => {
    if (error) {
      if (error.url) {
        res.redirect(error.url);
      } else if (error.code) {
        res.status(error.code).send('error code：' + error.code);
      }
    }
    res.send(html);
  }).catch(error => {
    console.log(error);
    res.status(500).send('Internal server error');
  });
};

app.get('*', isProd ? render : (req, res) => {
  // 等待客户端和服务端打包完成后进行render
  readyPromise.then(() => render(req, res));
});

const port = process.env.PORT || config.dev.port;
const autoOpenBrowser = !!config.dev.autoOpenBrowser;
const uri = 'http://localhost:' + port;
const ip = 'http://' + require('ip').address() + ':' + port;

app.listen(port, function (err) {
  if (err) {
    console.log(err);
    return;
  }

  console.log(chalk.cyan('\n' + '- Local: ' + uri + '\n'));
  console.log(chalk.cyan('- On your Network: ' + ip + '\n'));

  if (autoOpenBrowser) {
    opn(uri);
  }
});
```
我们仍需区分开发环境和生产环境，因为开发环境我们需要增加热更新的代码逻辑。
在 server/index.js 中判断当前环境是否是生产环境，生产环境保持原有的逻辑，非生产环境使用 webpack-dev-middleware 和 webpack-hot-middleware 进行客户端热更新。
服务端热更新则需要使用 outputFileSystem 属性指定打包输出的文件系统为内存文件系统，再使用watch函数检测文件变动，打包完成后同样从内存中获取 server-bundle.json 文件内容。
## server/dev-server.js
``` js
const path = require('path');
const webpack = require('webpack');
const MFS = require('memory-fs');
const clientConfig = require('../build/webpack.dev.config');
const serverConfig = require('../build/webpack.server.config');

module.exports = function setupDevServer(app, callback) {
  let bundle;
  let loadableStats;
  let resolve;
  const readyPromise = new Promise(r => { resolve = r; });
  const update = () => {
    if (bundle && loadableStats) {
      callback(bundle, loadableStats);
      resolve(); // resolve Promise让服务端进行render
    }
  };

  const readFile = (fs, fileName) => {
    return fs.readFileSync(path.join(clientConfig.output.path, fileName), 'utf-8');
  };

  // 客户端打包
  const clientCompiler = webpack(clientConfig);

  // 使用webpack-dev-middleware中间件服务webpack打包后的资源文件
  const devMiddleware = require('webpack-dev-middleware')(clientCompiler, {
    publicPath: clientConfig.output.publicPath,
    logLevel: 'warn'
  });
  app.use(devMiddleware);

  clientCompiler.hooks.done.tap('done', stats => {
    const info = stats.toJson();
    if (stats.hasWarnings()) {
      console.warn(info.warnings);
    }

    if (stats.hasErrors()) {
      console.error(info.errors);
      return;
    }
    loadableStats = JSON.parse(readFile(devMiddleware.fileSystem, 'loadable-stats.json'));
    update();
  });

  // 热更新中间件
  app.use(require('webpack-hot-middleware')(clientCompiler));

  // 监视服务端打包入口文件，有更改就更新
  const serverCompiler = webpack(serverConfig);
  // 使用内存文件系统
  const mfs = new MFS();
  serverCompiler.outputFileSystem = mfs;
  serverCompiler.watch({}, (err, stats) => {
    const info = stats.toJson();
    if (stats.hasWarnings()) {
      console.warn(info.warnings);
    }

    if (stats.hasErrors()) {
      console.error(info.errors);
      return;
    }

    bundle = JSON.parse(readFile(mfs, 'server-bundle.json'));
    update();
  });

  return readyPromise;
};
```
# 启动
最后我们对 package.json 进行修改
``` bash
"scripts": {
    "dev": "node server/index.js",
    "dll": "webpack --config build/webpack.dll.config.js",
    "start": "cross-env NODE_ENV=production node server/index.js",
    "build": "rimraf dist && npm run build:client && npm run build:server",
    "build:client": "webpack --config build/webpack.prod.config.js",
    "build:server": "webpack --config build/webpack.server.config.js"
},
```
## 本地启动工程

``` bash
# 安装依赖
npm install

# 初次配置需要执行，生成 vendor.dll 文件
npm run dll

# 启动工程
npm run dev
```

## 执行打包

```bash
# 使用生产环境配置打包
npm run build

# 启动工程
npm run start
```
# 总结
到这里，整个 React 同构体系中关键知识点的原理就串联起来了。
当然，本文所涉及的同构知识还是非常有限，下一篇中我还将介绍同构中异步数据的获取 + Redux 的使用。
如果你看了文章觉得云里雾里，可以将代码自行 fork 下来跑一跑，我相信可以帮助到你。
工程地址：[https://github.com/LiuLingyang/react-ssr-framework](https://github.com/LiuLingyang/react-ssr-framework)
