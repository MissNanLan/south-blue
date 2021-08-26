## 前言
>这篇文章简单介绍rollup，重点是后面的例子

## rollup简介
### 概览
>Rollup is a module bundler for JavaScript which compiles small pieces of code into something larger and more complex, such as a library or application. It uses the new standardized format for code modules included in the ES6 revision of JavaScript, instead of previous idiosyncratic solutions such as CommonJS and AMD. ES modules let you freely and seamlessly combine the most useful individual functions from your favorite libraries. This will eventually be possible natively everywhere, but Rollup lets you do it today

简单的理解就是rollup可以打包成库或者是应用。 常用来打包库，与webpack相比较的话，个人感觉使用rollup简单些。
### 按照和核心配置介绍

```
// 全局安装
npm install --global rollup 
```

```
// 本地安装
npm install rollup --save-dev
```
#### input
文件入口
#### output
##### file: 输出名字
##### format:输出格式
- iife

   一个自动执行的功能，使用script引入
- amd

  异步模块定义，用于像RequireJs这样的模块加载器
- cjs

 `CommonJs`适用于Node和Browserify/Webpack
- umd

  通用模块定义，以`amd`、`cjs`和`iife`为一体
- es/esm

  输出为es模块，可以通过`<script type=module></script>`标题引入
 
- system

 SystemJs加载器格式

