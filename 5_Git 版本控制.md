# Git 版本控制

任务目标：

1. 前言
2. 理解版本控制的意义
3. 掌握三⼤区域：⼯作区、暂存区、仓库
4. 熟练使⽤以下命令：
   - 克隆与同步： git clone , git pull , git push
   - ⽇常操作： git add , git commit , git status
   - 历史查看： git log , git diff
5. 分支管理
   - 创建/切换/合并： git branch , git checkout , git merge
   - 解决合并冲突
6. 理解 git fetch 与 git pull 的区别；
7. 掌握远程仓库管理： git remote
8. 掌握撤销操作：
   - git reset , git revert , git stash
9. 了解 Rebasing ⼯作流
10. 熟悉 Pull Request (PR) 提交流程
11. 能恢复误删的提交（如通过 reflog ）
12. 小结
13. 鸣谢

### 一、前言

说实话笔者突然接到这样一个大任务是有一点懵的，因为光是条目就有11条之多，而且不同于先前的Linux学习，这个学习中笔者并没有在 Windows 中配置类似东西的经验，所以笔者预料到应该学习过程会比较艰难……

### 二、理解版本控制

关于这个问题，一头雾水的笔者去询问了deepseek，得到了如下答复：

> - **历史记录**：保存每次代码变更，可以随时回退到任意版本
> - **团队协作**：多人同时开发，自动合并代码变更
> - **分支管理**：在不影响主线的同时开发新功能
> - **责任追踪**：谁、什么时候、改了什么地方、为什么改
>
> **类比理解**：就像写论文时的"最终版.docx"、"最终版2.docx"、"真的最终版.docx"，但 Git 帮你智能管理所有这些版本。

### 三、掌握三⼤区域：⼯作区、暂存区、仓库

通过查阅资料笔者了解到，工作区是指正在进行的部分，暂存区是指正在准备上传的文件区域，仓库则是放置最终稿的地方。

### 四、学习一些命令

1. 克隆与同步

   ``` shell
   # 从远程仓库克隆到本地
   git clone https://github.com/username/repo.git
   
   # 拉取远程最新变更并合并到当前分支
   git pull origin main
   
   # 将本地提交推送到远程仓库
   git push origin main
   ```

2. 日常操作

   ``` shell
   # 查看当前状态（最常用！）
   git status
   
   # 添加文件到暂存区
   git add filename.txt          # 添加单个文件
   git add .                     # 添加所有修改的文件
   git add *.js                  # 添加所有js文件
   
   # 提交到仓库
   git commit -m "描述这次提交做了什么"
   ```

3. 历史查看

   ``` shell
   # 查看提交历史
   git log                      # 详细历史
   git log --oneline            # 简洁历史
   git log --graph              # 图形化显示分支
   
   # 查看文件差异
   git diff                     # 工作区与暂存区的差异
   git diff --staged            # 暂存区与最后一次提交的差异
   git diff commit1 commit2     # 两个提交之间的差异
   ```

这些操作为笔者打开了新世界的大门，因为在配置git的时候笔者已经绑定了GitHub账号，所以这样笔者便可以在终端里面操作GitHub上面的内容 。~~（完美解决了笔者看不懂英文的问题）~~ 

### 五、分支操作

以笔者之见，分支操作就像我们在Windows里面操作文件夹里面的文件一样，可以让我们的git上面的文件变得条理清晰。

``` shell
#基础分支操作：
# 查看分支
git branch                   # 查看本地分支
git branch -a                # 查看所有分支（包括远程）

# 创建分支
git branch new-feature       # 创建新分支
git checkout new-feature     # 切换到新分支
# 或者合并成一条命令：
git checkout -b new-feature  # 创建并切换到新分支

# 现代 Git 推荐使用：
git switch new-feature       # 切换到已有分支
git switch -c new-feature    # 创建并切换到新分支

########################################################
#合并分支
# 先切换到要合并到的目标分支
git checkout main

# 合并特性分支
git merge new-feature

# 合并后删除已合并的分支
git branch -d new-feature

########################################################
```

此外，解决合并冲突也是一个重要的环节。

