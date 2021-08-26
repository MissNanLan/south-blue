### rem +js 布局


众所周知，rem是根据根元素的大小的变化而去改变的，rem的本质是等比缩放。先看看[rem](https://caniuse.com/?search=rem)的兼容性


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dedd7b2e298641f195c73547df3c3f22~tplv-k3u1fbpfcp-watermark.image)


浏览器支持程度还不错，
- Android Browser2.1+，iOS Safari4.1+（iOS Safari 5.0-5.1支持rem，但不支持媒体查询）
- 当在字体属性中使用时（整个声明被忽略）或在伪元素上使用时，IE 9和IE 10不支持rem单位

``` 
// $design-width: 设计稿的尺寸
// $blocks: 分成多少块
// $px: 元素的宽度
// rem与px对应关系，1rem代表在JS中设置的html的font-size值（为一块的宽度）

        $px                     $rem
    -------------    ===    ------------
    $design-width              $blocks
    $rem = $px / $design-width * $blocks
 ```
 

#### 步骤


1、 js动态设置html的font-size的大小,假设$block为3.75

```  js
document.documentElement.style.fontSize = document.documentElement.clientWidth/3.75 = 100
```

除以`3.75`并不是明文规定的，也有人除以`100`，就是把当成宽度分成多少份，3.75就是分成3.75份，100就是分成100份。也有人根元素不用js动态计算，用媒体查询（后面会提到）

一

|  机型   | 宽度  |  根元素大小 |
|  ----  | ----  |  ----  | 
| iphone5  | 320 | 85.33  |
| iphone6/7/8、iphone X  | 375 |  100 |
| iphone6/7/8 plus 、iphone XR、iphone XR Max| 414 |   110.4|

上面就是1rem在不同尺寸下等于不同大小的px

2、 px转成rem


设计稿通常用的是750px，我们不可能每次写的时候自己换算，所以需要把px转换成rem
假设有个300px的元素，换成rem就是 300/375x3.75 = 1.5rem(此处有问题，375是独立像素，不是设计稿的大小)，如此看来在320、375、414元素的实际尺寸是：根元素的大小乘以rem的尺寸 = 255.99px，300px，331.2px

-  postcss-plugin-px2rem 插件 

```  js
npm i --save postcss-plugin-px2rem

```

在webpack 3.x中 使用

``` js
 const webpack = require("webpack");
 const px2rem = require("postcss-plugin-px2rem");
  plugins: [
    new webpack.LoaderOptionsPlugin({
      vue: {
        postcss: [
          px2rem({
            rootValue: 100,
            exclude: /(node_module)/
          })
        ]
      }
    })
  ]
```

在 vue-cli3.x 中使用

``` js
css: {
        loaderOptions: {
            postcss: {
                plugins: [
                    require('postcss-plugin-px2rem')({
                    }),
                ]
            }
        }
  ```
[px2rem 相关参数介绍](https://www.npmjs.com/package/postcss-plugin-px2rem)

3、 从某网站学习扒下来的代码

他们是这样子计算html的font-size的大小的，首先监听窗口变化的事件(orientationchange事件在设备的纵横方向改变时触发),接着用屏幕宽度除以最小宽度乘以20，然后给fontSize做了取整的运算；如果是pc端访问，font-size就一直设为20px,移动端运用动态计算的fontSize
``` js
var docEl = doc.documentElement,
resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
recalc = function () {
    var clientWidth = docEl.clientWidth;
    if (!clientWidth) return;
    var fontSize = 20 * (clientWidth / 320);
    fontSize = (fontSize > 54) ? 54: fontSize;

    if (~~fontSize !== fontSize) {
      fontSize = ~~fontSize  // ~~ 相当于-(-(x+1) + 1)
    }

    //如果是pc访问
    if(!/windows phone|iphone|android/ig.test(window.navigator.userAgent)) {
        fontSize = 20;
    }

    docEl.REM2PX = 20;  // 这段没看懂
    docEl.style.fontSize = fontSize + 'px';
    var dpi =  window.devicePixelRatio;
   
    docEl.setAttribute('data-dpi',dpi); // 为html设置自定义data-dpi属性

if (!doc.addEventListener) return;
win.addEventListener(resizeEvt, recalc, false);
doc.addEventListener('DOMContentLoaded', recalc, false);
recalc();
```

### rem + 媒体查询

这个其实没啥好说的,就是js动态设置html的font-size的大小换成媒体查询。媒体查询感觉就是很不好判断界线，特别注意的是，iOS Safari的5.0-5.1不支持媒体查询，不过现在都更新到14了，这个不重要

``` CSS
@media screen and (max-width: 320px) {
  html {
    font-size: 14px;
  }
}

@media only screen and (max-width: 414px) and (min-width: 360px) {
  html {
    font-size: 16px;
  }
}

@media only screen and (max-width: 640px) and (min-width: 414px) {
  html {
    font-size: 18px;
  }
}

@media only screen and (max-width: 768px) and (min-width: 640px) {
  html {
    font-size: 20px;
  }
}

@media only screen and (min-width: 768px) {
  html {
    font-size: 22px;
  }
}

```
### vw 布局

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a622b3cf3f01490e923d464bb6df3610~tplv-k3u1fbpfcp-watermark.image)

