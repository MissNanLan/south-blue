踩过坑之后回头发现其实很简单，我尽量以简单的文字去描述

## sentry部署
### 官网注册
如果具体应用在公司一般是不推荐的，想了解大概流程可以参考我这篇

[篇（一）](https://juejin.im/post/6844904161415266317)
### 公司部署（推荐）
不想学习部署的同学可以直接跳过，实际工作中部署不是我们的活。运维会比较专业，但是我这里还想梳理下部署的流程以及踩的坑。

[docker镜像](https://github.com/getsentry/onpremise)

这个不止一次造运维吐槽过，依赖非常多，而且一不小心还会报错，不过看`github`更新的还是挺频繁的。如果运维自己用`docker`部署的话，自己去写`dockerFile`和`docker-compose`，工作量非常大的

#### 1、拉取镜像

这个过程非常漫长，如果遇到报错的话，可以去`github`的`issue`上面找下。我之前碰到过关于`kfaka`的问题，在他们的issue中找到了同样的错误。最终输入`localhost:9000`即可访问
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3caf620f9e5c46b5842fda770683028b~tplv-k3u1fbpfcp-zoom-1.image)

#### 2、邮件配置
拉取成功之后输入`localhost:9000`即可访问，在`config.yml`配置邮件信息，这里以企业微信邮箱为例
```
mail.backend: 'django_smtp_ssl.SSLEmailBackend'  # Use dummy if you want to disable email entirely
mail.host: 'smtp.exmail.qq.com'
mail.port: 465
mail.username: ''
mail.password: '1996810jyjJYJ'
mail.use-tls: true
# The email address to send on behalf of
mail.from: '你的邮件地址'
```
安装了一个插件`django_smtp_ssl`.安装完毕之后需要在`requirements.txt`加安装了一个插件`django_smtp_ssl`.
```
pip install django-smtp-ssl
```
之后需要在`requirements.txt`加上
```
# Add plugins here
# sentry-dingtalk-new
django-smtp-ssl~=1.0
```

## sentry在应用中怎么使用
在这里遇到的坑就是怎么也收集不了后端报的错误，原来是忘记自己`Sentry.captureException`这个api了，[篇二](https://juejin.im/post/6844904182529392647#heading-1)有提到过sentry有多个api
```
axios.interceptors.response.use(
  (response) => {
    return Promise.resolve(response);
  },
  (error) => {
   // 关键是这句
    Sentry.captureException(new Error(error));
    return Promise.reject(error);
  }
);
```


## sentry工作台
[篇二](https://juejin.im/post/6844904182529392647)我这篇文章中有提到过。这里讲下我在实际应用中用到的遇到的问题以及解决办法，之前提到过，设置警报的规则有这些

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ce0b069509d42c6b8593f2cf116e066~tplv-k3u1fbpfcp-zoom-1.image)
那我们现在遇到一个问题就是。某些错误上报只给某些人发送，刚好不是有个`setTag`api，利用起来便是

### 1、在代码中加入以下代码
```
  Sentry.configureScope(function (scope) {
  //  设置tag
      scope.setTag("specialError", "specialError");
      scope.setUser({
        "username": "nanlan",
      });
    });
    Sentry.captureException(new Error(error));
```
### 2、选中tag匹配的条件
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca01ff41226940da9c1a6b58a7d175d3~tplv-k3u1fbpfcp-zoom-1.image)

### 3、选中发送的动作
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b27bb43b00e045debde6a92e97a47885~tplv-k3u1fbpfcp-zoom-1.image)

## 总结
这篇文章主要讲了些自己分别在部署（主要是拉取镜像和邮件发送）、业务场景中踩的一些坑（上报不了服务器错误），不同的场景设置不同的警报规则，以后若有什么坑，将随时记录下来

[sentry搭建错误监控系统（二)](https://juejin.im/post/6844904182529392647)

[sentry搭建错误监控系统（一)](https://juejin.im/post/6844904161415266317)