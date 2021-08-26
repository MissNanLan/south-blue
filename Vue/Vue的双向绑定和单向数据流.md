<div style= "border: 1px solid #ddd; box-shadow: 2px 2px 2px #ddd; padding:10px;">五一假期在家补了课，看到双向绑定与单向数据流的概念。瓦特，我之前以为Vue的V-model就是双向数据流，其实不然,下面让我仔细来看看
</div>

### 双向绑定
简而言之，双向绑定就是model的更新会触发view的更新，view的更新会触发model的更新，它们的作用是相互的
![双向绑定](https://user-gold-cdn.xitu.io/2019/5/3/16a7caae1cc36343?w=997&h=365&f=png&s=47303)
### 单向数据流
简而言之，单向数据流就是model的更新会触发view的更新，view的更新不会触发model的更新，它们的作用是单向的
![单向数据流](https://user-gold-cdn.xitu.io/2019/5/3/16a7d4aacf9dccf9?w=1020&h=381&f=png&s=37652)

这不是废话吗，谁都知道的

下面就是真正的干货了，板凳坐好
<ul>
<li>Vue是单向数据流，不是双向绑定</li>
<li>Vue的双向绑定不过是语法糖</li>
<li>Object.definePropert是用来做响应式更新的</li>
</ul>

### v-model的实现原理
* 放在组件上

父组件

```

  <AnalysisSub v-model="phoneInfo" :zip-code.sync="zipCode" />
  
```
子组件
```
<template>
  <div>
    <input
      :value="phoneInfo.phone"
      type="number"
      placeholder="手机号"
      @input="handlePhoneChange"
    />
    <input
      :value="zipCode"
      type="number"
      placeholder="邮编"
      @input="handleZipCodeChange"
    />
  </div>
</template>
<script>
export default {
  name: "PersonalInfo",
  model: {
    prop: "phoneInfo", // 默认 value
    event: "change" // 默认 input
  },
  props: {
    phoneInfo: Object,
    zipCode: String
  },
  methods: {
    handlePhoneChange(e) {
      this.$emit("change", {
        ...this.phoneInfo,
        phone: e.target.value
      });
    },
    handleZipCodeChange(e) {
      this.$emit("update:zipCode", e.target.value);
    }
  }
};
</script>
```

父组件的写法等同于
```
 <AnalysisSub :phone-info="phoneInfo" @change="val => (phoneInfo = val)"
    :zip-code="zipCode"  @update:zipCode="val => (zipCode = val)"/>
```
* 放在input元素上

其实上文已经体现了
```
<input v-model=“phoneInfo.phone”/>
<input :value="PhoneInfo.phone" @input="val => { PhoneInfo.phone = val }"
```
以上两句是相等的
### TIPS
[model](https://cn.vuejs.org/v2/api/#model) 2.2.0+
>允许一个自定义组件在使用 v-model 时定制 prop 和 event。默认情况下，一个组件上的 v-model 会把 value 用作 prop 且把 input 用作 event，但是一些输入类型比如单选框和复选框按钮可能想使用 value prop 来达到不同的目的。使用 model 选项可以回避这些情况产生的冲突。

[.sync修饰符](https://cn.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A5%B0%E7%AC%A6) 2.3.0+
>在有些情况下，我们可能需要对一个 prop 进行“双向绑定”。不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。

### 总结
所以说呢， vue还是单向数据流，v-model只不过是语法糖，它是`:value="sth"`和` @change="val => sth = val"`的简写形式。我们通常在面试当中被提问，Vue是怎么实现数据响应式更新的，面试官期望听到的回答是通过`Object.defineProperty() `的`get`和`set`方法来实现响应式更新。 

v-model和.sync修饰符分别在组件单个属性、多个属性需要双向绑定下使用，这是二者使用的场景