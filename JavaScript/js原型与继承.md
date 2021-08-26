
 
### 原型与原型链

每一个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含指向原型对象的内部指针。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3476f53db2154b2093e9dc5e62c0dad0~tplv-k3u1fbpfcp-watermark.image)

``` js
person.__proto__ === Person.prototype
```

每个原型都有一个 constructor 属性指向关联的构造函数
```
Person.prototype.constructor === Person
```

es5获取对象的原型
```
Object.getPrototypeOf(person) === Person.prototype
```

#### 继承

##### 原型链
- 基本思想：利用原型让一个引用类型继承另一个引用类型的数组和方法。
- 缺点：优点也是原型链继承的缺点，引用类的原型属性会被所有的实例共享，且不能向父类型传递参数,请注意是引用类型的值
```
function Animal(){
  this.species = ['哈士奇','柯基']
}
function Dog(){
}

Dog.prototype = new Animal(); // 继承了Animal的属性和方法

var dog1 = new Dog();
dog1.species.push('萨摩耶')
var dog2 = new Dog();
console.log(dog1.species);    // ["哈士奇", "柯基", "萨摩耶"]
console.log(dog2.species);    // ["哈士奇", "柯基", "萨摩耶"]
```

##### 借用构造函数继承

- 基本思想：在子类型构造函数的内部调用超类型的构造函数
- 优点：相比原型链而言，借用构造函数有一个很大的优势，就是子类型函数构造函数可以向超类型构造函数传递参数
- 缺点：方法都在构造函数中定义，因此函数的复用性就无从谈起了(不明白)

``` js           
function Animal(){
  this.species = ['哈士奇','柯基']
}

function Dog(){
   Animal.call(this)
}
var dog1 = new Dog();
dog1.species.push('萨摩耶')
var dog2 = new Dog();
console.log(dog1.species);    // ["哈士奇", "柯基", "萨摩耶"]
console.log(dog2.species);    // ["哈士奇", "柯基"]
```
传递参数
``` js
function  Animal(speices){
  this.speices = speices;
}

function Dog(speices){
  Animal.call(this,speices);  
}

var dog1 = new Dog('中华田园犬');
var dog2 = new Dog();
console.log(dog1.speices);  //  中华田园犬
console.log(dog2.speices);  //  中华田园犬
```
#### 组合继承
- 基本思想： 原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承
- 优点： 在原型上定义方法是实现了函数复用，又能偶保证每个实例都有它自己的属性
- 缺点： 无论在什么情况下，都会两次调用超类型构造函数（具体什么原因，后面会讲）

```js
function Animal(speices){
  this.speices = speices;
  this.skills = ["jump","climb","catch","run"]
}

Animal.prototype.getSpeices = function(){
  console.log(this.speices)
}

function Dog(speices,color){
  // 借用构造函数继承，继承父类的属性
  Animal.call(this,speices);  // 第二次
  this.color = color;
}

// 原型继承，继承父类的方法
Dog.prototype = new Animal();  // 第一次

Dog.prototype.getColors = function(){
  console.log(this.colors);
}

var dog1 = new Dog('博美','white');  // 在这里用的时候
dog1.skills.push('acting');
console.log(dog.skills);  //  ["jump","climb","catch","run","acting"]
dog1.getSpeices();  // 博美

var dog2 = new Dog('柯基','brown');
console.log(dog2.skills); //  ["jump","climb","catch","run"]
dog2.getSpeices();  // 柯基
```
关于两次调用父类的构造函数，先画个图
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fbe1a9d41ed640d2ada2a32a04d2787d~tplv-k3u1fbpfcp-watermark.image)

在第一次调用Animal构造函数时，Dog.prototype会得到两个属性speices和skills，位于Dog的原型中。当调用Dog构造函数时，又会调用一次Animal构造函数，这一次位于新的实例对象上。在新的对象上创建了实例属性speices和skills，这两个属性会屏蔽原型中的两个同名的属性

