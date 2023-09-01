# **github、gitee创建仓库与推送代码**

> 创建git仓库

```
git init
git add .
git commit -m "注释"
git remote add origin git@...(SSH)
git push -u origin "master"
```

- `git init`：将当前目录初始化为一个Git仓库。

- `git add .`：将当前目录下所有文件添加到暂存区。

- `git commit -m "注释"`：将暂存区的文件提交到本地仓库，并添加注释。

- `git remote add origin git@...(SSH)`：将本地仓库与远程仓库建立关联，origin是远程仓库的别名，git@...(SSH)是远程仓库的地址。

- `git push -u origin master`：将本地仓库的master分支推送到远程仓库的master分支，并在推送的同时建立追踪关系（-u参数），以后可以直接使用`git push`命令推送修改。

  

  

  