## 小秘诀

新建一个与自己 github 同名的仓库，因为同名的仓库是一个特殊的仓库，README.md 将会出现在首页

## github 卡片统计

### 利用第一个开源库[github-readme-stats](https://github.com/MissNanLan/github-readme-stats)

#### GitHub Extra Pins

``` js
/api/pin?username=anuraghazra&repo=github-readme-stats
```
[点击这里测试](https://github-readme-stats.vercel.app/api/pin?username=anuraghazra&repo=github-readme-stats)


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36d85205514944dd8fc39e364b543305~tplv-k3u1fbpfcp-watermark.image)

#### GitHub Stats Card

``` 
/api?username=MissNanLan
```

[点击这里测试](https://github-readme-stats.vercel.app/api?username=MissNanLan)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f63f96fe50834db49a7f6b1a935f58a8~tplv-k3u1fbpfcp-watermark.image)

- 隐藏个人统计资料

  > Options: &hide=stars,commits,prs,issues,contribs

- 将私人贡献计数添加到总提交计数中

  > Options: &count_private=true

- 显示 icon

  > Options: &show_icons=true

- 显示主题
  > Options: &theme=dark, radical, merko, gruvbox, tokyonight, onedark, cobalt, synthwave, highcontrast, dracula
  - 自定义主题

#### Top Languages Card

```
/api/top-langs/username=MissNanLan
```
[点击这里测试](https://github-readme-stats.vercel.app/api/top-langs/?username=MissNanLan)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76b9d77176bd49cb946aca71885ec637~tplv-k3u1fbpfcp-watermark.image)

#### Wakatime Week Stats

[wakatime](https://wakatime.com/)

```
/api/wakatime?usename= 你的Wakatime用户名
```

[点击这里测试](https://github-readme-stats.vercel.app/api/wakatime?username=MissNanLan)

1、 在 `Wakatime` 安装自己 IDE 工具，比如 `VScode`

2、 在自己本地的 `VSCode` 安装 WakaTime 插件`⌘ + Shift + P`

![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fbc9d80eb8f405a8d45f2ae4be86950~tplv-k3u1fbpfcp-watermark.image)

### 利用 vercel 自己部署

1、 在[vercel](https://vercel.com/)用 github 登陆

2、 fork [github-readme-stats](https://github.com/MissNanLan/github-readme-stats)

3、 点击`New Project`选择你仓库下的`github-readme-stats`

4、 配置环境变量`PAT_1`,这一步很重要，在部署前先配置（我之前都是在构建前再去配置，导致没有生效，这里谁有解答可以告知）

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f5a8b190790400a9c649312b8a6b52f~tplv-k3u1fbpfcp-watermark.image)

`NAME` 是`PAT_1`,`Value`是github的个人授权token

5、创建Personal access token。 

在 github ->settings->Developer settings>Personal access tokens->New personal access token，最后点击`Generate Token`即可

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a211e4f6689949de9375b4b5de9a4472~tplv-k3u1fbpfcp-watermark.image)

6、 点击查看 `domain`，可以自定义
