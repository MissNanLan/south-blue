>原文链接：[https://github.com/vuejs/vue-loader#how-it-works](https://github.com/vuejs/vue-loader#how-it-works)
## 什么vue-loader
`vue-loader`是用于webpack的加载器，允许你用叫做`Single-File Components`单文件组件的格式来写Vue组件
``` js
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello world!'
    }
  }
}
</script>

<style>
.example {
  color: red;
}
</style>
```
这里有`vue-loader`提供许多炫酷的功能
- 允许`Vue`组件的每个部分使用其它的`webpack`加载器，比如`Sass`加载`<style>`和`Pug`加载`<template>`
- 允许`.vue`文件中的自定义块，这些(自定义块)能够运用于定制的加载程序链
- 将静态的`<style>`和`<template>`的`assets`引用视为模块依赖，并且用`webpack`加载程序去处理他们
- 模拟每个组件的`CSS`作用域
- 在开发的过程中使用热加载保持状态

简而言之，`vue-loader`和`webpack`的组合能够使你在写`Vue.js`应用时，提供现代化、灵活的和功能非常强大的前端工作流

## vue-loader是怎么工作

`vue-loader`不是简单的源转换器。它用自己专用的加载链（你可以认为每个块是虚拟的模块）处理SFC(Single-file Component 单文件组件)内部的每个语言块，最后将这些块组成最终的模块。这是整个过程的简要概述
1. `vue-loader`使用`@vue/component-compiler-utils`将`SFC`源码解析成`SFC`描述符,然后为每个语言块生成一个导入，实际返回的模块代码看起来像这样
``` js
// 从主加载程序返回的代码source.vue的代码

// import the <template> block
import render from 'source.vue?vue&type=template'

// import the <script> block
import script from 'source.vue?vue&type=script'
export * from 'source.vue?vue&type=script'

// import <style> blocks
import 'source.vue?vue&type=style&index=1'

script.render = render
export default script
```
注意这些代码是从`source.vue`导入的，每个块都有不同的请求查询

2. 我们想要`script`的内容被视为`.js`文件（如果是`<script lang="ts"`，我们想要被视为`.ts`文件）。其他的语言块也是同样的。所以我们想要webpack 申请任何已配置模块的规则去匹配`.js`，也看起来像`source.vue?vue&type=script`的请求。这就是`VueLoaderPlugin(src/plugin.ts)`作用：对于webpack的每个模块规则，它创建一个相对于Vue语言块请求的修改后的克隆

假设我们为所有的`*.js`配置过`babel-loader`.这些规则也一样会复制和应用于到Vue SFC的`<script>`块中，内部到webpack，一个像这样的请求
```
import script from 'source.vue?vue&type=script'
```
将扩展为
```
import script from 'babel-loader!vue-loader!source.vue?vue&type=script'
```
注意是`vue-loader` 也会匹配，因为`vue-loader`是应用于`.vue`的文件。同样地，如果你为`*.scss`文件配置了`style-loader`+`css-loader`+`sass-loader`
```
<style scoped lang="scss">
```
将通过`vue-loader`返回
```
import 'source.vue?vue&type=style&index=1&scoped&lang=scss'
```
webpack将会扩展成
```
import 'style-loader!css-loader!sass-loader!vue-loader!source.vue?vue&type=style&index=1&scoped&lang=scss'
```
3. 在扩展请求的过程中，主`vue-loader`将再次被调用。但是这次，加载器注意到这些请求有查询并且只针对于特定块。所以选择（`src/select.ts`）目标块的内容将传递与加载器匹配的内容
4. 对于这些`<script>`块，这就差不多了。但是对于`<template>`和`<style>`，一些额外的任务需要被执行：
- 我们需要使用Vue模板编译器编译模板
- 我们需要在`css-loader`之后但是在`style-loader`之前，为`<style scoped>`块进行CSS处理。

从技术上来看，这里有额外的加载器（`src/templateLoader.ts` 和 `src/stylePostLoader.ts`）需要注入到扩展的加载程序链。如果终端用户不去配置（项目），这将会很复杂，所以`VueLoaderPlugin`也可以注入到一个全局`Pitching Loader(src/pitcher.ts)`并且监听Vue`<template>`和`<style>`请求,注入必要的加载器中。最终的请求像下面这样：
```
// <template lang="pug">
import 'vue-loader/template-loader!pug-loader!source.vue?vue&type=template'

// <style scoped lang="scss">
import 'style-loader!vue-loader/style-post-loader!css-loader!sass-loader!vue-loader!source.vue?vue&type=style&index=1&scoped&lang=scss'
```