title: Electron 初探
date: 2020-10-13
author: LiuLingyang
categories: 多端

---

# 前言

随着浏览器，移动设备变得越来越强大，被移动和 web 应用取代的桌面应用呈稳定下滑趋势。
但是桌面应用依然有它得天独厚的优势--常驻、留存、离线等。
传统开发方式（C++，C#，Objective-c 等）费时又费力，基于 web 技术进行桌面应用的开发渐渐变成了主流（只需要写一份代码，打包出来的应用可以在 windows，linux，mac OS 上面运行，“write once,run everyWhere”）。

# Electron 是什么？

使用 JavaScript，HTML，CSS 和 Node.js 构建跨平台的桌面应用程序。
可以简单的理解为 Electron 为 web 项目套上了 Node.js 环境的壳，使得我们可以调用 Node.js 的丰富的 API。这样我们可以用 JavaScript 来写桌面应用，拓展很多我们在 web 端不能做的事情。

# 代表产品

VsCode，Atom 等

# 打造你的第一个 Electron 应用

```bash
# 官网已经有 electron-quick-start 仓库克隆下来
git clone https://github.com/electron/electron-quick-start
# 进入文件夹
cd electron-quick-start
# 安装依赖包并运行
npm install && npm start
```

![](https://upload-images.jianshu.io/upload_images/13276697-18c65266d81b1c07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 与 React 结合

1.使用 create-react-app 来创建 React 项目

```bash
npm install -g create-react-app
create-react-app electron5-react-demo
cd electron5-react-demo && npm start
```

2.集成 Electron 环境

```bash
npm install electron electron-builder electron-is-dev cross-env
touch public/electron.js
```

将[官网](https://github.com/electron/electron-quick-start/blob/master/main.js) main.js 代码拷贝进 electron.js

3.修改 package.json

```js
{
    "name": "electron5-react-demo",
    "version": "0.1.0",
    "private": true,
+    "main": "public/electron.js",
+    "homepage": "./",
}
```

```js
  "electron-start": "electron .",
  "electron-build": "electron-builder",
  "build": "yarn react-build && yarn electron-build",
  "start": "concurrently \"cross-env BROWSER=none yarn react-start\" \"wait-on http://localhost:3000 && electron .\""
```

4.启动

```bash
  npm start
```

看到如下界面，恭喜你，已经成功启动了：
![](https://upload-images.jianshu.io/upload_images/13276697-5e10f19ddf1c9376.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 与 Vue 结合

使用 vue-cli 来创建 vue-electron 工程：

```bash
# Install vue-cli and scaffold boilerplate
npm install -g vue-cli
vue init simulatedgreg/electron-vue my-project

# Install dependencies and run your app
cd my-project
npm install
npm run dev
```

# 主进程和渲染进程

## 主进程

Electron 运行 package.json 的 main 脚本的进程被称为主进程。 在主进程中运行的脚本通过创建 web 页面来展示用户界面。
一个 Electron 应用总是有且只有一个主进程。
主进程还负责应用程序的生命周期（app 打开退出）和一些 app 事件的监听，同时负责系统底层 API 的调用

## 渲染进程

Electron 使用了 Chromium 来展示 web 页面，每个页面运行在自己的渲染进程中。
使用 BrowserWindow 类开启一个渲染进程并将这个实例运行在该进程中，当一个 BrowserWindow 实例被销毁后，相应的渲染进程也会被终止。
渲染进程可以有多个，每个渲染进程都是相互独立的，它们只关心自己所运行的 web 页面

![](https://upload-images.jianshu.io/upload_images/13276697-9c61cc139b1b1110.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 进程间通信

## ipcMain 和 ipcRenderer

ipcMain 和 ipcRenderer 是两个好基友，通过这两个模块可以实现进程的通信。

ipcMain
在主进程中使用，用来处理渲染进程（网页）发送的同步和异步的信息

ipcRenderer
在渲染进程中使用，用来发送同步或异步的信息给主进程，也可以用来接收主进程的回复信息。

以上两个模块的通信，可以理解成发布订阅模式/观察者模式。

主进程

```js
// 主进程主动向渲染进程发送消息
win.webContents.send("something1", "消息来自主进程"); // webContents

const { ipcMain } = require("electron");
// 监听渲染程序发来的事件
ipcMain.on("something", (event, data) => {
  event.sender.send("something1", "我是主进程返回的值");
});
```

渲染进程

```js
const { ipcRenderer } = require("electron");
// 发送事件给主进程
ipcRenderer.send("something", "传输给主进程的值");

// 监听主进程发来的事件
ipcRenderer.on("something1", (event, data) => {
  console.log(data); // 我是主进程返回的值
});
```

## remote 模块

使用 remote 模块, 渲染进程可以调用主进程对象的方法, 而不必显式发送进程间消息。

```js
const { dialog, getGlobal } = require("electron").remote;
dialog.showMessageBox({
  type: "info",
  message: "在渲染进程中直接使用主进程的模块",
});

getGlobal("sharedObject"); // 获取全局共享变量
```

# 打包

目前，主流的打包工具有两个 electron-packager 和 electron-builder

## electron-packager

```bash
electron-packager <sourcedir> <appname> --platform=<platform> --arch=<arch> --out=out --icon=assets/app.ico --asar --overwrite --ignore=.git
```

- sourcedir 项目入口 根据 package.json 的目录
- appname 包名
- platform 构建平台 包含 darwin, linux, mas, win32 all
- arch 构建架构 包含 ia32,x64,armv7l,arm64
- out 打包后的地址
- icon 打包图标
- asar 是否生成 app.asar, 不然就是自己的源码
- overwrite 覆盖上次打包
- ignore 不进行打包的文件

## electron-builder

配置文件写在 package.json 中的 build 字段中

```js
"build": {
    "appId": "com.example.app", // 应用程序id
    "productName": "测试", // 应用名称
    "directories": {
        "buildResources": "build", // 构建资源路径默认为build
        "output": "dist" // 输出目录 默认为dist
    },
    "mac": {
        "category": "public.app-category.developer-tools", // 应用程序类别
        "target": ["dmg", "zip"],  // 目标包类型
        "icon": "build/icon.icns" // 图标的路径
    },
    "dmg": {
        "background": "build/background.tiff or build/background.png", // 背景图像的路径
        "title": "标题",
        "icon": "build/icon.icns" // 图标路径
    },
    "win": {
        "target": ["nsis","zip"] // 目标包类型
    }
}
```

这里我们使用 electron-builder 打包

```dash
npm run build
```

构建完成后，项目目录中会出现一个 dist 目录（第一次打包时间会比较长）：

```dash
.
├── builder-effective-config.yaml
├── electron5-react-demo-0.1.0-mac.zip
├── electron5-react-demo-0.1.0.dmg
├── electron5-react-demo-0.1.0.dmg.blockmap
├── latest-mac.yml
└── mac
    └── electron5-react-demo.app
```

目录中的 dmg 就是 Mac 上面的安装程序

# 最后

本文所有代码请戳 [github](https://github.com/LiuLingyang/electron5-react-demo) 了解更多
