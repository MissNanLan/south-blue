项目中的一些npm太旧了，几乎几年的时间都没有更新过。很久之前我想过，当这些npm在不断地更新升级的时候，`package.json`中依赖的包有这么多，大约十几二十几个，这还是你看到的，如何更新升级。下面先补下 `npm install`一些相关的命令

#### npm install 常用参数
- **npm install**: 模块会安装到`node_modules`文件夹下面，显示在`package.json`下面的`dependencies`
- **-D，--save-dev**: 模块会安装到`node_modules`文件夹下面,同样显示在`devDependencies`
- **-S,--save**: 模块会安装到`node_modules`文件夹下面,同样显示在`dependencies`
- **-g,--global**

- **-P，--save-prod**: 和 `npm install` 一样
- **-O, --save-optional**:（从未用过诶）模块会安装到`node_modules文件夹下面`,显示在`package.json`里面的`optionalDependencies`
- **--no-save**: 阻止保存在`dependencies`
- **-E, --save-exact**:（未用过诶）保存一个精确的版本在`dependencies`,而不是使用`npm’s default semver range operator`
- **-B, --save-bundle**: （未用过诶) 保存在`bundledDependencies`里面

更多关于`npm install` 的参数,点击[https://docs.npmjs.com/cli/install](https://docs.npmjs.com/cli/install)

#### 升级npm 包

1、 `npm outdated`  查看你旧的包到底有哪些

![](https://user-gold-cdn.xitu.io/2020/7/23/1737a569b85cb324?w=1292&h=1068&f=png&s=315484)

2、 安装npm-check-updates
```
$ npm install -g npm-check-updates
$ ncu
$ ncu -u
```
也可以局部更新

```
$ ncu one, two, three
```

官方地址 [https://www.npmjs.com/package/npm-check-updates](https://www.npmjs.com/package/npm-check-updates)

3、npm update

据个人实践，有时输入这个命令并没有反应，不知道咋回事，有时只是更新了 `dependencies`，当然也可以加`-D`的参数。我理解到的是，它更新的规则不是直接更新到最新的包，而是根据`caret dependencies`插入依赖符^和`Tlide dependencies`~波浪依赖符的匹配规则

以下是官网截图
![](https://user-gold-cdn.xitu.io/2020/7/23/1737aa2037a8fa3f?w=1934&h=1134&f=png&s=149345)

4、 版本号的规则 `MAJOR,MINOR PATCH`
 比如`1.4.2`
- 1 是 `major version`,表示是一个大的版本，可能向后不兼容;
- 4 是`minor version`，表示增加了新的功能，并且可以向后兼容;
- 2 是`patch version`，修复bug，向后兼容

5、 更新有风险，谨慎操作

![](https://user-gold-cdn.xitu.io/2020/7/23/1737ab8ad3c8d0b2?w=192&h=192&f=png&s=31222)


更多链接：
[semver](https://docs.npmjs.com/misc/semver)、
[npm-update](https://docs.npmjs.com/cli/update#tilde-dependencies)、
[npm升级所有可更新包](https://segmentfault.com/a/1190000005857342)