```
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```
这是一个很简单`rollup.config.js`文件，安装`rollup`之后可直接运行
```
rollup -c rollup.config.js/rollup --config rollup.config.js 
```
rollup官网上虽然提供了很多配置参数[big-list-of-options](https://www.rollupjs.org/guide/en/#big-list-of-options)，但是实际用得并不很多，除了`input`、`output`这些两个基本配置参数，另外的就是插件的应用了

### 常用的插件
#### @rollup/plugin-node-resolve
[node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve)
处理外部的依赖，比如node_modules
#### @rollup/plugin-commonjs
[commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs)
 将 common js 模块转成es6
#### @rollup/plugin-babel
 [babel](https://github.com/rollup/plugins/tree/master/packages/babel)
 用Babel编译文件.配置babel的时候需要在项目根目录下创建`.babelrc`
 ```
 // .babelrc
 {
  "presets": [
    ["@babel/env", {"modules": false}]
  ]
}
 ```
#### rollup-plugin-terser
 [terser](https://github.com/TrySound/rollup-plugin-terser), 用Terser压缩

#### rollup-plugin-uglify
[uglify](https://github.com/TrySound/rollup-plugin-uglify),用UglifyJS压缩

#### @rollup/plugin-typescript
[typescript](https://github.com/rollup/plugins/tree/master/packages/typescript),用在rollup和typescript之间的无缝集成
### 其他关于rollup的连接

[ rollup 插件库](https://github.com/rollup/awesome)

[learn-rollup](https://www.learnwithjason.dev/blog/learn-rollup-js/)

[rollup.js官网](https://rollupjs.org/)

## 动手实战
### 案例一
```
// rollup.config.js
import resolvePlugin from "rollup-plugin-node-resolve";
import babelPlugin from "rollup-plugin-babel";
import typescript from "@rollup/plugin-typescript";
import commonjs from "@rollup/plugin-commonjs";
import { uglify } from "rollup-plugin-uglify";
export default [
  {
    input: "packages/index.ts",
    output: {
      file: "core.umd.js", // 导出文件
      format: "umd", // 打包文件支持的形式
      name: "core",
    },
    plugins: [
      resolvePlugin(),
      typescript(),
      commonjs(),
      babelPlugin({
        exclude: "node_modules/**",
        runtimeHelpers: true,
      }),
      uglify()
    ],
  }
];

```
然后在`package.json`下面的script加入
```
"scripts": {
    "clear": "rimraf lib",
    "build": "rollup -c rollup.config.js",
    "build:watch": "rollup -c rollup.config.js --watch",
    "build:prod": "cross-env NODE_ENV=production rollup -c rollup.config.js"
  }
```
我试着输出不同的格式,看看他们的区别

umd（目前库的通用做法）

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcc3040a923443b2b849159024d0face~tplv-k3u1fbpfcp-zoom-1.image)

iife

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c267b511a5374d0a898fa30c18f10705~tplv-k3u1fbpfcp-zoom-1.image)

amd

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b22c29aa6ab4bc88ec72de0f2f65d65~tplv-k3u1fbpfcp-zoom-1.image)

cjs

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e065ce5e90b548bb89a1d6b05f71b558~tplv-k3u1fbpfcp-zoom-1.image)

以上是没有经过压缩的，由于某些原因，代码不是很方便完整地放出来
### 案例2

早就想用原生的方式写个库发个`npm`包，于是动手实战了下，用了`jquery`
#### 效果展示
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/722a5699f5674e0fa805a5a96b07cdb5~tplv-k3u1fbpfcp-zoom-1.image)
#### 主要的代码
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/898442b8ff5d49dc8bfde4ad1bfbb988~tplv-k3u1fbpfcp-zoom-1.image)

这段代码和上文提到的输出`umd`格式是类似的，唯一不同的地方在于我这里引入`jquery`。如果是`script`引入的话，那么必须要在代码里引入`jquery`，因为它进的是最后一个`else`,挂载到全局上面

```
(function (g, fn) {
  if (typeof require !== "undefined") {
    if (g.$ === undefined) {
      g.$ = require("./jquery.min.js");
    }
  }
  var $ = g.$;
  if (typeof define === "function" && define.amd) {
    define(function () {
      return fn($);
    });
  } else if (typeof module !== "undefined" && module.exports) {
    module.exports = fn($);
  } else {
    g.Tab = fn($);
  }
 }(typeof window !== "undefined" ? window : this, function ($) {
  var Tab = function(container,options){
  // container 是挂载的id，options是传入的参数
  }
  Tab.prototype = {
  //
  }
   return Tab
 }
```
#### rollup打包
```
import { uglify } from "rollup-plugin-uglify";
export default [
  {
    input: "lib/tab.js",
    output: {
      file: "tab.min.js", // 导出文件
      format: "umd", // 打包文件支持的形式
      name: "tab",
    },
    plugins: [
      uglify()
    ],
  }
];
```
这里就用了一个压缩的插件。
#### 踩坑
1、 我尝试用import引入，在使用的时候报错，`new Tab（）`这句代码报Tab找不到。解决办法是新建了一个index.js的文件
```
module.exports = require('./lib/tab.min.js');
```
然后`package.json`的`main`由`lib/tab.min.js`改成了`index.js`才没有报错。后面我仔细一想完全没有必要用`rollup`,因为我代码都写得很详尽了，后面入口改成`lib/tab.js`就可以了

2、 发布npm报的时候遇到`2FA`的验证 [双因素验证](https://www.ruanyifeng.com/blog/2017/11/2fa-tutorial.html)，chorme装了个`身份验证器插件`

3、注意发布npm包的时候指定`npm`包为公用
```
npm publish --access=public
```
#### 小技巧
公司一般都会搭建私服,如果不想发布公用的包，小范围测试的话,可以使用`npm link`。现在你的库中`npm link `，在使用项目中 npm link + 包名
具体参照[npm 私有模块的3种方法](https://www.jianshu.com/p/a9540d9f8d9c)

大家尽情吐槽
[源码地址](https://github.com/MissNanLan/toolkit)

## 规范commit&发布relase版本

因为第一次写npm包，所以在提交下用了一些插件来规范自己的提交，知名的开源库基本上都是这么做的。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/989fdbae6cbb4572be43ee77f4b50d5e~tplv-k3u1fbpfcp-zoom-1.image)
下面我总结下我用了哪些工具

### 使用husky+commlint 规范提交信息
1、 安装
```
npm install husky --save-dev
npm install --save-dev @commitlint/cli @commitlint/config-angular
```

2、配置

在package.json下面加入以下配置，
```
  "husky": {
    "hooks": {
      "pre-commit": "echo checking commit msg...",  // commit之前需要做些什么
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"  // HUSKY_GIT_PARAMS是一个环境变量，就是commit信息
      // commitlint是在做检测
    }
  },
```
除了在packge.json里面加入配置之外。还可以
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7a2cd608c5d4cb68463653aa7d8f86a~tplv-k3u1fbpfcp-zoom-1.image)

3、commlint

就是一个commit 规范的检测工具，根据[conventional commit format](https://www.conventionalcommits.org/en/v1.0.0/#summary)来定义的。我发现这个还是很有必要遵循的，尤其是在大厂和写开源库的时候的
```
# 主要type
feat:     增加新功能
fix:      修复bug

# 特殊type
docs:     只改动了文档相关的内容
style:    不影响代码含义的改动，例如去掉空格、改变缩进、增删分号
build:    构造工具的或者外部依赖的改动，例如webpack，npm
refactor: 代码重构时使用
revert:   执行git revert打印的message

# 暂不使用type
test:     添加测试或者修改现有测试
perf:     提高性能的改动
ci:       与CI（持续集成服务）有关的改动
chore:    不修改src或者test的其余修改，例如构建过程或辅助工具的变动

```

以上来自来自于[Git commit message 规范](https://juejin.im/post/6844903871832145927)。貌似看到有些开源库是利用图标去做的

4、 简单介绍下commitlint的规则

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- type就是第三点的截图 
- scope是修改的返回，比如
```
fix(src): fix login bug
```
- description 是commit 信息
- body和footer都是可选的，body是commit的详细描述，说明代码提交的详细说明。footer提交是不兼容变更或关闭缺陷，则Footer必需，否则可以省略

5、 commit配置

新建了一个`commitlint.config.js`的文件
```
// commitlint.config.js
module.exports = {
    // 继承默认配置
    extends: [
      "@commitlint/config-angular"
    ],
    // 自定义规则
    rules: {
      'type-enum': [2, 'always', [
        'upd',
        'feat',
        'fix',
        'refactor',
        'docs',
        'chore',
        'style',
        'revert',
      ]],
      'header-max-length': [0, 'always', 72]
    }
  };
  
```
6、验证结果
- fails
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5b53834868748ceb86945687cb2f151~tplv-k3u1fbpfcp-zoom-1.image)
- success
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47502ecdce8945ad9d604547a1b5d952~tplv-k3u1fbpfcp-zoom-1.image)

7、推荐阅读

[husky](https://github.com/typicode/husky)

[commitlint](https://github.com/conventional-changelog/commitlint)

[介绍commit一些场景](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional#type-enum)

[conventionalcommits](https://www.conventionalcommits.org/en/v1.0.0/#summary)

[Angular规范](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)

[Cz工具集使用介绍 - 规范Git提交说明](https://juejin.im/post/6844903831893966856#heading-7)

[从 Commit 规范化到发布自定义 CHANGELOG 模版](https://juejin.im/post/6844903888072654856#heading-3)


### 使用`standard-version` 自动生成CHANGELOG

1、 安装
```
npm i --save-dev standard-version
```
2、 配置
```
// package.json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "rollup -c rollup.config.js",
    "release": "standard-version"
  },
```
3、运行
```
npm run release
```
4、 打一个tag
```
git push --follow-tags origin master
```
注意`tag`会自己生成，并且会更新`package.json`里面的`version`
还可以运行`npm publish`发布到`npm`

5、 发布一个release版本

第一步
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e43a82ec223449b48f18d80645367394~tplv-k3u1fbpfcp-zoom-1.image)
第二步
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6595b5f63f66497ab9876f6b9b77b24c~tplv-k3u1fbpfcp-zoom-1.image)

4、 推荐阅读

[standard-version](https://github.com/conventional-changelog/standard-version)

[git commit 规范校验配置和版本发布配置](https://juejin.im/post/6844903857718312967#heading-7)

### 总结

就是利用了`husky`、`commitlint`、`standard-version`这些插件去规范提交的日志以及自动生成`CHANGELOG`，还有一个更为强大的工具`commitizen`。工具万千，自己习惯和适用于项目实际情况就好

才疏学浅，文章有点长，难免有疏漏之处，请及时斧正，以免误导别人