[兼容性](https://caniuse.com/?search=vw)
- IE9的部分支持是指支持“vm”而不是“vmin”
- IE10 部分支持是指不支持“vmax”单元。
- 在iOS7中的部分支持是由于 "vh "单元的错误行为。

>vw —— 视口宽度的 1/100；vh —— 视口高度的 1/100 —— MDN

```
/*
    vw与px对应关系，100vw为视窗宽度，$vw即为$px对应占多宽

        $px                    $vw
    -------------    ===    ------------
    $design-width              100vw
*/
$vw = ($px / $design-width) * 100vw


```

### vw+rem

这种组合通常是是html的font-size用vw，其他的则用rem

 - 根元素设置
``` js
/* 移动端页面设计稿宽度 */
$design-width: 750;
/* 移动端页面设计稿dpr基准值 */
$design-dpr: 2;
/* 将移动端页面分为10块 */
$blocks: 10;
/* 缩放所支持的设备最小宽度 */
$min-device-width: 320px;
/* 缩放所支持的设备最大宽度 */
$max-device-width: 540px;

/* html根元素的font-size定义，简单地将页面分为$blocks块，方便计算 */
@mixin root-font-size() {
    font-size: 100vw / $blocks;

    body {
        @include container-min-width();
    }

    /* 最小宽度定义 */
    @media screen and (max-width: $min-device-width) {
        font-size: $min-device-width / $blocks;
    }

    /* 最大宽度定义 */
    &[data-content-max] {  // 为
        body[data-content-max] {
            @include container-max-width();
        }

        @media screen and (min-width: $max-device-width) {
            font-size: $max-device-width / $blocks;
        }
    }
}

/* 设置容器拉伸的最小宽度 */
@mixin container-min-width() {
    margin-right: auto;
    margin-left: auto;
    min-width: $min-device-width;
}

/* 设置容器拉伸的最大宽度 */
@mixin container-max-width() {
    margin-right: auto;
    margin-left: auto;
    max-width: $max-device-width;
}

```
- 其他的元素设置
``` css
/* 单位px转化为rem */
@function px2rem($px) {
    @return #{$px / $design-width * $blocks}rem;
}
```

- 字体的大小
``` css
/* 设置字体大小，不使用rem单位， 根据dpr值分段调整 */
@mixin font-size($fontSize) {
    font-size: $fontSize / $design-dpr;

    [data-dpr="2"] & {
        font-size: $fontSize / $design-dpr * 2;
    }

    [data-dpr="3"] & {
        font-size: $fontSize / $design-dpr * 3;
    }
}

```
使用


``` css
html {
    @include root-font-size();
}

header {
    height: px2rem(300);
    line-height: px2rem(300);
    text-align: center;
    background-color: #f2f2f2;
}
 
```

以上代码[参考](https://github.com/imwtr/rem-vw-layout)



### flexiable
- [flexiable](https://github.com/amfe/lib-flexible)

> 由于viewport单位得到众多浏览器的兼容，lib-flexible这个过渡方案已经可以放弃使用，不管是现在的版本还是以前的版本，都存有一定的问题。建议大家开始使用viewport来替代此方。

### 总结

简单的界面可以用rem+js实现，对于内容复杂可以用rem+vw的方式。我们在平时当中，一般都是直接写px，所以需要运用相关插件把我们自动转化成rem，配合postcss，有兴趣可以试下。看了很多代码，实现方式大同小异。
 
### 参考

- https://yanhaijing.com/css/2017/09/29/principle-of-rem-layout/

- https://www.cnblogs.com/imwtr/p/9648233.html#s3

- https://www.npmjs.com/package/px2rem

- amfe-flexible



