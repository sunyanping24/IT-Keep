<!-- TOC -->

- [提交/更新代码常用命令](#提交更新代码常用命令)
- [合并代码](#合并代码)
  - [1、git merge](#1git-merge)
  - [2、git checkout](#2git-checkout)
- [分支常用操作](#分支常用操作)
- [标签常用操作](#标签常用操作)
- [git stash](#git-stash)
- [git版本处理](#git版本处理)
- [git配置](#git配置)
- [git远程库常用操作](#git远程库常用操作)
- [gitignore使用常用](#gitignore使用常用)
- [git使用遇到的问题](#git使用遇到的问题)
  - [git无法pull仓库refusing to merge unrelated histories](#git无法pull仓库refusing-to-merge-unrelated-histories)

<!-- /TOC -->

[点此可查看阮一峰作者整理的常用命令](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)

# 提交/更新代码常用命令

- 在同一仓库操作

```
git status //查看当前分支、查看当前改动状态
git add . // 将改动代码添加到暂存区
git commit -m 'message' // 将暂存区的代码添加到本地仓库
git pull origin dev // 更新远程仓库的代码到本地仓库，自动进行合并，若有冲突手动解决
git push origin dev // 将本地仓库的代码推送到远程仓库
```
- fork出来的仓库

fork出来的仓库作为自己的开发仓库，代码开发完成，请求合并到主分支上。

```
git clone <remoteUrl>	// clone fork出来的仓库到本地
git remote add <originName> <原仓库的地址> // 添加原仓库的地址为本地源
git pull <originName> <branchName>	// 拉取/同步源仓库的代码
git push <originName> <branchName>  // 推送代码贷源仓库对应的分支 
```


# 合并代码

## 1、git merge

 一般情况下合并代码从版本低的分支往版本高的分支合并出现冲突的几率会比较小
```
git merge preBranch // 合并preBranch到当前分支
git branch --merged	 // 查看所有已经合并的分支
git branch --no-merged  // 查看还未合并的分支
git reset --merge 	// 取消正在合并
git merge <branchName> --allow-unrelated-histories     // 允许合并一个和当前分支没有关系的分支到当前分支
```
*Q1: Could Not Merge 3.4.0: You have not concluded your merge (MERGE_HEAD exists).*

上述错误会出现在合并分支中。若分支正在合并，再次进行合并时会出现上述错误，使用`git reset --merge` 可以取消合并。
但是需要注意的是，取消合并之后相当于前一次的合并就失效了，需要重新来进行合并操作。

*Q2: fatal: refusing to merge unrelated histories*

有时在使用`git pull`、`git push`、`git merge`时会出现上述描述的错误，这是因为拉取、提交、合并代码时，两个分支之间没有任何联我们可以强制允许两个没有任何相关历史的分支进行合并。
在使用命令时加上参数`--allow-unrelated-histories`，允许没有历史关系的分支进行操作处理。  
比如：`git merge <branchName> --allow-unrelated-histories`

**保持本地代码和远程仓库同步（使用远程代码强制覆盖本地代码）**

```
git reset --hard origin/master    // 使用远程master分支代码覆盖本地分支代码
```

## 2、git checkout 

> `git merge` 常常用来合并整个分支上的改变，有时需要合并分支上的某个文件时可以使用`git checkout`

```
git checkout <source_branch> <file-path>    // 合并目标分支上的文件到当前分支上
```

# 分支常用操作
```
git branch // 查看当前分支
git branch sss // 本地仓库新建分支sss
git checkout sss  // 切换当前分支到sss分支
git merge sss // 合并sss分支代码到当前所在分支
git branch --merged // 查看已合并分支
git branch --no-merged  // 查看未合并分支
git branch -d sss // 删除sss分支
git branch -a	//可以查看远程分支（显示红色的为远程分支）
git push origin <branchname>	//将本地分支推送到远程分支
git push --delete origin <branchname>	//删除远程分支
git branch -m <oldBN> <newBN>	//重命名本地分支
要想重命名远程分支，按照上面的删除远程分支->重命名本地分支->推送本地分支到远程分支
git push -u origin/remote_branch	//远程已有分支，但未和本地分支有联系，使用此命令
git push origin local_branch:remote_branch	//远程没有此分支，将本地分支新增为远程分支
git fetch -p    // 保持本地仓库和远程仓库同步（会将本地仓库对应的远程仓库没有的分支删除）
git push origin :refs/heads/<branchName>	// 当分支和标签名称相同时删除分支的
git reset --hard <anotherBranch>   // 强制将本地另一个分支覆盖到本地当前分支
git push origin <localBranch>:<remoteBranch> -f     // 强制将本地分支代码覆盖到远程指定分支
```

# 标签常用操作

标签的使用场景一般是：在某个分支，针对某个节点需要记录的，可以打标签记录。该标签会指定到当前的提交的对象引用。标签包含轻量级和含附注。
含附注标签：是仓库中一个独立的对象。在本地切换到标签和切换到分支的命令是相同的
轻量标签：是一个指向提交对象的引用
```
git tag -a <tagName>   // 创建当前所在分支的tag
git tag -a <tagName> -m '备注'  // 创建当前所在分支包含附注的tag
git tag     // 查看当前所有的tag
git show <tagName>    // 查看tag信息
git push origin <tagName>   // 分享标签（将标签推送到远程分支）
git push origin --tags  // 推送所有的tag到远程
git tag -d <tagName>    // 删除本地仓库标签
git push origin :refs/tags/<tagName>	// 删除远程仓库标签
git push --delete tag <tagName>     // 删除远程仓库标签（只适合于删除轻量级标签）
git fetch origin tag <tagName>  // 获取远程tag
git tag <newTagName> <oldTagName>   // 修改tag名称 修改完tag名称后可将本地和远程仍然存在的旧的tag删除，将新名称的tag推动到远程
```

# git stash

> 通常情况下，我们会遇到这种情况。功能做了一半有人过来反馈一个bug，让马上解决，但是新功能又做了一半又不想提交，
这时就可以使用`git stash`命令把当前的进度保存起来， 然后切换到另一个分支去修改bug，修改完提交后，再切回dev分支，
使用`git stash pop`来恢复之前保存的进度继续开发新的功能。

```
git stash   // 保存当前进度，会把暂存区和工作区的改动保存起来，当前的工作区是一个干净的环境
git stash save 'message'  // 保存进度时可以增加一些注释
git stash list    //  显示保存进度的列表，git stash 可以执行多次
git stash pop   // 恢复最新的进度到工作区(默认会把工作区和暂存区的改动都恢复到工作区)
git stash pop --index   // 恢复最新的进度到工作区和暂存区（尝试将原来暂存区的改动恢复到暂存区）
git stash pop <stash_id>    // 恢复指定的进度到工作区。（可以使用git list查询stash_id）
git stash pop   // 删除保存的最新的一个进度
git stash drop <stash_id>   // 删除指定的存储的进度
git stash clear   // 删除所有的进度
git stash apply // 恢复最新的进度到工作区(不会删除stash list)
                //git stash apply 的相关操作和git stash pop的相关操作是类似的，唯一的区别就是git stash pop时会将stash list删除
```

# git版本处理

```
git reset HEAD <filename>   // 从暂存区移除文件
git reset --hard HEAD~1     // GIT版本回退，~后的数字代表向后回退几个版本
git reset --hard HEAD <commitID>    // git版本回退都指定的版本

上述几种回退方式，处理完成之后直接使用`git push -f`来进行远程仓库的版本回退。这样历史commit记录也同样会消失
```

# git配置

`git config`共分为3个级别：`--system`-系统级别、`--global`-系统用户级别、`--local`-仓库级别，优先级`local>qlobal>system`
```
git config --global user.name "sunyanping"		//设置提交代码用户名
git config --global user.email sunyanping24@sina.com	//设置提交代码用户邮箱
git config --system http.sslverify false    // 当git clone时经常遇到输入用户名和密码的窗口不弹出，可以使用此命令设置即可
git config http.postBuffer 524288000    //设置http请求的缓存区的大小
git config [--system|--global|--local] -l   // 查看git的系统级别/用户级别/仓库级别配置，查看仓库级别配置时需要在对应的仓库目录
```

# git远程库常用操作

```
git remote -v   // 查看对应项目的远程克隆地址
git remote set-url origin [url]     // 更换git对应项目的远程克隆地址
```


# gitignore使用常用
- 创建.gitignore文件
```
需要在工作区中创建一个.gitignore文件，此文件中编写要忽略的文件，然后将此文件一并提交
touch .gitignore //在工作区创建git提交时忽略的文件文件
git check-ignore -v app.class	// 查看app.class文件在.gitignore文件中哪个位置被控制忽略
git add -f app.class	强制提交要忽略的文件
```

在win环境下创建.gitignore时，由于win默认文件是以`.`后缀结尾，后缀前面需要键入文件名，所以创建时可以先创建为`.gitignore.`，
然后保存时提示"修该文件扩展名可能导致文件不可用"时继续保存即可。

- 修改.gitignore文件并使其生效

```
git rm -r --cached .  // 清除当前根目录下所有的缓存
git add .   // 重新trace文件
git commit -m 'message'
git push origin master  // 提交
```

# git使用遇到的问题
## git无法pull仓库refusing to merge unrelated histories
出现这个问题的原因是：git pull是发现本地的commit历史和远程仓库的git commit历史不一样，git自动会认为可能是本地配置仓库地址不正确，如果自己确认是没有问题的，那可以使用命令`git pull --allow-unrelated-histories`来解决这个问题。

