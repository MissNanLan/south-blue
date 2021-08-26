
### let与var、const的区别
稍微熟悉前端，都应该回答出来，很经典的面试题

#### let与var的区别
##### let作用域不会被提升
 - 示例
 ```js
 console.log(b)  // 报错
 let b = 1
 
 console.log(a)  //  undefined
 var a = 1
 ```
 - 原因
 
js引擎遇到var声明，会把它的作用域提升到顶部，而遇到let或const会放到TDZ（Temporal Dead Zone，暂时性死区），访问TDZ的变量会触发运行时错误，只有执行变量声明的语句之后，变量才会从TDZ中取出。

##### let重复声明会报错
- 示例

```
var a = 2
var a = 22
console.log(a) // 22

let a = 2
let a = 22
console.log(a) // Identifier 'a' has already been declared
```
- 原因

因为let不允许在相同作用域内重复声明相同的变量,注意关键词 "相同作用域",下面的例子是没问题的
```
let a = 2
function f(){
  let a =3
  console.log(a)
}
f()
```

##### let不绑定全局作用域

- 示例
 ```js
 let a = 1
 console.log(window.a) // undefined
 
 var a = 2
 console.log(window.a) // a
 ```
 - 原因
 暂没有想到
 
 #### let与const的区别
 
 const 一般用来声明常量，一旦声明一般类型，值就可以被修改，但是声明引用类型的值，值可以被修改，但是绑定不可以修改
 
 - 示例
 ```js
 const a = 2;
 a = 3    // 可以
 
 const a = {name: "nanlan"，age: 99}
 a.name = "xiaojuju"  // 可以
 a.sex = "female" // 可以
 a = { }  // 不可以，"a" is read-only
 a = {name:"nanlan”,age: 99,hobby: "coding"}  // 不可以，"a" is read-only
 
const arr = [1,2,3,4]
arr[1] = 88   // 可以
arr.push(5)     // 可以
arr.splice(0,2)   // 可以
arr.pop()       //  可以
arr = []        // 不可以，"arr" is read-only
 ```
 - 原因
 
 原始数据类型： undefined、null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）、Symbol
 
 细分的话，普通类型（String、Null、Undefined、Number、Boolean、Symbol）与引用类型（Function、Object、Array）存储的方式不一样。 请看下图（太丑请尽情吐槽）
 
 ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5b8dcbddac74dc8b99012cb1023f7ac~tplv-k3u1fbpfcp-watermark.image)
 
 MDN截图
 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/034302c8bcd74ffb81c43524625e527d~tplv-k3u1fbpfcp-watermark.image)
 
总结： 我理解的就是const声明就是一个值的只读引用。普通类型的地址也存放在栈内存中，引用类型在栈内存中存放了指向堆内存的地址，堆内存的值可以改变，栈内存的地址不可改变。

更好的答案:
>const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。

### Object.freeze

冻结对象，如果对象里面嵌套了对象，则无效，目前想到的办法就是递归调用。比如
```js
// 代码1
const foo = { a: "aaa", b: "bbb", c: "ccc"}
Object.freeze(foo)
foo.c = "ddd"
console.log("foo",foo)  // { a: "aaa", b: "bbb", c: "ccc"}  // 不变


// 代码2
const foo = { a: "aaa", b: "bbb", c: "ccc", d:{e:"eee"} }
Object.freeze(foo)
foo.d.e ="ddee"
console.log("foo",foo) // { a: "aaa", b: "bbb", c: "ccc", d:{ e:"ddee"} }  
// 会变
```
deepFreeze
```js
function deepFreeze(obj) {
  if (Object.prototype.toString.call(obj) !== "[object Object]") {
    throw new TypeError("Excepted Object,got " + typeof obj);
  }

  Object.keys(obj).forEach(function (name) {
    if (typeof obj[name] == "object" && obj[name] !== null)
      deepFreeze(obj[name]);
  });
  return Object.freeze(obj);
}

const foo = { a: "ddd", b: "bbb", c: "ccc", d: { e: "eee" } };
deepFreeze(foo);
// Object.freeze(foo)
foo.d.e = "ddee";
console.log(foo); // { a: "ddd", b: "bbb", c: "ccc", d: { e: "eee" } } 
// 不变

```
 
### 块级作用域
一个经典的面试题

```js
// 代码1
var funs = []
for(var i=0;i<3;i++){
  funs[i] = function(){
      console.log(i)
  }
}
// funs[0]()  // 3


// 代码2
for(let i=0;i<3;i++){
  funs[i] = function(){
      console.log(i)
  }
}
// funs[0]()  // 0


// 代码2的伪代码

// 伪代码
(let i = 0) {
    funcs[0] = function() {
        console.log(i)
    };
}

(let i = 1) {
    funcs[1] = function() {
        console.log(i)
    };
}

(let i = 2) {
    funcs[2] = function() {
        console.log(i)
    };
};


```
所以代码2的原因是因为存在块级作用域。即每次循环都会创建一个新的变量（按理来说，let声明的变量不可重复），其实循环的圆括号内会创建一个新的作用域，所以避免了let不可重复声明的特性，可以理解为模仿闭包来简化循环过程

下面是来自阮老师的例子，先不看答案

```js
var tmp = new Date();
function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';  // 将这里换成let试试，思考下结果的区别和原因
  }
}
f(); 
```

值得注意的是，在块级作用域中声明函数相当于是用var关键字声明，作用域会被提升到块级作用域的头部。猜猜下面的结果是多少

```js
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }
  f();
}());
```

### 参照:

[ES6 系列之 let 和 const](https://github.com/mqyqingfeng/Blog/issues/82)

[MDN const](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/const)

[JavaScript 深入了解基本类型和引用类型的值](https://segmentfault.com/a/1190000006752076)

[let 和 const 命令](https://es6.ruanyifeng.com/#docs/let)

[「前端进阶」JS中的栈内存堆内存](https://juejin.cn/post/6844903873992196110)
