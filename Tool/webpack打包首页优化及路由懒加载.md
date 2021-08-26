
### 前言
最近做了一个小型的vue的h5项目，发现在手机上运行的时候，第一次进去的首页比较慢，然后同事提了下可以在生产环境用cdn，于是我尝试了下
### 生产环境开启cdn
1、 在`vue.congfig.js`里面加入
```
const isProduction = process.env.NODE_ENV == "production";
 configureWebpack: (config) => {
    if (isProduction) {
      config.externals = {
        vue: "Vue",
        "vue-router": "VueRouter",
        vant: "vant",
        _: "lodash",
      };
```
externals是让里面的库不被webapck打包，也不影响通过import（或者其他AMD、CMD等）方式引入

2、 在`index.html`引入cdn资源
```
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/vant@2.8/lib/index.css" />
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.11"></script>
  <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/vant@2.8/lib/vant.min.js"></script>
  <script src="https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.15/lodash.core.min.js"></script> 
```
3、 优化下写法
```
// vue.config.js

const cdn = {
  css: ["https://cdn.jsdelivr.net/npm/vant@2.8/lib/index.css"],
  js: [
    "https://cdn.jsdelivr.net/npm/vue@2.6.11",
    "https://unpkg.com/vue-router/dist/vue-router.js",
    "https://cdn.jsdelivr.net/npm/vant@2.8/lib/vant.min.js",
    "https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.15/lodash.core.min.js",
  ],
};


chainWebpack: (config) => {
    if (isProduction) {
      config.plugin("html").tap((args) => {
        args[0].cdn = cdn;
        return args;
      });
    }
  }
 
 // index.html
   <% for(var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.css){%>
  <link src="<%= htmlWebpackPlugin.options.cdn.css[i] %>"></link>
  <%}%>
  <% for(var i in htmlWebpackPlugin.options.cdn && htmlWebpackPlugin.options.cdn.js){%>
  <script src="<%= htmlWebpackPlugin.options.cdn.js[i] %>"></script>
  <%}%>
```
优化前
![](https://user-gold-cdn.xitu.io/2020/6/7/1728e03107aa7774?w=1244&h=726&f=png&s=133005)
优化后

![](https://user-gold-cdn.xitu.io/2020/6/7/1728e03a5f6f2b0f?w=1296&h=726&f=png&s=135559)

### 开启gzip压缩
```

cnpm install compression-webpack-plugin --save-dev 

// vue.config.js
const CompressionPlugin = require("compression-webpack-plugin");

 configureWebpack: (config) => {
    if (isProduction) {
      config.externals = {
        vue: "Vue",
        "vue-router": "VueRouter",
        vant: "vant",
        _: "lodash",
      };
      config.plugins.push(
        new CompressionPlugin({
          algorithm: "gzip",
        })
      );
    }
  },
```

### 路由懒加载
```
  {
    path: "/index",
    name: "index",
    component: () => import("../views/index.vue"),
    meta: {
      keepAlive: false,
    },
  },
```

平常当中经常这么写，只不过你可能不知道这就是路由懒加载。[官网](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)，官网上提到这是由于[异步组件](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%BC%82%E6%AD%A5%E7%BB%84%E4%BB%B6)和weback的代码分割[code-splitting](https://webpack.js.org/guides/code-splitting/)功能，每个异步路由会打包成不同的块，实现按需加载。除了上面的` ES2015`模块的写法，它还有
```
 {
    path: "/index",
    name: "index",
    component: (resolve) => require(["../views/index.vue"], resolve),
    meta: {
      keepAlive: false,
    },
  },
```
根据官网提到的，首先创建一个返回`Promise`的工厂函数
```
const Index = () => Promise.resolve({ /* 组件定义对象 */ })
```

然后用[`动态 import`](https://github.com/tc39/proposal-dynamic-import)语法定义代码分块的分割点
```
import ('./Index')
```
两者结合，就可以自动分割代码
```
const Index = () => import('../views/index.vue')
```

先看了下这个[code-split](https://webpack.js.org/guides/code-splitting/)，

![](https://user-gold-cdn.xitu.io/2020/6/7/1728e562724b3f11?w=1462&h=338&f=png&s=58543)

然后去看了下vue-cli的具体配置，路由已经是`Dynamic Import`，如上图第二点提到.
下面说说针对于`chunk-vendors`的分割，正如上图第三点提到，配置`splitChunks`是了重复数据删除和分割`chunks`
#### optimization(开发和生产都是一样的)
```
optimization:{
      splitChunks: {
        cacheGroups: {  // 缓存组，可以定义多个
          vendors: {     //创建一个 自定义的vendor的chunk
            name: "chunk-vendors",
            test: /[\\\/]node_modules[\\\/]/,  // 匹配node_modules
            priority: -10,  // 理解为缓存的级别
            chunks: "initial",
        },
          common: {
            name: "chunk-common",
            minChunks: 2,  // 分割之前必须共享模块的最小chunk数
            priority: -20,
            chunks: "initial",
            reuseExistingChunk: true,  // 是否复用存在的块
        },
      },
    } 
}
 
```
- cacheGroups  缓存组，可以定义多个
```
 cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
```
- priority 缓存组的级别
> A module can belong to multiple cache groups. The optimization will prefer the cache group with a higher priority. The default groups have a negative priority to allow custom groups to take higher priority (default value is 0 for custom groups)

翻译过来大概就是一个模块可以有多个cash groups，`optimization`将会选择优先级更高的cash group, `default group` 的优先级为负数，可以允许`custom group`有更高的优先级（默认值是0）。
简而言之，`custom group`可以通过priority比`default group`高
- reuseExistingChunk 是否复用存在的块

#### `output`配置有些不同
```
// 生产
  output: {
    path:"/dist",
    filename: "static/js/[name].[contenthash:8].js",
    publicPath: "/",
    chunkFilename: "static/js/[name].[contenthash:8].js",  // 生成一个8位的hash值
  }
  
 // 开发
   output: {
    path:"/dist",
    filename: "static/js/[name].js",
    publicPath: "/",
    chunkFilename: "static/js/[name].js",
  }
```
运行打包

![](https://user-gold-cdn.xitu.io/2020/6/7/1728e6d647181876?w=1600&h=1464&f=png&s=278476)

生产输出了css,而开发环境没有,估计是生产配置了这句
```
  // 生产
    new MiniCssExtractPlugin({
      filename: "static/css/[name].[contenthash:8].css",
      chunkFilename: "static/css/[name].[contenthash:8].css",
    })
```
那么问题来了
- 开发模式打包出来文件的css到底怎么引入的
- 打包出来的dist的index.html的link和script标签引入的文件为何不同
-  preload 和 prefetch的区别
```
// development
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <link rel="icon" href="/favicon.ico">
  <title>南蓝</title>
 
  <link href="/static/js/0.js" rel="prefetch"><link href="/static/js/1.js" rel="prefetch"><link href="/static/js/2.js" rel="prefetch">
  <link href="/static/js/3.js" rel="prefetch">
  <link href="/static/js/app.js" rel="preload" as="script">
  <link href="/static/js/chunk-vendors.js" rel="preload" as="script">
</head>
<body>
  <noscript>
    <strong>We're sorry but f2e-client doesn't work properly without JavaScript enabled.
      Please enable it to continue.</strong>
  </noscript>
  <div id="app"></div>
<script type="text/javascript" src="/static/js/chunk-vendors.js"></script><script type="text/javascript" src="/static/js/app.js"></script></body>

</html>
 ```
 再来看看生产打包出来的index.html(production)
 ```
 // production
 <!DOCTYPE html>
<html lang=en>
<head>
    <meta charset=utf-8>
    <meta http-equiv=X-UA-Compatible content="IE=edge">
    <meta name=viewport content="width=device-width,initial-scale=1">
    <link rel=icon href=/favicon.ico>
    <title>鸭嘴兽</title>
    <script src=https://cdn.jsdelivr.net/npm/vant@2.8/lib/index.css></script>
    <script src=https://cdn.jsdelivr.net/npm/vue@2.6.11></script>
    <script src=https://unpkg.com/vue-router/dist/vue-router.js></script>
    <script src=https://cdn.jsdelivr.net/npm/vant@2.8/lib/vant.min.js></script>
    <script src=https://cdn.bootcdn.net/ajax/libs/lodash.js/4.17.15/lodash.core.min.js></script>
    <link href=/static/css/chunk-0c3564fc.b3594289.css rel=prefetch>
    <link href=/static/css/chunk-56255d30.c1e34d24.css rel=prefetch>
    <link href=/static/css/chunk-5ba5813d.a56f098f.css rel=prefetch>
    <link href=/static/css/chunk-9945b478.66fcd057.css rel=prefetch>
    <link href=/static/js/chunk-0c3564fc.9d02dfcc.js rel=prefetch>
    <link href=/static/js/chunk-56255d30.d0115acd.js rel=prefetch>
    <link href=/static/js/chunk-5ba5813d.10f75234.js rel=prefetch>
    <link href=/static/js/chunk-9945b478.dc8a4502.js rel=prefetch>
    <link href=/static/css/app.faa29057.css rel=preload as=style>
    <link href=/static/css/chunk-vendors.97c56160.css rel=preload as=style>
    <link href=/static/js/app.5deb2d45.js rel=preload as=script>
    <link href=/static/js/chunk-vendors.158e0179.js rel=preload as=script>
    <link href=/static/css/chunk-vendors.97c56160.css rel=stylesheet>
    <link href=/static/css/app.faa29057.css rel=stylesheet>
</head>

<body>
<noscript><strong>We're sorry but f2e-client doesn't work properly without JavaScript enabled. Please enable it to
            continue.</strong></noscript>
    <div id=app></div>
    <script src=/static/js/chunk-vendors.158e0179.js></script>
    <script src=/static/js/app.5deb2d45.js></script>
</body>

</html>
 ```
- 为啥有些js用link标签引入，有些用scipt标签引入
- link的as属性为啥有时有时无

![](https://user-gold-cdn.xitu.io/2020/6/7/1728e85023ca5064?w=658&h=656&f=png&s=910787)

### 弹出vue-cli配置webpack设置
```
开发换环境 npx vue-cli-service inspect --mode development  webpack.config.development.js
生产环境：npx vue-cli-service inspect --mode production  webpack.config.production.js
```

不管前端轮子怎么变化，万变不离其宗，这个宗指的就是`html`、`css`、`js`，所有要回归本质上去,我提出的那些问题，希望下次出篇文章弄懂它，有错误或者有建议可以提出来




