  ### 关于事件循环 
   可以看看这几篇文章  [JavaScript中的Event Loop（事件循环）机制](https://zhuanlan.zhihu.com/p/145383822)  [详解JavaScript中的Event Loop（事件循环）机制](https://zhuanlan.zhihu.com/p/33058983) 

  ### 宏任务与微任务 
   ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ce503f5fe0f4b9486f836d8a211aada~tplv-k3u1fbpfcp-watermark.image)   

   ### vue.$nextTick 实现  
   - 大于2.6 $nextTick的源码[>2.6](https://github.com/vuejs/vue/blob/dev/src/core/util/next-tick.js)  提交记录：[fix: always use microtasks for nextTick #8450](https://github.com/vuejs/vue/pull/8450)  使用了Promise、MutationObserver、setImmediate、setTimeout 

    -  小于2.6大于等于2.5 $nextTick的源码[>2.5](https://github.com/vuejs/vue/blob/v2.5.21/src/core/util/next-tick.js)   使用了setImmediate、MessageChannel、setTimeout、promise     提交记录:[fix: use MessageChannel for nextTick](https://github.com/vuejs/vue/commit/6e41679a96582da3e0a60bdbf123c33ba0e86b31) 

     - 小于2.5  $nextTick的源码[<2.5](https://github.com/vuejs/vue/blob/v2.4.4/dist/vue.js)    使用了Promise、MutationObserver、setTimeout  
     
   #### 2.6.x的版本实现  
     这里有这么一大堆注释，我翻译下吧。  
     ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb513523847d484fb23ab2e6dcdb38a4~tplv-k3u1fbpfcp-watermark.image)     这里我们有使用微任务的异步延迟包装器。在2.5的版本中，我们使用了(宏观)任务(与微观任务相结合)（注：2.5x,依次使用setImmediate(宏)、MessageChannel(宏)、setTimeout(宏)、promise（微）），但是在重绘前改变状态时，会出现一些微妙的问题。   比如(e.g. [#6813](https://github.com/vuejs/vue/issues/6813), out-in transitions，注：这个问题我实在没有看出来，v-show是重绘(display:none)，v-if是重排)。 在事件处理程序中使用（宏）任务会导致一些奇怪的行为。那是我们绕不过去的(e.g. [#7109](https://github.com/vuejs/vue/issues/7109), [#7153](https://github.com/vuejs/vue/issues/7153), [#7546](https://github.com/vuejs/vue/issues/7546), [#7834](https://github.com/vuejs/vue/issues/7834), [#8109](https://github.com/vuejs/vue/pull/8019),注:以上问题我只看了#7546，在2.5的版本确实ipad,iphone确实要点击两次才能弹出键盘，在2.4的版本只需要点一次)，所以我们现在处处使用微任务。这种取舍的一个主要缺点是，有些场景下，微任务的优先级太高，在错综复杂的事件之间会出错(e.g. [#4521](https://github.com/vuejs/vue/issues/4521), [#6690](https://github.com/vuejs/vue/issues/6690), which have workarounds)，即使在同一事件的两次冒泡间([#6566](https://github.com/vuejs/vue/issues/6566))。  >fire in between supposedly sequential events  or even between bubbling of the same event 这句英文翻译不过来，啊啊啊     ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59ae4efe273d44719a640b34f7d13a9a~tplv-k3u1fbpfcp-watermark.image) nextTick行为利用微任务队列，可以通过原生Promise.then或MutationObserver来访问。MutationObserver有更广泛的支持，但是在iOS >= 9.3.3的UIWebView中，当在触发触摸事件，它有严重的bug。触发几次后就完全停止工作了......所以，如果有支持原生的Promise，我们就用Promise。          
        ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f89f3e295f99472da9d6344466155dbd~tplv-k3u1fbpfcp-watermark.image) 在有问题的UIWebViews中，Promise.then并没有完全崩溃，但它可能会卡在一个奇怪的状态，即回调被推送到微任务队列中，但队列并没有被刷新，直到浏览器需要做一些其他工作，例如处理一个计时器。因此我们可以通过添加一个空的时间来 "强制 "微任务队列被刷新。

   ### 小惊喜 
      我发现 vue判断是否支持原生 ``` js function isNative (Ctor) {   return typeof Ctor === 'function' && /native code/.test(Ctor.toString()) } isNative(Promise) isNative(MutationObserver) isNative(setImmediate) ``` 反正我是想不到这么写，哭唧唧~  
     
   ### 留给自己的思考 
      [#7546](https://github.com/vuejs/vue/issues/7546)。对于这个问题，暂时还没从源码上想明白，为啥在ipad上[2.4](https://github.com/vuejs/vue/blob/v2.4.4/dist/vue.js)版本只要点击一次，[2.5](https://github.com/vuejs/vue/blob/v2.5.21/src/core/util/next-tick.js)版本需要点击两次，呜呜。 而两个版本的区别在于 - 2.4版本依次使用setImmediate(宏)、MessageChannel(宏)、setTimeout(宏)、promise（微） - 2.5版本依次使用Promise、MutationObserver、setImmediate、setTimeout  

   ### 其他链接 
       [MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver) 
       [MessageChannel](https://developer.mozilla.org/zh-CN/docs/Web/API/MessageChannel)  
       [Vue.js 升级踩坑小记](https://github.com/DDFE/DDFE-blog/issues/24)  [简单理解Vue中的nextTick](https://juejin.cn/post/6844903557372575752)  [事件循环模拟网站](http://latentflip.com/loupe/?code=ZnVuY3Rpb24gYygpIHt9CmZ1bmN0aW9uIGIoKSB7CgljKCk7Cn0KZnVuY3Rpb24gYSgpIHsKCXNldFRpbWVvdXQoYiwgMjAwMCkKfQphKCk7)  [冴羽的博客](https://github.com/mqyqingfeng/Blog)     以上有错误，请指出，欢迎补充
