Proxy的出现，给vue响应式带来了极大的便利，比如可以直接劫持数组、对象的改变，可以直接添加对象属性，但是兼容性可能会有些问题

## vue中的defineProperty和Proxy的区别


![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e4598dd281e430ca23f76432a3edf94~tplv-k3u1fbpfcp-watermark.image)

### Proxy可以劫持的数组的改变,defineProperty 需要变异
 
#### defineProperty 中劫持数组变化的变异的方法
 可以理解为在数组实例和原型之间，插入了一个新的原型的对象，这个原型方法实现了变异的方法，也就真正地拦截了数组原型上的方法
 
 我们来看下vue2.x的源码
 
 ```js
 // vue 2.5.0
 var arrayProto = Array.prototype;
 var arrayMethods = Object.create(arrayProto);  // Array {}
 function def(obj, key, val, enumerable) {
		Object.defineProperty(obj, key, {
			value: val,
			enumerable: !!enumerable,
			writable: true,
			configurable: true
		});
}
 var methodsToPatch = [
		'push',
		'pop',
		'shift',
		'unshift',
		'splice',
		'sort',
		'reverse'
	];

	/**
	 * Intercept mutating methods and emit events
	 */
	methodsToPatch.forEach(function(method) {
		// cache original method
		var original = arrayProto[method];  
        // 比如 method是push，则结果为
        // ƒ push() { [native code] }
		def(arrayMethods, method, function mutator() {
			var args = [],
				len = arguments.length;
			while (len--) args[len] = arguments[len];

			var result = original.apply(this, args);
			var ob = this.__ob__;
			var inserted;
			switch (method) {
				case 'push':
				case 'unshift':
					inserted = args;
					break
				case 'splice':
					inserted = args.slice(2);
					break
			}
			if (inserted) {
				ob.observeArray(inserted);
			}
			// notify change
			ob.dep.notify();
			return result
		});
	});
    
    /**
	 * Observe a list of Array items.
	 */
	Observer.prototype.observeArray = function observeArray(items) {
		for (var i = 0, l = items.length; i < l; i++) {
			observe(items[i]);  // 后续的逻辑
		}
	};

 ```
 #### Proxy可以直接劫持数组的改变
 ```js
    let proxy = new Proxy(fruit, {
        get: function (obj, prop) {
            return prop in obj ? obj[prop] : undefined

        },
        set: function (obj, prop, newVal) {
            obj[prop] = newVal
            console.log("newVal", newVal)  // 输出{ name: "lemon", num: 999 }
            return true;
        }
    })
    proxy.push({ name: "lemon", num: 999 })
    console.log(fruit)
 ```
 ![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19bce47fc5a64a28806628dafbc41293~tplv-k3u1fbpfcp-watermark.image)
### Proxy代理可以劫持对象的改变，defineProperty需要遍历
#####  defineProperty
 ```js
      let fruit = {
           "apple": 2,
           "pear": 22,
           "peach": 222
     }
     Object.keys(fruit).forEach(function (key) {
            Object.defineProperty(fruit[i], key, {
                enumerable: true,
                configurable: true,
                get: function () {
                    return val;

                },
                set: function (newVal) {
                    val = newVal;  //  输出 newVal 888
                    console.log("newVal", newVal)
                }
            })
        })
       fruit.apple = 888
 
 ```
 #### Proxy
 ```js
      let fruit = {
           "apple": 2,
           "pear": 22,
           "peach": 222
     }
     let proxy = new Proxy(fruit, {
        get: function (obj, prop) {
            return prop in obj ? obj[prop] : undefined

        },
        set: function (obj, prop, newVal) {
            obj[prop] = newVal
            console.log("newVal", newVal) //  输出 newVal 888
            return true;
        }
    })
     proxy.apple = 888
 ```

###  Proxy代理可以劫持对象属性的添加，defineProperty用this.$set来实现
#### defineProperty，如果属性不存在，则需要借助this.$set
``` js
<div id="app">
    <span v-for="(value,name) in fruit">{{name}}:{{value}}个 </span> 
    <button @click="add()">添加柠檬</button>
</div>
<script src="https://unpkg.com/vue"></script>
<script>
    new Vue({
        el: '#app',
        data() {
            return {
                fruit: {
                    apple: 1, 
                    banana: 4, 
                    orange: 5 
                }
            }
        },

        methods: {
            add() {
                    this.fruit.lemon = 5;   //  不会让视图发生变化
                // this.$set(this.fruit,"lemon",5)   // this.$set可以
            }
        }
    })
</script>
```

```js
    Object.keys(fruit).forEach(function (key) {
        Object.defineProperty(fruit, key, {
            enumerable: true,
            configurable: true,
            get: function () {
                return val;

            },
            set: function (newVal) {
                val = newVal;
                console.log("newVal", newVal)  // 根本没有进去这里
            }
        })
    })
```

#### Proxy 直接可以添加属性
``` js
// vue 3
<div id="app">
    <span v-for="(value,name) in fruit">{{name}}:{{value}}个 </span>
    <button @click="add()">添加柠檬</button>
</div>
<script src="https://unpkg.com/vue@next"></script>
<script>
    Vue.createApp({
        data() {
            return {
                fruit: {
                    apple: 1,
                    banana: 4,
                    orange: 5
                }
            }
        },

        methods: {
            add() {
                this.fruit.lemon = 5;  // 这样子是可以的
            }
        }
    }).mount('#app')  // vue 3 不再是使用el属性，而是使用mount
</script>
```

```js
    let proxy = new Proxy(fruit, {
        get: function (obj, prop) {
            return prop in obj ? obj[prop] : undefined

        },
        set: function (obj, prop, newVal) {
            obj[prop] = newVal
            console.log("newVal", newVal)  // lemon, 888
            return true;
        }
    })
    proxy.lemon = 888
```
## Proxy
### 其他属性
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/548c44e7c28b47d5bc08eb6117677105~tplv-k3u1fbpfcp-watermark.image)
### 应用场景 promisify化
用Proxy写一个场景，请求都是通过回调，如果我们需要用promise包一层的话，则可以
``` js
// server.js
// 假设这里都是回调
export const searchResultList = function (data, callback, errorCallback) {
  axios.post(url, data, callback, errorCallback)
}
```

 ```js
// promisify.js
 import * as server from './server.js'
const promisify = (name,obj) => (option) => {
  return new Promise((resolve, reject) => {
    return obj[name](
      option,
      resolve,
      reject,
    )
  })
}
const serverPromisify = new Proxy(server, {
  get (target,prop) {
    return promisify(prop, server)
  }
})
export default serverPromisify

```

使用
```js
// index.js
import serverPromisify from './serverPromisify'
serverPromisify.searchResultList(data).then(res=>{

})

```
## 总结
|属性|   数组劫持 | 是否监劫持对象 | 对象属性的添加的劫持 |劫持操作
|------|------------|------------|------------|------------|
| defineObjectProperty  |   不可以    | 不可以，对于单个的key可以    | 不可以 | 2种
| Proxy  | 可以     | 可以     | 可以  | 12种


如有不正确，望请指出

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93d90da947234c27858dc76524faf3bd~tplv-k3u1fbpfcp-watermark.image)

留下一个疑问，既然兼容性不是很好，那么尤大是怎么处理polyfill呢

