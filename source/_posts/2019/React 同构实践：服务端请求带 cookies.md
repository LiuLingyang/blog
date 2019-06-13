title: React 同构实践：服务端请求带 cookies
date: 2019-06-13
author: LiuLingyang
categories: React
---

# 前言
前段时间搭完 React 同构框架后，自以为很完美，还自我陶醉了好几天。结果，在接公司单点登录系统的时候立马现了原形。
公司的单点登录是用 cookie 实现的，传统的 spa 系统 cookie 由浏览器自动携带，不存在任何问题。
而 ssr ，是在服务器请求的接口，默认情况肯定带不了浏览器的 cookie，然而一些登录后的页面数据又必须要 cookie，这可怎么办？
# 思路
将 cookies 作为参数注入到组件的 loadData 方法，然后用传参数的方法把 cookies 传给 api。
## 1. cookie-parser 中间件解析 cookies
在项目启动文件中（server/index.js）引入 cookie-parser 中间件解析 cookies
``` js
const cookieParser = require('cookie-parser');

// 解析 cookie
app.use(cookieParser());
```

## 2. 将 cookies 作为参数传给 loadData 方法
``` js
let matchs = matchRoutes(router, request.path);
promises = matchs.map(({ route, match }) => {
  const loadData = route.loadData;
  // 携带上cookie，合并到 params 中，axios 中处理
  return loadData ? loadData(store, Object.assign(match.params, request.query, { cookies: request.cookies })) : Promise.resolve(null);
});
```

## 3. loadData 传给异步 action
``` js
loadData(store, params) {
  return store.dispatch(fetchTopList(params));
}
```

## 4. 发起请求
在异步 action 处理函数中带着 cookies 发起异步请求
``` js
export function fetchTopList(data) {
  return (dispatch) => {
    return getTopList(data).then(result => {
      dispatch(setTopList(result.topList));
    });
  };
}
```

## 5. axios 封装
在 axios 中解析出 params 携带的 cookies，并加到 axios 的 headers 里
``` js
const parseCookie = cookies => {
  let cookie = '';
  Object.keys(cookies).forEach(item => {
    cookie+= item + '=' + cookies[item] + '; ';
  });
  return cookie;
};

request(method) {
    return function (url, data = {}, opts = {}) {
      // 处理 cookie
      let cookies;
      if (data.cookies) {
        cookies = data.cookies;
        delete data.cookies;
        opts = Object.assign(opts, {
          headers: {
            Cookie: parseCookie(cookies)
          }
        });
      }

      return axiosCache[method](url, {
        ...opts,
        params: data
      });
    };
}
```
# 结尾
至此，就完成了服务端请求携带 cookie 的问题，但总觉得实现上有点麻烦，不知道是否有更好的办法？
希望大神能不吝指教...