## 小秘诀

新建一个与自己 github 同名的仓库，因为同名的仓库是一个特殊的仓库，README.md 将会出现在首页

## github 卡片统计

### 利用第一个开源库[github-readme-stats](https://github.com/MissNanLan/github-readme-stats)

#### GitHub Extra Pins

``` js
/api/pin?username=anuraghazra&repo=github-readme-stats
```

![Readme Card](https://github-readme-stats.vercel.app/api/pin/?username=MissNanLan&repo=github-readme-stats&show_owner=true)

#### GitHub Stats Card

```
/api?username=MissNanLan
```

![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=MissNanLan&count_private=true)

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

[![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=MissNanLan)](https://github.com/anuraghazra/github-readme-stats)

#### Wakatime Week Stats

[wakatime](https://wakatime.com/)

```
/api/?usename= 你的Wakatime用户名
```

1、 在 Wakatime 安装自己 IDE 工具，比如 VScode

2、 在自己本地的 VSCode 安装 WakaTime 插件
⌘ + Shift + P

![image](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fbc9d80eb8f405a8d45f2ae4be86950~tplv-k3u1fbpfcp-watermark.image)

### 利用 vercel 自己部署

1、 在[vercel](https://vercel.com/)用 github 登陆
2、 fork [github-readme-stats](https://github.com/MissNanLan/github-readme-stats)
3、 点击`New Project`选择你仓库下的`github-readme-stats`
4、 配置环境变量`PAT_1`,这一步很重要，在部署前先配置（我之前都是在构建前再去配置，导致没有生效，这里谁有解答可以告知）
![image](https://github.com/MissNanLan/south-blue/blob/main/assets/variable.png)
NAME 是PAT_1
Value  是github的个人授权token
5、创建Personal access token。 在 github ->settings->Developer settings>Personal access tokens->New personal access token，最后点击`Generate Token`即可
6. 点击查看 domain，可以自定义
