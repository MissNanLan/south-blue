typescript 正式版本已经到[4.2](https://devblogs.microsoft.com/typescript/)，
其对应的实现都能在 `lib.es5.d.ts`中找到
### Partial < Type>

- 作用：构建一个类型的所有属性都设置为可选的类型

- 使用

``` ts
interface Todo {
  title: string;
  description: string;
}

function updateTodo (todo: Todo, fieldsToUpdate: Partial<Todo>) {
    return { ...todo, ...fieldsToUpdate }
  }
  
 updateTodo({
    title: 'nanlan',
    description: 'description'
    },
    { 
     title:'xiaoju'   // 可选
    })
```

- 实现 

``` ts
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```
- keyof 与in
`keyof`可以取得一个对象接口所有的`key`，`in`遍历枚举类型,后面的内容都会用到

如 

``` ts
type T = keyof Todo ->  // "title" | "description"
```

``` ts
type Obj = {
 [p in Todo] : Todo[p]   //  title: string; description: string;
}
```

### Required< Type>

- 作用：构建一个类型的所有可选的属性都设置为比必填的类型，与`Partial`相反

- 使用
``` ts
interface Todo {
  title?: string;
  description?: string;
}

const Obj: Required<Todo>={
   title: "nanlan"
}
// 类型 "{ title: string; }" 中缺少属性 "description"，但类型 "Required<Todo>" 中需要该属性
```
- 实现
``` ts
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```
- -？
上文对应的`-？`代表着去掉可选，与之对应的还有`+？`，两者正好相反

### Readonly< Type>
- 作用： 将所有的属性设为只读
- 使用

``` ts
type A = {
  a: number;
  readOnly b: number;
};

const c: Readonly<A> = {
 a: 1,
 b: 2
}

c.b = 3 // NO
c.a = 2 // NO

```
- 实现
``` ts
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```
是不是发现蛮简单的

### Record<Keys,Type>

- 作用：Keys变成keys，Type变成values
- 使用

``` ts
interface Todo {
  title: string;
  description: string;
}

type Todo1 = 'key1' | 'key2'

const obj: Record<Todo1, Todo> = {
    key1: { title: 'nanlan', description: 'ddd' },
    key2: { title: 'nanlan', description: 'ddd' },
  }
```
注： Keys只能是`string|number|symbol`
- 实现

``` ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

### Pick<Type, Keys>
- 作用： 从Type选择一组Keys属性

- 使用

``` ts
export interface Todo {
    remark: string;
    required: boolean;
    hobby: string[];
}

const A:Pick<Todo,'remark'|'required'> ={
'remark': '111',
required: false
}

// 表示A只能有remark、required
```

- 实现

```
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
 // K extends keyof T,这句表示，确保K是T的子集
```

### Omit<Type, Keys>
- 作用： 从Type选择所有属性，然后在删除Keys
- 使用
``` ts
export interface Todo {
    remark: string;
    required: boolean;
    hobby: string[];
}
type A: Omit<Todo,'remark'|'required'>
// 表示Type A只能有hobby
```
- 实现
``` ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```
出现个有个Exclude，接着介绍

### Exclude<Type, ExcludedUnion>
- 作用：通过从Type中排除可分配给ExcludedUnion的所有联合成员来构造类型，类似于差集
- 使用
``` ts
type Todo:Exclude<"a" | "b" | "c", "a">
//  type Todo = "b" | "c"
```
- 实现
``` ts
type Exclude<T, U> = T extends U ? never : T;
```
- 与Omit的区别
Exclude 的Type是联合类型，啥时联合类型，接着讲

### 联合类型
``` ts
 let a: string | string[];
 
 // 表示a可以是string或者是string[]
```
### 交叉类型
``` ts
type A = {
  a: string;
}

type B = {
  b: number;
}

type C = A & B

// 集合A和B所有的功能

```

### Extract<Type, Union>
- 作用： 通过从Type中提取可分配Union的所有联合成员来构造类型，类似于交集
- 使用
```
 type Todo:Exclude<"a" | "b" | "c", "a"> = { 'a' }
```
- 实现
``` ts
type Exclude<T, U> = T extends U ? T : never;
```

### NonNullable<Type>
- 作用: 通过从Type中排除null和undefined来构造类型
- 使用
```
 const A:NonNullable<string|boolean|null|undefined>  
 // A只能是string或者boolean，不能是null或undefined
```
- 实现
``` ts
  type NonNullable<T> = T extends null | undefined ? never : T;
```
- 思考

NonNullable 怎么使用于对象
```
type Todo = {
 n: null;
 u: undefined;
 s: string;
 nn:number
}
// 要求定义的对象只能是s和nn
```


### Parameters<Type>
- 作用： 从函数类型type的形参中使用的类型构造元组类型
- 使用
``` ts
type A = Parameters<(s: string) => void>
const obj: A = ['11']

type B = Parameters<<T>(arg: T) => T>;
const obj1:B = [1] // 任意类型都可以

type C = Parameters<string>
// 类型“string”不满足约束“(...args: any) => any”
```
- 实现
``` ts
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
```
出现个infer，下面会讲


### infer

表示在 extends 条件语句中待推断的类型变量

```
```


### ConstructorParameters<Type>
- 作用：从构造函数类型的类型构造元组或数组类型。它产生一个包含所有参数类型的元组类型(如果type不是函数，则该类型为never)

- 使用
``` ts
type Todo = ConstructorParameters<FunctionConstructor>;
// type Todo = string[]
// const obj: Todo = ['111','222']
```
看看FunctionConstructor的是怎么定义

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac1813b57d9346b6b3c440b77e4e63bb~tplv-k3u1fbpfcp-watermark.image)

看看源码就明白了,`T extends new (...args: any)`
- 实现

``` ts
type ConstructorParameters<T extends new (...args: any) => any> = T extends new (...args: infer P) => any ? P : never;
```

### ReturnType<Type>
- 作用: 构造一个由函数的返回类型组成的类型

- 使用
``` ts
type ToDo = ReturnType<() => string>
const obj: ToDo = '11'

type ToDo1 = ReturnType<(s: string) => void>;
const obj: ToDo1 = void

```
- 实现

``` ts
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

### InstanceType<Type>
- 作用: 构造由构造函数的实例类型组成的类型
- 使用
``` ts
type  Todo = InstanceType<never>
const obj: Todo = nerver
```
- 实现
``` ts
type InstanceType<T extends new (...args: any) => any> = T extends new (...args: any) => infer R ? R : any;
```


### Intrinsic String Manipulation Types
- 作用: 内部字符串操作类型
- Uppercase<StringType> 转化成大写字母
- Lowercase<StringType>转化成小写字母
- Capitalize<StringType> 首字母大写
``` ts
const s: Capitalize<'aabb'> = 'Aabb'
```
- Uncapitalize<StringType> 首字母小写
``` ts
const s: Uncapitalize<'Aabb'> = 'aabb'
```
- 实现
``` ts
type Uppercase<S extends string> = intrinsic;
```

### 几种令人迷惑的类型
- nerver： 代表一种不存在的状态，它的返回值为 never 类型)，此外`never`可以分配给任何一种类型，但是任何一种类型不能分配给never（除了nerver自身）。 表示函数返回值，在函数永不返回时或者总是抛出错误


- void: 返回空值

```
  const func = ():void=> {
    // 
  }  // it's ok 
  
  const foo = ():nerver=> {
    throw new Error('err')
    // 或者 while(true){}
  }  // bad 
  
```

- unknown： 表示any类型，与any极为相似，但是比`any`更加安全，因为使用`unknown都是不合法的`
- any：表示`any`类型
```
let obj:any={}
obj.a = 'aaa' //it's ok

let obj:unknow={}
obj.a = 'aaa' // bad

```
- 思考题
``` ts
 type NonNullableObject<T> = { [P in keyof T]: Partial<NonNullable<T[P]>>}
```
测试
``` ts
type Todo = {
  s: string;
  n: null;
  u: undefined;
}

 const obj: NoNullableObject<Todo> = {
    s: '111',
    u:undefined,
  }
 // 不能将类型“undefined”分配给类型“never”
```

你觉得还有其他的方法么

### type与interface的区别


### 参照
- [官网](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype)
- [type-challenges](https://github.com/type-challenges)
- [TS 一些工具泛型的使用及其实现](https://zhuanlan.zhihu.com/p/40311981)


