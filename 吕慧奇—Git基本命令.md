## GIT是什么？
* Git是目前世界上最先进的分布式版本控制系统。
## Git的工作流程

* git 是分布式版本控制系统，每一台客户端都是一个独立的Git仓库(有git的全套机制)
  * 一个Git仓库分为三个区域：
    * 1.工作区：平时写代码的地方。
    * 2.暂存区:把一些写好的代码暂时存储的地方。
    * 3:历史区：生成一个版本记录的地方。
## Git的安装
* 这里就不详细说怎么安装了，根据不同的系统，安装的方式也不同，网上有很多的教程。我想说的是，安装完成后，还需要最后一步设置，在命令行输入：
```python
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```
* 因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和邮箱。
## 创建版本库
#### 什么是版本库呢？
* 简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。
* 找一个文件夹，创建一个空目录；
```python
$ mkdir directory
$ cd directory
$ pwd
/Users/project/directory
```
* 通过` git init `命令把这个目录变成Git可以管理的仓库：
```python
$ git init
```
* 创建完成后，会在项目的根目录中展示，git这个文件（隐藏文件）：有这个 `.git`文件
  才叫做git仓库，没有则不能被称为git仓库(因为暂存区和历史区的内容都是存储在这个
  文件夹中的)。
####  把工作区的内容提交到暂停区
* 把某个文件提交到暂存区：
```python
$git add "xxx"
```
*  把所有修改的文件(修改和新增的包含，删除不包含)：
```python
$git add . 
```
* 把所有修改的文件(包含修改和删除的，但是不包含新增的):
```python
$ git add -u
```
* 是点和U的结合体，所有修改  新增 删除的信息都会提交到暂存区:
```python
$ git add -A
```
#### 查看当前文件的状态
```python
$git status
```
* 红色 ：在工作区中，还没有提交到暂存区。
* 绿色： 在暂存区中，还没有提交到历史区。
#### 把暂存区内容提交到历史区
```python
$ git commit 
```
* 最好在每回提交的时候，把提交记录名写的清晰一些：
```python
$ git commit -m "提交的记录名字"
```
* 提交暂存区和提交到历史区的步骤合在一起完成
```python
$ git commit -a -m "提交的记录名字"   
```
#### 查看提交历史记录
* 不管是从工作区提交到暂存区，还是从暂存区提交到历史区，每一个区域当前的内容都是一直保存的，不会消失。
* 只能查看当前回退版本以前的版本
```python
$ git log
```
* 都是查看历史提交记录（也相当于查看历史版本号），在没有历史版本回滚的时候，我们用哪个都可以，有历史版本回滚
```python
$ git reflog
```
* 我们发现以上的命令查看日志不太美观，下面这个命令可以可以让我们的日志变的更加清晰、美观。
```python
$ git lg
```
* 什么？输入上去，什么都没有，别着急同学们。把这个命令复制到你的git 终端里面。
```python
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"  
```
#### 暂存区所有内容撤回工作区
* 可以把`.`替换为具体的文件名。（不管暂存区的内容是否已经提交到历史版本上了）
```python
$ git rm --cached . r
```
* 把暂存区内容撤回工作区(覆盖现工作区中的内容，无法找回)
```python
$git chechout .  
```
* .在暂存区中，回滚到上一次暂存区中记录的内容(暂存区先回滚一次)
```python
$git reset HEAD .
```
* 指定回退到哪次提交的版本中(历史区的提交记录) "xxx" 是提交的历史记录。当我们回滚到某一个历史版本之后，暂存区和工作去的内容都将被这个版本内容所代替。
```python
$git reset --hard "xxx"  
```
#### 查看各个区域的代码变动
* 查看不同区域之间代码的不同，我们一般都是基于可视化的页面来查看不一样的
* 工作区vs暂存区：
```python
$ git diff
```
* 工作区vs历史区: 
```python
$ git diff master
```
* 暂存区vs历史区: 
```python
$ git diff --cached
```
#### 将工作区的内容进行保存
* 将本地的工作区的内容先给它暂时的保存起来
```python
$ git  stash
```
* 将保存起来的代码，进行还原
```python
$ git stash apply
```

#### 本地仓库和远程仓库保持关联
* 远程仓库地址  "xxx"  给远程仓库起的名字
```python
$ git remote add "xxx"
```
* 移除本地仓库跟远程仓库之间的关联  "xxx" 远程仓库的名字
```python
$ git remote rm "xxx"
```
* 查看当前仓库和哪些远程仓库保持关联
```python
$ git remote -v 
```
#### 让本地历史区信息和远程仓库信息保持同步
* 把本地代码推送到远程仓库中  `origin`代表的是远程仓库的名字  `master`代表的是远程仓库的主分支名字
```python
$ git push origin master
```
* 强制推送代码到远程上(一般建议大家在工作中不要使用这个命令） 
```python
$ git push origin HEAD:master
```
* 把远程仓库中的代码下载到本地中
```python
$ git pull origin master
```
* 如果拉我本地没有的分支：不要使用~~``git pull``~~ 命令  `origin `代表的是远程仓库的名字 `master` 就是远程分支的名字
```python
$ git fetch origin master-auth:master-auth
```
* 如果 `pull`时候觉得麻烦可以 进行克隆。 `url` 在远程仓库中的代码仓库的地址。
```python
$ git clone  "url"
```
#### 操作分支的基础命令
* 查看现在所有的分支
```python
$ git branch
```
* 创建一个新的分支（当切换到某个分支上的时候，会把当前master分支中新信息同步到这个新分支上）
```python
$ git branch "xxx"
```
* 切换到某个分支上
```python
$ git chechout "xxx"
```
* 创建一个新分支并切换到这个分支上
```python
$ git chechout -b "xxx"
```
* 再本地创建一个新的分支，并拉取远程分支到本地新创建的分支上。
```python
$ git checkout -b  "xxx"[本地新创建分支名字] origin[远程仓库名]/"xxx"[远程仓库的分支]
```
* 删除某个分支（一定要切换到其他分支上才可以删除当前分支）
```python
$ git branch -D "xxx"
```
* 合并分支内容
```python
$ git merge "xxx"
```
* 在有分支的情况下，可以更清楚看分支提交和合并内容
```python
$ git log --graph / --oneline
```