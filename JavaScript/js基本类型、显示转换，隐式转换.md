## 数据的类型
 
 
- 基本类型（primitive values）: 也称作原始类型，NULL、Undefined、String、Boolean、Number、BigInt、Symbol

- 引用类型(reference values)：Object，细分的话还有Date、Array、Function、RegExp

### 区别

- 引用类型是存储在堆内存的

``` js
var a = {name:"南蓝"};
var b;
b = a;
a.name = "nanlan";
console.log(b.name);    // nanlan
b.age = 18;
console.log(a.age);     // 18
var c = {
  name: "xiaoju",
  age: 20
};
```
未命名文件.png

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93c9c699a8bb4c468d41674deb94b18c~tplv-k3u1fbpfcp-watermark.image)

a和b都指向同一个地址，c是单独新开辟出来的。它们作为引用类型都会在栈内存中存放一个引用地址，这个引用地址指向存在堆内存中存放的值。 所以也就有浅拷贝，引用类型的值会改变原有的对象，因为他们的引用地址是一样



- 基本类型是存储在栈内存的


涉及问题： 深拷贝与浅拷贝


## 类型检测

 ### typeof 
特点：一般检测基本类型
缺点：对于null或者引用类型(Date、RegExp)都是返回object

 ```
 typeof null         // object
 typeof []           // object
 typeof /^\w/g       // object
 typeof new Date()   // object 
 ```
 
 ### instanceof
 特点：变量是不是给定引用类型(根据原型链来判断)的实例
 缺点：所有引用类型都是Object的实例
 
 ```js
 result = variable instanceof Constructor
 ```
 
 ```js
 let arr = [];
 let reg = /^\w\g/
 arr instanceof Array       // true
 arr instanceof  Object     // true
 reg instanceof Regxp       // true
 reg instanceof  Object     // true
 ```
 
 #### instanceof的原理
 
 右边的原型在左边的原型链上
 
 ```js
function myInstanceOf(leftValue, rightValue) {
  leftValue = left.__proto__;
  let rightProto = right.prototype;
  while (true) {
    if (leftValue === null) return false;
    if (leftValue === rightProto) return true;
    leftValue = leftValue.__proto__;
  }
}
```

涉及知识点:原型链继承

然后看看其他的几个有趣的例子

``` js
function Foo() {
}

Object instanceof Object // true
Object instanceOf Function  // true
Function instanceof Function // true
Function instanceof Object // true
Foo instanceof Foo // false
Foo instanceof Object // true
Foo instanceof Function // true
```

比如说第一个， 第一次进入
```
leftValue = Object.__proto__ === Function.prototype
rightProto = Object.prototype
// 不相等
leftValue = Function.prototype.__proto__ =  Object.Prototype
// 第二次
leftValue === rightProto
```
再看看 ```Foo instanceof Foo```为false

``` js
// 第一次判断不相等
leftValue = Foo.__proto__ = Function.prototype
rightProto = Foo.prototype

// 第二次判断不相等
leftValue  = Funtion.protype._proto_ = Object.prototype
rightProto = Foo.prototype

// 第三次判断不相等
leftValue = Object.prototype._proto_ = null
返回false
```


### 最佳的判断类型的值
```
 Object.prototype.tostring.call()
```
 ```js
 let arr = []
 let n = null
 // Object.prototype.tostring.call(arr) // [object Array]
 // Object.prototype.tostring.call(n) // [object Null]
 ```
 
 ## 类型转换
 
[JavaScript 深入之头疼的类型转换](https://github.com/mqyqingfeng/Blog/issues/159),这篇博客讲得很详细
 
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1a8343c7e2748cd93178e8f5adb8fbd~tplv-k3u1fbpfcp-watermark.image)

