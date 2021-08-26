
>早在三个月，某人带着我为他的公务员同事只做了个桌面应用，用的就是这个[electron-react-boilerplate](https://github.com/electron-react-boilerplate/electron-react-boilerplate)这个模板，踩了不少坑。当时想自己尝试引入在electron中引入react，也尝试过在react中引入electron均无果，直到我看了一个教学视频，现在记录下

## 初始化项目
1、 使用npm命令初始化项目
```
npm init -y 
npm install electron -D
```
2、建立`main`和`render`文件夹
众所周知，electron是由主线程和渲染进程构成，中间通过IPC进行通信（具体这个在后面讲）。我们在`render`文件夹建立`src`和`pages`两个文件夹,在`src`文件夹下面输入
```
create-react-app main
```
3、启动命令修改
在外【package.json】中的`scripts`修改启动脚本
```
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start:render": "cd renderer/src/main && npm run start"
  },
```
4、启动react项目
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/329077525a3a43b3b8b31f483350a3be~tplv-k3u1fbpfcp-zoom-1.image)

5、启动electron项目
脚本修改
```
 "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start:main": "electron .",
    "start:render": "cd renderer/src/main && npm run start"
  },
```
## 项目结构
```
electron-react-template
├─package.json   // 外面的packeage.json
├─result.txt
├─renderer    // 渲染进程
|    ├─src
|    |  ├─main
|    |  |  ├─.gitignore
|    |  |  ├─README.md
|    |  |  ├─config-overrides.js
|    |  |  ├─package.json   // 里面的packeage.json
|    |  |  ├─yarn.lock
|    |  |  ├─src
|    |  |  |  ├─App.css
|    |  |  |  ├─App.js
|    |  |  |  ├─App.test.js
|    |  |  |  ├─index.css
|    |  |  |  ├─index.js
|    |  |  |  ├─logo.svg
|    |  |  |  ├─serviceWorker.js
|    |  |  |  └setupTests.js
|    |  |  ├─public
|    |  |  |   ├─favicon.ico
|    |  |  |   ├─index.html
|    |  |  |   ├─logo192.png
|    |  |  |   ├─logo512.png
|    |  |  |   ├─manifest.json
|    |  |  |   └robots.txt
|    ├─pages
├─main   //  主进程
|  └index.js

```

## 在react中引入electron
在`App.js`这样子写报错
```
import { ipcRender } from 'electron';
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2247ffe901b4715b3be233738dbe392~tplv-k3u1fbpfcp-zoom-1.image)

有两种方法

### 使用window.require('electron')
```
const { ipcRender } = window.require('electron')
```
### 修改target（篡改cra）
具体可以参照: [target](https://webpack.js.org/configuration/target/#root)

在【内packerage.json】安装以下依赖
```
npm i customize-cra react-app-rewired -D
```
[react-app-rewired](https://www.npmjs.com/package/react-app-rewired)

[customize-cra](https://www.npmjs.com/package/customize-cra)

1、写入`config-overrides.js`(放在react根目录下面)
```
const { override } = require('customize-cra')
function addRendererTarget(config) { 
    config.target = 'electron-renderer'
    return config;
}
module.exports = override(addRendererTarget)
```
2、修改react项目的scripts
```
// 之前
  "start": "BROWSER=none react-scripts start",
// 之后
    "start": "BROWSER=none react-app-rewired start"
```
3、 App.js引入ipcRender
可以看到引入成功
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7001b3220ef04f0c8dec006a61e14893~tplv-k3u1fbpfcp-zoom-1.image)

## 命令组合修改
我们可以看到使用两个命令启动项目特别麻烦，可以把它们俩组合在一起
在【外package.json】安装
```
npm i concurrently wait-on -D
```
[concurrently](https://www.npmjs.com/package/concurrently)
[wait-on](https://www.npmjs.com/package/wait-on)

修改启动命令【外package.json】
```
 "start": "concurrently \"npm run start:render\" \"wait-on http://localhost:3000 && npm run start:main\" ",
```
意思是等`npm run start:render`启动完毕之后再启动`npm run start:main`
输入`npm run start`可以访问
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a13abf8606c4d99b8c2e2fe237b91b9~tplv-k3u1fbpfcp-zoom-1.image)

## 其他的tips
- 启动react项目的时候可以设置不自动跳转浏览器
```
"start": "BROWSER=none react-scripts start"
```
- 打开electron调试工具
```
 win.webContents.openDevTools()
```

## 其他模板
[electron-vue](https://github.com/SimulatedGREG/electron-vue)

[svelte-template-electron](https://github.com/Rich-Harris/svelte-template-electron)

[angular-electron](https://github.com/maximegris/angular-electron)


## 项目地址
[https://github.com/MissNanLan/electron-react-template](https://github.com/MissNanLan/electron-react-template)

## 存在的问题

因为有两个package.json，所以我得安装两边依赖，很不方便
