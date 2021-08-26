### 一、 前言
很多公司都会搭建自己的错误监控系统。我自己想尝试搭建监控系统，源于我们公司内部小程序线上报错，不能及时定位问题，刚好看到有博文在推荐`sentry`,于是我饶有兴趣去关注它。
### 二、注册服务
- 在官网上注册服务
- 搭建自己的服务

 ### 1、在官网上注册服务
 
 #### 1）注册个人或公司信息
 [官网传送门](https://sentry.io/welcome/)
 
![](https://user-gold-cdn.xitu.io/2020/5/17/17221f7157e77b17?w=2850&h=1352&f=png&s=1429784)

依照个人或公司需求填写如上信息。因为我是个人项目测试，填写的是自己相关的信息


 #### 2）创建项目
 
![](https://user-gold-cdn.xitu.io/2020/5/17/17221fee2996a6b1?w=2876&h=1226&f=png&s=545181)
创建之后会有个快速的指导


![](https://user-gold-cdn.xitu.io/2020/5/17/17222018e75c070a?w=2878&h=1568&f=png&s=723457)

我这里是`react`项目，在`index.jsx`的文件里面输入,直接拷贝过来即可

![index.jsx](https://user-gold-cdn.xitu.io/2020/5/17/1722200b8050574a?w=1688&h=1246&f=png&s=238708)
#### 3）根据指导的提示，测试错误
```
return <button onClick={methodDoesNotExist}>Break the world</button>;
```
#### 4） 验证错误，会看到控制台已经发送sentry请求了
![](https://user-gold-cdn.xitu.io/2020/5/17/1722204e6ac47354?w=2878&h=684&f=png&s=216641)
打开sentry后台就可以看到错误已经上报了
![](https://user-gold-cdn.xitu.io/2020/5/17/172220632b1625d9?w=2838&h=976&f=png&s=411324)

### 2、自己搭建服务(docker)
 [官网镜像传送门](https://github.com/getsentry/onpremise)
 > 要求
Docker 17.05.0+
Compose 1.23.0; 
最小的储存空间
2400RMB

其实方法很简单的。就是`git clone`仓库，然后`./install.sh`,这个步骤很耗时间，因为它依赖非常多。如果顺利的话，安装成功直接运行
```
docker-compose up -d
```

sentry-onpremise启动成功
![](https://user-gold-cdn.xitu.io/2020/5/17/1722213a303e5330?w=2878&h=1482&f=png&s=290710)

打开`localhost:9090`就可以和在官网上一样

![](https://user-gold-cdn.xitu.io/2020/5/17/172221486f97423f?w=2872&h=1442&f=png&s=526018)

安装的时候可能会报，我还给官网提了一个issue。提完之后发现，已经有类似的issue了[https://github.com/getsentry/onpremise/issues/490](https://github.com/getsentry/onpremise/issues/490)
```
Connection to Kafka failed
```
解决办法就是
```
docker-compose down
docker volume rm sentry-kafka sentry-zookeeper
docker volume rm sentry_onpremise_sentry-kafka-log sentry_onpremise_sentry-zookeeper-log
./install
```
最后我顺利跑起来了。公司的话可以叫运维在服务器上部署，记住至少需要`2.4G`的空间和依赖于`docker`

### 三、上传sourceMap到sentry

上传sourceMap的目的是为了出错，能够方便具体定位到哪行源代码

官网了提供了几种方式

![](https://user-gold-cdn.xitu.io/2020/5/17/172222a680a4c129?w=1650&h=410&f=png&s=75169)

[souecemap传送门](https://docs.sentry.io/platforms/javascript/sourcemaps/)


 我这里用了`sentry-cli`,所以下面演示用`sentry-cli`
 #### 1）安装sentry-cli
 ```
 npm i -g @sentry/cli
 sentry-cli -V  查看版本
 ```
 #### 2） 登录拿到`Auth Tokens`
 ```
 sentry-cli 
 ```
 默认是连接到`senty.io`,如果想要连接自己的输入，则
 ```
 sentry-cli --url https://myserver.invalid/ login
 ```
 之后打开
 
![](https://user-gold-cdn.xitu.io/2020/5/17/17222336282a43e5?w=2878&h=1256&f=png&s=601747)

#### 3）在自己的根目录下新建`~/.sentryclirc`
 - mac的`~`代表`/user/你的用户名`
 - 建议用vim创建比较方便
 ```
   vim ~/.sentryclirc/
   i 键进入插入模式
   esc 退出插入模式
   :wq!  保存并退出
   :q!   不保存退出
 ```

将`auth token`拷贝过来，其他同时也需要配置
![](https://user-gold-cdn.xitu.io/2020/5/17/172223a3ee846d7e?w=1140&h=904&f=png&s=185105)
```
[auth]
token=7c44fb0b77b7430b93cdad42cd8f5ad6386fcc8d7dd6477e86b48c7aa490d793
[defaults]
url=https://sentry.io/
org=116e23a13af3
project=nanlan-blog
```
之前我纠结的org和project到底怎么查看，后面查了一些资料。
在`Organzation Settings`的`General Settings`里面查看`org`

![](https://user-gold-cdn.xitu.io/2020/5/17/17222bdbc16354a1?w=2736&h=1400&f=png&s=484670)
在`Project Settings`的`General Settings`里面查看`project`

![](https://user-gold-cdn.xitu.io/2020/5/17/172224c3f5e48874?w=2790&h=1384&f=png&s=580526)


#### 4) 上传

![](https://user-gold-cdn.xitu.io/2020/5/17/172224686d3e0419?w=1672&h=970&f=png&s=199746)

- 上传的时候创建一个`release`名字
- 然后使用`upload-sourcemaps`上传，后面的url是本地的`url`,即是打包后`build`或者`dist`文件加下面的js
- 默认是以`.map`或者`.js`的文件，你可以更改扩展名
- 最终发布

这里有个坑。当时我一直上传不上去,忘记什么问题了。 我是这么上传的
```
sentry-cli --log-level=debug releases files v1.0.2  upload-sourcemaps ./build --rewrite
```
`./build`是自己本地打包的文件夹

上传成功
![](https://user-gold-cdn.xitu.io/2020/5/17/17222e00b609cf7d?w=1322&h=630&f=png&s=124852)

记住免费是的40MB

在`release`菜单下面，根据刚刚设置的`release`名字，可以查看`Artifacts`下面有刚刚上传的文件

![](https://user-gold-cdn.xitu.io/2020/5/17/17222e7c71b55260?w=2878&h=1490&f=png&s=619888)

官网有这种写法
```
sentry-cli releases files VERSION upload-sourcemaps . --url-prefix '~/scripts'
```
- --url-prefix是线上的url
- ~是你网站的域名，比如`http://localhost:9000`

以下来自官方的截图

![](https://user-gold-cdn.xitu.io/2020/5/17/17222fcf3a17c687?w=1414&h=370&f=png&s=48762)

有个问题，到现在还没有解决，就是出错之后我并没有定位到源码的位置，不知道为啥
`release`是对的
### 四、设置时区

![](https://user-gold-cdn.xitu.io/2020/5/17/17222ff3c7de60e2?w=2878&h=1626&f=png&s=522467)

篇(二)会解决上一个问题（目前还在看），以及
- 上报的规则
- 区分环境
- 与`git commit` 关联
- ...

才疏学浅，有什么意见或建议提出来，相互交流