笔者查阅 [资料](https://zhuanlan.zhihu.com/p/21943908560) 得知：

> 合并冲突是指在执行`git merge`时，Git无法自动合并两个分支的更改。通常，冲突发生在同一文件的相同部分被两个分支修改，或者两个分支对同一行代码进行不同的修改。Git无法判断哪个版本更好，因此需要开发者介入解决冲突。

通常情况下解决合并冲突要按照下面的顺序：

1. 检测合并冲突
2. 查看冲突位置
3. 手动解决冲突
4. 标记此处冲突已经解决
5. 执行合并

``` shell
# 1. 发生冲突后，Git 会提示
git merge feature-branch

# 2. 查看冲突文件
git status

# 3. 手动编辑文件，选择要保留的代码，删除冲突标记
# 4. 标记冲突已解决
git add resolved-file.txt

# 5. 完成合并
git commit
```

### 六、git   fetch   vs   git   pull

笔者经过了解后得知，git fetch 只负责下载远程文件但是不会进行合并，但是 git  pull 会合并 。

### 七、**远程仓库管理**

``` shell
# 查看远程仓库
git remote -v

# 添加远程仓库
git remote add origin https://github.com/user/repo.git

# 修改远程仓库URL
git remote set-url origin new-url
```

### 八、**撤销操作**

人们常说“人生不买后悔药” ，但是我们的git有撤销操作来充当“后悔药”的作用。

``` shell
# 重置暂存区（取消git add）
git reset filename.txt

# 重置到某个提交（谨慎使用！）
git reset --hard commit_hash   # 彻底回退
git reset --soft commit_hash   # 保留修改到暂存区

# 创建新的提交来撤销之前的提交（安全）
git revert commit_hash

# 临时保存未提交的修改
git stash                      # 保存当前修改
git stash list                 # 查看保存的修改
git stash pop                  # 恢复最近保存的修改
```

### 九、了解 Rebasing ⼯作流

经过了解，笔者认为Rebasing ⼯作流是另一种合并的方法，它与merge的区别在于，merge是创建、合并，而它则是线性操作，将历史更改，要根据实际需求情况来研究。

``` shell
# 将当前分支变基到main分支
git checkout feature-branch
git rebase main

# 交互式变基（合并、修改提交信息等）
git rebase -i HEAD~3          # 修改最近3个提交
```

### 十、熟悉 Pull Request (PR) 提交流程

这是 GitHub/GitLab 等的协作流程，显然我们要用GitHub与他人协作的话是要熟悉这个流程的。

1. Fork 仓库 → 在网页点击 Fork

2. 克隆到本地：

   ```shell
   git clone https://github.com/yourname/repo.git
   ```

3. 创建特性分支：

   ```shell
   git checkout -b my-feature
   ```

4. 开发并提交：

   ```shell
   git add .
   git commit -m "Add new feature"
   git push origin my-feature
   ```

5. 创建 PR → 在 GitHub 页面操作

6. 代码审查 → 等待维护者审查

7. 合并 → 维护者合并你的 PR

### 十一、能恢复误删的提交（如通过 reflog ）

另一种“后悔药”：

```shell
# 查看所有操作历史（包括"丢失"的提交）
git reflog

# 找到要恢复的提交哈希，然后：
git checkout commit_hash      # 临时查看
git branch recovered-branch commit_hash  # 创建新分支指向该提交
```

实际恢复场景：

```shell
# 不小心删除了分支
git branch -d important-branch  # 误删

# 恢复步骤：
git reflog
# 找到删除前的提交哈希，比如 abc123
git branch important-branch abc123  # 恢复分支
```

### 十二、小结

对git的学习完成之后，笔者意识到并没有笔者所设想的那么困难，完全可以把git当作另一种编程语言来处理。

### 十三、鸣谢

~~(tips:笔者其实并不知道应不应该有这部分，但是笔者决定加上)~~

在此次学习过程中笔者主要要感谢群中学长拓展了笔者的视野。

其次要感谢deepseek以及网上各路大神的帖子帮助了我的学习。

# THE END