#### 原型式继承
- 基本思想：没有使用构造函数，借助原型可以基于已有的对象创建新对象，同时还不必创建自定义类型。就是 `ES5 Object.create` 的模拟实现，将传入的对象作为创建的对象的原型。
- 优点： 想让一个对象与另一个对象保持类似，就不用创建构造函数了
- 缺点： 包含引用类型的值始终会共享，这跟原型链继承的缺点一样
``` js
function createObj(o){
 function F(){}
 F.prototype = o;
 return new F();
}

```
使用

```js
var dog = {
  species: '比熊犬',
  color: 'gold',
  skills: ["jump"]
}

var　dog1 = createObj(dog); //  dog 对象作为dog1 对象的基础，在ES5当中，这里可以写成
dog1.species = ' 泰迪';
dog1.color = 'brown';
dog1.skills.push('acting');

var　dog2 = createObj(dog);
dog2.species = ' 吉娃娃';
dog2.color = 'grey';
dog2.skills.push('show');

console.log(dog1.skills)   // ["jump","acting","show"]
console.log(dog2.skills)    // ["jump","acting","show"]

```
原型式继承就是`Objecte.create`ES5的写法,他还有第二个参数，就是与`Object.defineProperties()`方法的第二个参数格式相同，比如

``` js
var person = {
name: "南蓝",
age: 24,
hobby: ['acting','codeing;]
}
Object.create(person,{
  
})
```
``` js
var o
o = {};
// 以字面量方式创建的空对象就相当于:
o = Object.create(Object.prototype);
```

#### 寄生式继承
- 基本思路： 与原型式继承紧密相关，创建一个用于封装继承过程的函数
- 优点： 在考虑对象不是自定义类型和构造函数的情况下，寄生式继承也是一种很有效的方式
- 缺点：使用寄生式继承式为对象添加函数，不能够做到函数复用

```js
function createDog(obj){
 var clone = Object.create(obj);
 clone.getColor = function(){
   console.log(clone.color)
 }
 return clone
}

var dog = {
 species: '贵宾犬',
 color: 'yellow'
}

var dog1 = createDog(dog);
dog1.getColor();  // yellow
```

#### 寄生组合式继承
基本思想：利用借用构造函数继承属性，原型链的混成形式来继承方法。和组合继承有点类似，不过它解决组合继承调用两次父类的构造函数，就是在子类的原型调用父类的构造函数时做下改变，用寄生式继承来做这步操作，我们想要无非不就是父类原型的一个副本而已

优点： 解决了组合继承两次调用父类的构造函数，普遍人员认为这是最理想的一种方式

```js
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

function inheritPrototype(child,parent){
  var prototype = object(parent.prototype);  //  创建对象，此处可以用  Object.create(parent.prototype)
  prototype.constructor = child;   // 增强对像
  child.prototype = prototype;      // 指定对象
}

function Animal(speices){
  this.speices = speices;
  this.skills = ["jump","climb","catch","run"];
}

Animal.prototype.getSpeices = function(){
  console.log(this.speices)
}

function Dog(species,color){
  Animal.call(this,species);
  this.color = color;
}

inheritPrototype(Dog,Animal); 
// 在组合继承里面，这句是 Dog.prototype = new Animal()

var dog1 = new Dog('牧羊犬','black');
dog1.getSpeices(); // 牧羊犬
```

涉及问题：仔细对比组合继承（借用构造函数继承+原型继承）与寄生组合继承（借用构造函数与原型链的混成方式）
### class关键字
#### 实例属性
``` js
class Person {
    constructor(name) {
        this.name = name;  // 实例属性
    }
    
    sayHello() {
        return 'hello, I am ' + this.name;
    }
}

var kevin = new Person('Kevin');
kevin.sayHello()
```

新的写法
``` js
class Person {
    name = "小菊"
    sayHello() {
        return 'hello, I am ' + name;
    }
}
```

对应的es5代码

``` js
function Person(){
   this.name = name
}

Person.prototype.sayHello = function() {
      return 'hello, I am ' + this.name;
 }

```

#### 静态属性和方法

所有在类中定义的方法或方法，都会被实例继承。如果在一个方法前，加上 static 关键字，就表示该方法或属性不会被实例继承，而是直接通过类来调用，这就称为“静态方法或属性”。

