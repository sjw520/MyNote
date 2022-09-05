# 1 git常用命令

| 命令名称                             | 作用           |
| ------------------------------------ | -------------- |
| git config --global user.name 用户名 | 设置用户签名   |
| git config --global user.name 邮箱   | 设置用户签名   |
| git init                             | 初始化本地库   |
| git status                           | 查看本地库状态 |
| git add 文件名                       | 添加到暂存区   |
| git commit -m '日志信息' 文件名      | 提交到本地库   |
| git reflog                           | 查看历史记录   |
| git rest --hard 版本号               | 版本穿梭       |

## 1.1 初始化本地库

> git init

## 1.2 查看本地库状态

> git status

## 1.3 添加暂存区

> git add 文件名

### 1.3.1 删除暂存区文件

> git rm --cached 文件名

## 1.4 提交本地库

> git commit -m 'first commit' 文件名 

## 1.5 穿梭版本号

通过git reflog 查看版本号，然后使用git reset --hard 版本号

# 2 分支操作

| 命令名称                   | 作用                          |
| -------------------------- | ----------------------------- |
| git branch 分支名          | 创建分支                      |
| git branch -v              | 查看分支                      |
| git checkout 分支名        | 切换分支                      |
| git merge 分支名           | 把指定 的分支合并到当前分支上 |
| git checkout -b 新分支名称 | 在当前分支创建一个新的分支    |

##  2.1 合并冲突

合并分支时，两个分支在同一个文件的同一个位置有两套完全不同的修改，git无法决定，必须人为决定新代码内容。

> 打开文件进行手动修改

再进行添加暂存区，提交本地库，不要带文件名 git commint -m ' '

# 3 GitHub操作

## 3.1 远程仓库操作

| 命令名称                           | 作用                                                     |
| ---------------------------------- | -------------------------------------------------------- |
| git remote -v                      | 查看当前所有远程地址别名                                 |
| git remote add 别名 远程地址       | 起别名                                                   |
| git push 别名 分支                 | 推送本地分支上的内容到远程仓库                           |
| git clone 远程地址                 | 将远程仓库的内容克隆到本地                               |
| git pull 远程库地址别名 远程分支名 | 将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并 |



## 3.2 团队协作

添加操作仓库成员

settings->Collaboraors->add people

发送链接

fork：将别人仓库拉取到自己仓库

推送到别人仓库：pull requests->new pull request ->create pull request->create pull request

申请合并提交申请：Merge pull request->comfirm merge

## 3.3 免密登录

# 4 idea集成git

## 4.1 配置git忽略文件

1. 创建忽略规则文件 xxxx.ignore(前缀名随便起，建议是git.ignore)

   ```ignore
   .log
   .jar
   ```

2. 在.gitconfig文件中引用忽略配置文件（此文件在windows的家目录中）

   ```
   [core]
   	excludesfile= C:/User/asus/git.ignore
   ```



## 4.2 idea初始化本地库

1. 初始化本地库：VCS->import->version->control->create git repository
2. 添加到暂存区：git->add
3. commit directory
4. commit



## 4.3 idea切换版本

右击版本，checkout Revision 

## 4.4 idea创建分支

git->Repository->Branches

## 4.5 idea合并分支

合并到当前分支：右下角点击分支->merge into current

## 4.6 idea合并冲突分支

1. 右下角点击分支->merge into current
2. merge->手动选择（左边master，右边分支代码，中间没有冲突的代码）

## 4.7 idea代码推送到远程仓库

push->点击中间地址或别名，点击（define remote），输入ssh的url（可以不用设置）->push



注意：push是将本地库代码推送到远程库，如果本地库代码跟远程库代码版本不一致，push的操作是会被拒绝的。也就是说，想要铺设成功，一定要保证本地库的版本要比远程库的版本高。如果本地库的代码版本已经落后，切记要先pull拉取一下远程库的代码，将本地代码更新到最新后，然后再修改，提交，推送

## 4.8 远程拉取

vcs->git->pull

注意：pull是拉取远端仓库代码到本地，如果远程代码和本地库代码不一致，会自动合并，如果自动合并失败，还会涉及到手都冻解决冲突的问题。

