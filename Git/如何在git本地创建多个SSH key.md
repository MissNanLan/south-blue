## 背景

公司用 gitlab，git 的地址用的 ssh 地址。众所周知，用 ssh 地址的话则需要在本地产生一个私钥和公钥；而自己的项目用的是 github，git 也用的是 ssh 地址
那么这个时候小心你的 .ssh 不要被覆盖了

## 具体步骤

### 生成一个 github 的 SSH key

```
ssh-keygen -t rsa -C '你github绑定的邮箱' -f ~/.ssh/github_id_rsa
```

### 生成一个 gitlab 的 SSH key

```js

ssh-keygen -t rsa -C '你公司绑定的邮箱' -f ~/.ssh/gitlab_id_rsa
```

### 在.ssh 下新建一个 config 的文件

```
touch config

// 把以下拷贝进去
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa

# gitlab
Host gitlab.zatech.online
HostName gitlab.zatech.online
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitlab_id_rsa
```

以上步骤正常的话，你就会在~/.ssh 下面看到

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/185ce412a57b41339109dbcee53a5fa5~tplv-k3u1fbpfcp-watermark.image)

### 分别在 gitlab 和 github 上配置 SSH keys

拷贝公钥

#### github

settings->SSH and GPG keys->New SSH key

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/557b0341fdd347b19d351caac3466f93~tplv-k3u1fbpfcp-watermark.image)

#### gitlab 也是如此

## 参考

[Git 配置多个 SSH-Key](https://gitee.com/help/articles/4229#article-header0)

[服务器上的 Git - 生成 SSH 公钥](https://git-scm.com/book/zh/v2/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%9F%E6%88%90-SSH-%E5%85%AC%E9%92%A5)
