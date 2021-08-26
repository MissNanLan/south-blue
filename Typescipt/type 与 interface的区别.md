### interface可以扩展，而type不可以

如

``` ts
// interface
 interface Basic {
   name: string;
   age: number;
}

interface Person extends Basic {
  hobby: string[]
}

// type
 type Basic = {
   name: string;
   age: number;
}

type Person & Basic = {
  hobby: string[]
}

```

### interface可以声明合并，type不可以

``` ts
interface Basic {
    name: string
}

interface Basic {
    title: string;
}

const p: Basic = {
    title: 'aa'
}
// 类型 "{ title: string; }" 中缺少属性 "name"，但类型 "Basic" 中需要该属性

type Basic  = {
   name: string
}


type Basic  = {
  title: string;
}

// 标识符“Basic”重复
```


### interface只能声明对象，type可以声明Primitive type、联合类型、元组，当然也包括对象

``` ts
inferface Basic {
  name: string;
}

type Baic = string
type Size = 'string' | 'number' 
type Tu = [1,2,3]
```

其他的情况都可以互换，如声明函数等

``` ts
  type Fuc= (x: number, y: number, desc?: string)=>string
 
  interface Func {
      (x: number, y: number, desc?: string): string
   }
   // 使用
   var foo: Fuc = (x,y,desc)=>{
        return  'aaa'
   }

```