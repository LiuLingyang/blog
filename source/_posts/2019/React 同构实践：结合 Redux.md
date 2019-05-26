title: React 同构实践：结合 Redux
date: 2019-05-25
author: LiuLingyang
categories: React
---

# 前言
上一篇文章中，我们介绍了 React 同构的基本原理，但没有涉及数据获取方面。本章我们将结合 Redux 来讲解同构应用中服务端如何异步获取数据。
如果你对 Redux 还不够熟悉，请先移步 [http://cn.redux.js.org/](http://cn.redux.js.org/)
# 数据预取
客户端渲染中，异步数据结合 Redux 的使用方式遵循下面的流程：
* 创建 Store
* 根据路由显示组件
* 派发 Action 获取数据
* 更新 Store 中的数据
* 组件 Rerender

而在服务器端，页面一旦确定内容，就没有办法 Rerender 了，这就要求组件显示的时候，就要把 Store 的数据都准备好，所以服务器端异步数据结合 Redux 的使用方式，流程是下面的样子：
* 创建 Store
* 匹配请求路由
* 获取数据
* 结合数据和组件生成 HTML

下面我们分析下服务器端渲染这部分的流程。
## 创建 Store
``` js
export default (initialState) => {
  return createStore(
    reducer,
    initialState,
    compose(
      applyMiddleware(thunkMiddleware),
      (process.env.NODE_ENV !== 'server' && window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()) || compose
    )
  );
};

const store = createStore({});
```
这里需要注意的是服务端每次 renderToString 都需要创建一个全新的空 Store。

## 匹配请求路由
服务端在正式渲染前，如果某些内容是通过异步接口请求获取的，那么就需要在 renderToString 前加载数据，但一般不同的路由都会对应不同的异步请求，所以这里我们需要添加路由匹配的逻辑。
``` js
const { matchRoutes } = require('react-router-config');

// 匹配路由
let matchs = matchRoutes(router, request.path);
promises = matchs.map(({ route, match }) => {
  const loadData = route.loadData;
  // match.params获取匹配的路由参数
  return loadData ? loadData(store, Object.assign(match.params, request.query)) : Promise.resolve(null);
});
```
## 获取数据
这里请求数据需要使用[异步 Action](https://link.juejin.im/?target=http%3A%2F%2Fwww.redux.org.cn%2Fdocs%2Fadvanced%2FAsyncActions.html)，使用 redux-thunk 中间件就可以dispatch函数了。
而请求数据一般两种做法，第一种直接挂在了路由上，另一种做法是把请求数据的方法放到对应的组件上定义成静态方法。
但这里我用了 loadable 动态加载组件后，定义在组件上的静态方法在服务端就取不到，所以只能采用第一种做法（不知道大家有没有遇到这个问题？）。
``` js
{
    path: '/top-list',
    component: loadable(() => import('../containers/TopList')),
    exact: true,
    loadData(store) {
      return store.dispatch(fatchTopList());
    }
  }
```
## 返回 HTML
获取数据，更新 store 后，可以用 renderToString 生成 html 了。
``` js
// resolve所有loadData
Promise.all(promises).then(() => {
  // 异步数据请求全部完成后进行render
  render()；
})；
```
这里的 render 方法核心就是 renderToString，因为还有一些其他逻辑，所以封装了一层。
# 同步数据
客户端渲染出来的 html 内容要和服务端返回的 html 内容一致，这就需要保证客户端的数据和服务端的数据是一致的。也就是说服务端获取数据生成 store 后，需要同步到客户端。
服务端调用 store.getState()，获取到 state 后，通过 window.__INITIAL_STATE__ 保存在客户端。
``` html
<script type="text/javascript">
  window.__INITIAL_STATE__ = ${JSON.stringify(store.getStat())}
</script>
```
# 客户端获取数据
服务端将数据存到 window.__INITIAL_STATE__ 后，客户端需要在 createStore 的时候获取初始化的 state。
``` js
  // 获取服务端初始化的state，创建store
  const initialState = window.__INITIAL_STATE__;
  const store = createStore(initialState);

  const App = () => {
    return (
      <Provider store={store}>
        <Router>
          <Component />
        </Router>
      </Provider>
    );
  };
  return <App />;
```
# 总结
本章主要讲解结合 Redux 情况下的 React 同构应用，所涉及的知识点还是比较多的，单单一篇文章也很难涉及全部的知识点，实际开发过程中更是容易
出现千奇八怪的问题。实践出真知，还是要亲自写一下才能深入了解。