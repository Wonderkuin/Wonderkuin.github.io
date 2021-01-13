# ProGit

### [<主页](https://www.wangdekui.com/)

## SSH

ssh-keygen -t rsa -C "wonderkuin@gmail.com"

git remote set-url origin git@github.com:Wonderkuin/Wonderkuin.github.io.git

---

## Config
优先级 每个级别覆盖上一级别  
1. /etc/gitconfig
2. ~/gitconfig
3. .git/config

### 浏览配置
```
git config --list
git config user.name
```

### 全局配置
```
git config --global user.name "wonderkuin"  
git config --global user.email "wonderkuin@all.com"  
```

### 可以针对特定项目使用不同的信息
```
git config user.name "wonderkuin"  
git config user.email "wonderkuin@gmail.com"  
```

### 文本编辑器
```
git config --global core.editor emacs
```

### 别名
```
git config --global alias.st status
# 定义外部命令
git config --global alias.visual '!gitk'
```

### 颜色
```
git config --global color.ui true
```

---

## Help
```
git help config
git config --help
man git-config
```

---

## 获取Git仓库

### 空仓库
```
git init
touch note
git add .
git commit -m "add note"

git remote add origin https://gitee.com/wangdekui/learngit.git
git remote -v

git pull git@gitee.com:wangdekui/learngit.git master
git push -u origin master

git push --set-upstream origin master
git push origin master
```

### Clone
```
git clone https://github.com/aseprite/aseprite.git

git clone https://github.com/aseprite/aseprite.git myAseprite

git clone --recurse-submodules

git clone --recursive https://github.com/aseprite/aseprite.git
cd aseprite
git pull
git submodule update --init --recursive
```

---

## 忽略文件
```
$cat .gitignore
#可以用标准的glob模式匹配
#简化的正规表达式 * [abc] [0-9] a/**/z任意中间目录 可以匹配 a/b/z a/b/c/z
#匹配模式可以以/开头防止递归
#匹配模式可以以/结尾指定目录
#!取反
*.txt
!Liscense.txt
/TODO
build/
doc/*.txt
doc/**/*.pdf
```

git add -f Nex.io # 强制添加忽略文件  
git chckout-ignore -v Nex.io # 查看违反了哪条忽略规则  

---

## 文件操作

```
touch NewFile
git status
git add NewFile
git reset HEAD NewFile
git add *
git commit -m "add New File"
echo "test" > NewFile
git checkout -- NewFile
```

### git add 添加内容到下一次提交中
1. 开始追踪新文件
2. 把已跟踪的文件放入暂存区
3. 在合并时把有冲突的文件标记为已解决状态

### git status
```
#git status --short
$git status -s
 M NewFile1
MM NewFile2
A  NewFile3
M  NewFile4
?? NewFile5
```
1. ?? 新添加未追踪  
2. A 新添加到暂存区  
3. 右M 修改了未放入暂存区  
4. 左M 修改了并放入暂存区  

### git diff
```
#查看暂存的内容  
git diff --cached  
git diff --staged  #相同，但是好记  

git diff -- notegit.txt
git diff HEAD
```

### git commit
会打开默认编辑器
git commit -v 将diff输出放入编辑器  
git commit -m 常用 一行信息提交  
git commit -a 跳过暂存步骤，直接提交全部  

git commit --amend 将落下的add后的文件提交到上次commit

### git rm
移除文件 去掉追踪
git rm -f  如果已经放入暂存区，可以强制删除 无法被git恢复  
git rm --cached log/\*.log 反斜杠 防止shell帮忙展开匹配  
git rm \*~ 删除所有~结尾的文件

### git mv
改名 相当于  
mv NewFile.md NewFile
git rm NewFild.md
git add NewFild

### git log
```
git log -p -2
git log --stat
git log --pretty=oneline

git log --pretty=format:"%h - %an, %ar : %s"
git log --pretty=format:"%h %s" --graph

git log --author
git log --grep
git log --author wang --grep skill --all-match
# --all-match 意味着&&，不然就是||

git log -Sfunction_name
# 用于查询具体操作

git log --pretty="%h - %s" --author=wang --since="2020-08-03" \
--before="2020-08-05" --no-merges -- t/

# 查看分支
git log --oneline --decorate --graph --all
```

