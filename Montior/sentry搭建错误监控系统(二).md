上次没有看到源码的问题解决了。之前没有看到源码，是因为本地跑的，后面我用docker起个服务，发现可以定位到源码，开森~

![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec1692f129?w=1240&h=719&f=png&s=152349)

### 一、上报的规则

#### 1、服务端设置报警的规则
可以在project setting ->alert里面去设置
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec197cdd71?w=1240&h=659&f=png&s=46379)

#### 2、 客户端设置
这是Sentry客户端的一些api。我们用`configureScope`这个来试下
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec1bd60566?w=1240&h=604&f=png&s=78929)
在客户端设置
```
Sentry.configureScope((scope) => {
    scope.setUser({email: 'nanlan.yj@foxmail.com'});
    scope.setExtra('character_name', 'Mighty Fighter');
});
```
构建之后，会看到刚刚设置的用户和`extra Data`

![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec1bdfb812?w=1240&h=360&f=png&s=98871)


![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec1df4bfa1?w=1240&h=531&f=png&s=47878)

此外还有setTag、setExtra、setLevel的方法，具体可以参考 [传送门](https://docs.sentry.io/enriching-error-data/additional-data/?platform=browser#predefined-data)
(ps:withScope和configureScope的区别在于作用域不同

![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec4b29806a?w=1240&h=241&f=png&s=148179)

这段话大概意思就是`congfigure-scope`，当前scope改变的话，其他的也会改变。而`with-scope`相当于创建一个当前`scope`的副本，并且与原先的完全隔离直到函数调用完成


### 二、区分环境
```
process.env.NODE_ENV === "production" && Sentry.init({
    release: 'v1.0.6',
    dsn: 'https://f36c12db53d94e21b3eeecdf890dfcc4@o384506.ingest.sentry.io/5227797',
    environment: 'staging',
    debug: true,
    // beforeBreadcrumb(breadcrumb, hint) {
    //     console.log(breadcrumb)
    //     return breadcrumb.category === 'ui.click' ? null : breadcrumb;
    //   },
});
```
可以看到刚刚设置的环境，你可以提出实际需求，设置`local`、`relase`
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec4ec26803?w=1240&h=474&f=png&s=175842)
当然还可以在`project setting`中`environment`中管理

![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec52fbb0c3?w=1240&h=559&f=png&s=110686)
更多关于`Sentry.init`的配置参数，[传送门](https://docs.sentry.io/sdks/javascript/config/basics)

### 三、创建发布并关联提交
#### 1、首先在 `Integration`里面安装github

在设置->Integration->安装 github
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec557322d0?w=1240&h=576&f=png&s=195736)

配置你的github地址
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec5762419d?w=1240&h=647&f=png&s=94249)
#### 2、创建发布并关联提交
[官网传送门](https://docs.sentry.io/workflow/releases/?platform=browsernpm#create-release)
- 用`CLI` 创建发布并提交

在自己的项目录下创建`xx.sh`的文件

```
#!/bin/bash  

export SENTRY_AUTH_TOKEN=
export SENTRY_ORG=
export VERSION=v1.0.6
# 创建一个发布，可以new多个
sentry-cli releases new -p nanlan-blog  $VERSION
# 与commit相关联，默认是最新的20次
sentry-cli releases set-commits --auto $VERSION
# 最终发布
sentry-cli releases finalize $VERSION
```

- 运行`sh xx.sh`即可

#### 3、查看是否成功

- 在`发布`的菜单下可以看到，证明操作成功
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec5ae59bf2?w=1240&h=191&f=png&s=36604)

#### 4、链接`git issue`
在问题最右侧的面板点击`Link Github Issue`创建一个链接
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec7eb8ec0a?w=1240&h=761&f=png&s=222902)
可以看到github的issue增加了一个

![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec8b050a50?w=1240&h=452&f=png&s=108139)

问题的issue id
![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec8be9444c?w=1240&h=592&f=png&s=181974)
下次提交的时候，提交日志加上issue的id,那么这个问题就会从`unresolved`变成`solved`
```
git commit -m "fix NANLAN-BLOG-E"
```

![image](https://user-gold-cdn.xitu.io/2020/6/7/1728dbec8cceb765?w=1240&h=804&f=png&s=249416)
