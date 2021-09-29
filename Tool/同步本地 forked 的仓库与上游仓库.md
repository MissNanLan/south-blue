## 同步本地 forked 的仓库与上游仓库

```
git remote add upstream https://github.com/whoever/whatever.git

git fetch upstream

git  checkout main

git rebase  upstream/master

```

## 更改本地的仓库用户名和邮箱

```
git commit --amend --author="github用户名 <github邮箱>
```
