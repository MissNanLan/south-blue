引起对这个问题好奇的原因是因为，发现在日常开发中，即用了JSX.Element,又用了ReactNode，对此颇有些困惑

[stackoverflow](https://stackoverflow.com/questions/58123398/when-to-use-jsx-element-vs-reactnode-vs-reactelement)也有一样的困惑

### ReactNode

可以说是最大的层级


![未命名文件.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/286a86f2af234e2abfc5b4c9c3b60340~tplv-k3u1fbpfcp-watermark.image)

我们可以顺着类型声明往上找可以发现

``` ts
type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined;
```
ReactNode是联合类型，由 `ReactChild、ReactFragment、ReactPortal、boolean、null、undefined`的集合.

- props.children是ReactNode类型

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34d78ebc5d064e3a8ad8d9b9f1e415b7~tplv-k3u1fbpfcp-watermark.image)

- class组件render返回的是ReactNode类型

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e120c3b896d7422e982351d8c7a17080~tplv-k3u1fbpfcp-watermark.image)

[render](https://reactjs.org/docs/react-component.html#render)

#### ReactChild
``` ts
type ReactChild = ReactElement | ReactText;

type ReactText = string | number;
type Key = string | number;

interface ReactElement<P = any, T extends string | JSXElementConstructor<any> = string | JSXElementConstructor<any>> {
        type: T;
        props: P;
        key: Key | null;
  }
```

#### ReactFragment
    
``` ts
interface ReactNodeArray extends Array<ReactNode> {}
type ReactFragment = {} | ReactNodeArray;
```

#### ReactPortal

``` ts
interface ReactPortal extends ReactElement {
        key: Key | null;
        children: ReactNode;
 }
 ```
 
### JSX.Element

与ReactElement类似

``` ts
declare global {
    namespace JSX {
        interface Element extends React.ReactElement<any, any> { }
        }
    }
 ```
由以上可以看出， JSX.Element是继承React.Element. 并且函数式组件一般返回JSX.Element

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/488cfb355db14fff902275f254e1a7ee~tplv-k3u1fbpfcp-watermark.image)

图中函数组件infer类型为JSX.Element

### 其它-Node与Element的区别

Node 是一个接口，各种类型的 DOM API 对象会从这个接口继承

Node的类型

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ef9d44f9b6a411686d45fb2e7f624a4~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e78743f5a48443e8197c235086cf0cc~tplv-k3u1fbpfcp-watermark.image)

[nodeType](https://developer.mozilla.org/zh-CN/docs/Web/API/Node)

  