---

## 远程仓库

```
git remote -v
git remote show origin

git remote add gitee https://github.com/aseprite/aseprite.git

git fetch gitee

git remote rename origin github
git remote rm gitee

git push gitee master
```

## 标签

```
git tag

git tag -l v1.0.0

#创建附注标签annotated
git tag -a v1.2 -m "version 1.2"

#创建轻量标签lightweight
git tag v1.2-tc

git show v1.2
git show v1.2-tc

#对之前的提交加标签
git tag -a 1.2 80808abcdd

#push并不会将标签发送到服务器，需要显式推送
git push origin v1.2
git push origin --tags #全部
```

---

# 分支

## 本地
```
git branch
git branch issss
git checkout -b hotfix
git checkout master
git merge hotfix
git branch -d hotfix
git branch -v #查看每个分支最新提交
git branch --merged #查看已经合并到当前分支的分支
git branch --no-merged #查看没合并到当前分支的分支
```

## 远程
```
# 查看远程分支信息
git ls-remote

# 不适用默认名origin初始化本地分支
git clone -o boo

# 拉取origin分支
git fetch origin

# 每次输入密码
git config --global credential.helper cache
```

### 跟踪
```
git checkout --track origin/fox

# 根据远程分支建立本地分支，使用不同名字
git checkout -b foxex origin/fox

# 设置已有本地分支跟踪远程分支
git branch -u origin/fox

#设置好跟踪分支后，可通过
#@{upstream} 或 @{u} 引用被跟踪的分支
#git merge @{u} 等价于 git merge origin/master

# 查看跟踪
git fetch --all
git branch -vv

```

### 推送
```
# 将本地的fox分支推送到origin的fox
git push origin fox
git push origin fox:fox #与上面相同 别人拉取到origin/fox 不是一个新的分支，是不可修改的指针
git merge origin/fox # 合并拉取的心分支到自己的分支
git push origin fox:master #将本地的fox分支推送到远程的master分支
```

### 删除远程分支
```
git push origin --delete fox
```

## 变基 rebase
```
git checkout exper
git rebase master
git checkout master
git merge exper
git branch -d exper


# master 分出 server ， server分出 client
# 将client变基到master 后面
git rebase --onto master server client
git checkout master
git merge client
git rebase master server # 不必切换到server再执行rebase
git checkout master
git merge server
git branch -d server
git branch -d client
```

### 用变基解决变基
```
不要对在你的仓库外有副本的分支执行变基。
如果你遵循这条金科玉律，就不会出差错。否则，人民群众会仇恨你，你的朋友和家人也会嘲笑你，唾弃你

git pull --rebase

git fetch
git rebase some/master

```

## 服务器上的Git 略

## 分布式Git 到这里暂停，目前已经够用了

## TODO
```
git reset --hard HEAD
git reset --hard HEAD^

git reflog

git switch -c dev  
git switch master



git stash  
git stash list  
git stash apply  
git stash drop  
git stash pop  
git stash apply stash@{0}  
git cherry-pick 4c805e2  
git checkout -b dev origin/dev  
git switch -c dev origin/dev  
git branch --set-upstream-to=dev origin/dev  
```

### 在缓慢的网络中
```
#第一种，不好
mkdir proj_dir
cd proj_dir
git clone -b master https://gitee.com/lemongame/slgrpg_2d.git --depth=1
git pull

#第二种，改进
mkdir proj_dir
cd proj_dir
git init
git fetch --depth=1 https://gitee.com/lemongame/slgrpg_2d.git master:refs/remotes/origin/master
git checkout master

#再改进
mkdir proj_dir
cd proj_dir
git init
git remote add origin https://gitee.com/lemongame/slgrpg_2d.git
git pull origin master --depth=1
```

## [<主页](https://www.wangdekui.com/)