``` js
class Person {
  static name = "xiaoju"
  static sayHello = function(){
       return 'hello, I am ' + name;
  }
}

console.log(Person.name)
Person.sayHello()
```
对应的es5代码

``` js
function Person() {}

Person.sayHello = function() {
    return 'hello';
};

Person.sayHello(); // 'hello'

var kevin = new Person();
kevin.sayHello();   // TypeError: kevin.sayHello is not a function
```


``` js
class Person {

 constructor(){
   this.age = 24
 }

  static bar = 'bar';
  static onSayHello = function(){
      return this.bar
  }
  sayHello() {
     return 'hello, I am ' + this.age;
   }
}
```
对应的es5代码
```
 function Person(){
    this.age = 24
 }
 Person.prototype.sayHello = function(){
    retur 'hello,I am' + this.age
 }
 Person.bar = "bar"
 Person.onSayHello = function(){
    return this.bar
 }
```
编译后的代码
``` js
"use strict";

function _instanceof(left, right) { if (right != null && typeof Symbol !== "undefined" && right[Symbol.hasInstance]) { return !!right[Symbol.hasInstance](left); } else { return left instanceof right; } }

function _classCallCheck(instance, Constructor) { if (!_instanceof(instance, Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

function _defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } }

function _createClass(Constructor, protoProps, staticProps) { if (protoProps) _defineProperties(Constructor.prototype, protoProps); if (staticProps) _defineProperties(Constructor, staticProps); return Constructor; }

function _defineProperty(obj, key, value) { if (key in obj) { Object.defineProperty(obj, key, { value: value, enumerable: true, configurable: true, writable: true }); } else { obj[key] = value; } return obj; }

var Person = /*#__PURE__*/function () {
  function Person() {
    _classCallCheck(this, Person);

    this.age = 24;
  }

  _createClass(Person, [{
    key: "sayHello",
    value: function sayHello() {
      return 'hello, I am ' + this.age;
    }
  }]);

  return Person;
}();

_defineProperty(Person, "bar", 'bar');

_defineProperty(Person, "onSayHello", function () {
  return this.bar;
});
```

#### class继承

ES6 通过 extend实现继承，在子类的constructor中必须通过super关键字调用父类的构造函数，不然子类无法实例，因为子类没有自己的this对象。super()相当于是Parent.call(this)

ES6的代码
``` js
class Parent {
    constructor(name) {
        this.name = name;
    }
    getName (){
       return this.name
    }
}

class Child extends Parent {
    constructor(name, age) {
        super(name); // 调用父类的 constructor(name)
        this.age = age;
    }
}

var child1 = new Child('kevin', '18');

console.log(child1);
```
子类的__proto__属性
``` js
console.log(Child.__proto__ === Parent)
console.log(Child.prototype.__proto__ === Prarent.prototype)
```
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51d4138134584c47abbcbf3de1796ce9~tplv-k3u1fbpfcp-watermark.image)

ES5的代码

``` js
function Parent (name) {
    this.name = name;
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child
var child1 = new Child('kevin', '18');
console.log(child1);
```


### call的原理
```js
Function.prototype.myCall = function (context) {
    var context = context || window;
    context.fn = this;
    var args =[...arguments].slice(1);
    var result = context.fn(...args)
    delete context.fn
    return result;
}
```
### apply的原理
```
Function.prototype.myApply = function (context) {
      context = context ? context : window
      context.fn = this
      let args = [...arguments][1]
      if (!args) {
        return context.fn()
      }
      let result = context.fn(...args)
      delete context.fn;
      return result
    }
```
### bind 实现
``` js
Function.prototype.bind2 = function (context) {

    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}

```

### new的原理
```js
function objectFactory() {

    var obj = new Object(),

    Constructor = [].shift.call(arguments);

    obj.__proto__ = Constructor.prototype;

    var ret = Constructor.apply(obj, arguments);

    return typeof ret === 'object' ? ret : obj;

};
```

### 参考

https://github.com/mqyqingfeng/Blog/issues/105
https://github.com/mqyqingfeng/Blog/issues/106

《js高级程序设计 第3版》
