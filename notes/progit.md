# ProGit

### [<主页](https://www.wangdekui.com/)

## SSH

ssh-keygen -t rsa -C "wonderkuin@gmail.com"  

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
git config user.email "wonderkuin@all.com"  
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
```

### git commit
会打开默认编辑器
git commit -v 将diff输出放入编辑器  
git commit -m 常用 一行信息提交  
git commit -a 跳过暂存步骤，直接提交全部 不建议  

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

## [<主页](https://www.wangdekui.com/)

```
git reset --hard HEAD
git reset --hard HEAD^

git reflog

git diff HEAD -- notegit.txt
git diff HEAD

#git restore --staged readme.txt
#git restore readme.txt

git rm readme.txt
```