以上是我总结的显式转换与隐式转换，[https://mubu.com/app/edit/home/6lAnSx4NbJ](https://mubu.com/app/edit/home/6lAnSx4NbJ)
 
 ### 显式转换
 
 
常用的主要是以下几种,给出的连接地址是es5规范，写得很详尽。

- Boolean() : [http://es5.github.io/#x9.2](http://es5.github.io/#x9.2)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/212eaf40cd994dec83b9228f98f812cd~tplv-k3u1fbpfcp-watermark.image)

按照ToBoolean规范去转化，Boolean还是比较简单的

- Number() : [http://es5.github.io/#x9.3](http://es5.github.io/#x9.3)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40570e3bc6d440e8af8b4a68c29109da~tplv-k3u1fbpfcp-watermark.image)

按照ToNumber规范去转化。直接注意的是入参是String和Object类型，如果是String类型的，[9.3.1](http://es5.github.io/#x9.3.1)写了一大篇，简单来讲，就是String如果不是数字的话，返回NaN，是数字的话看情况,举几个例子

``` js
Number("123") ->123
Number("0xA") -> 十六进制转化十进制10，当然其他进制也是如此，都会转成十进制
Number("0xA") -> 十六进制转化十进制10
Number('-0') ->-0
Number('-00123')  -> 123
Number('Infinity') ->Infinity
Number('3.3e-7') ->3.3e-7
```

如果是入参是Object类型，则调用[ToPrimitive](http://es5.github.io/#x9.1),然后再根据ToNumber去转化

- String() : [http://es5.github.io/#x9.8](http://es5.github.io/#x9.8)


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fbaaa384bc3497d99fa03a3bc19e26f~tplv-k3u1fbpfcp-watermark.image)

如果入参是Number类型话，则会根据[9.8.1](http://es5.github.io/#x9.8.1)去转化。举几个例子
``` js
String(-0)/String(+0) ->0
String(NaN) -> NaN
String(Infinity)  -> "Infinity"
String(3.3e-7) -> "3.3e-7"
String(9007199254740991) -> "9007199254740991"
```
如果入参是Object类型，也是调用[ToPrimitive](http://es5.github.io/#x9.1)方法，只是默认参数不一样，hint String


#### String() 与toString()

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0410aecb8c5d4066af6f2cadfa5b093f~tplv-k3u1fbpfcp-watermark.image)

区别在于String()，可以转化null和undefined，而toString()不可以，如果还有其他的区别请告诉我. 还有就是toString()要比String()所处的位置要先一层，比如说当入参是Object类型，ToPrimitive 转化为原始类型的方法，即会调用toString(), 下面会说ToPrimitive

#### valueof() 与toString()、ToPrimitive

- [toString](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString): 一个表示该对象的字符串。null 和undefined没有toString方法

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76cb221c11ff454aa00e13dcc9f8be82~tplv-k3u1fbpfcp-watermark.image)

- [valueOf](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf): 返回值为对象的原始值

``` js
console.log(({}).valueOf()) // {}
console.log([].valueOf()) //  []
console.log([0].valueOf()) // [0]
console.log([1, 2, 3].valueOf()) // [1,2,3]
console.log((function(){var a = 1;}).valueOf()) // function (){var a = 1;}
console.log((/\d+/g).valueOf()) //  /\d+/g
console.log((new Date(2021, 1, 1)).valueOf())  // 1612108800000
```
- toPrimitive [http://es5.github.io/#x9.1](http://es5.github.io/#x9.1)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aab5f8d2c40346a497d55ba22984fb22~tplv-k3u1fbpfcp-watermark.image)

看完有没有很理解了

 ### 隐式转换 
 
 触发隐式转换的条件：一元、二元（+）、==、if、? :、&&
 
 ### 一元操作符
 
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6da8f3a9ca7456b9b4b479976172586~tplv-k3u1fbpfcp-watermark.image)

举几个例子吧

 ### 二元操作符
 
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5ae472d217c4cecad63077f6b3750ab~tplv-k3u1fbpfcp-watermark.image)
 
 举几个例子吧
[]+[], {}+{},[]+{}

[]+[]
   1、lprim = ToPrimitive([]) = ToPrimitive,相当于ToPrimitive([], Number)，所以调用valueof方法，返回[],由于[]不是原始类型，接着调用[].toString(),返回空字符串""
   2 、rprim = ToPrimitive([])也是如此
   3、 两边都是字符串，做拼接的效果
   4、 所以[]+[] 是""
   
依次类推{}+{},[]+{},[]+undefined,[]+null 也是如此,

 
## 一段代码

``` js
function isNative (Ctor) {
  return typeof Ctor === 'function' && /native code/.test(Ctor.toString())
}
isNative(Promise)
isNative(MutationObserver)
isNative(setImmediate)
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/863683d092d94ea8bbb1f8fbe9ac47bf~tplv-k3u1fbpfcp-watermark.image)

以上这段代码是在尤大的vue源码中找到的，我觉得比较有意思

 ## 经典面试题 
 ### 面试题1
 ``` js
如何使 a==1 && a==2 && a==3
 ```
 
 ``` 
 function A(value){
  this.value = value
}

A.prototype.toString = function(){
  return this.value ++
}
const a = new A(1);
if (a == 1 && a == 2 && a == 3) {
  console.log("hi nanlan!");
}
```

接着a===1 && a===2 a===3(与隐式转换无关)

```
var value = 0; //window.value
Object.defineProperty(window, 'a', {
    get: function() {
        return this.value += 1;
    }
});

console.log(a===1 && a===2 && a===3) //true
```
### 面试题2

```
如何使 f(1)(2)(3) = 6;
```

``` js
function f(){
  let args = [...arguments];
  let add = function(){
    args.push(...arguments)
    return add
  }
  add.toString = function(){
    return args.reduce((a,b)=>{
      return a+b
    })
  }
  return add
}

console.log(f(1)(2)(3))
```

以上两个面试题的做法都是重写toString()方法

## 总结

看似简单的数据类型，其实可以衍生出来很多知识点。 基本类型（也叫做原始类型 primitive values）和引用类型的区别，了解之后就知道const声明的对象的值为啥可以改变，而引用地址不可变

接着是每种检测基本类型和引用类型的方法，以及他们之间的区别。最后是类型转换，有显式转换(Number()、Boolean()、String()...)和隐式转换,隐式转换最主要是了解toString()、valueOf()以及ToPrivitive
 
## 参照

[JavaScript深入之头疼的类型转换上)
](https://github.com/mqyqingfeng/Blog/issues/159)

[JavaScript深入之头疼的类型转换(下)
](https://github.com/mqyqingfeng/Blog/issues/164)