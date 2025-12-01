---
创建时间: "2025-11-12 11:03:32"
作者: wangxiaoming
tags:
---
#### 一、git reset (重置)
##### 三种重置模式
###### --soft (软重置)
```bash
# 移动分支指针，保留工作区和暂存区的更改
git reset --soft HEAD~1
```
 - **只移动HEAD指针**，不修改工作目录和暂存区
- **用途**：重新提交之前的更改

###### --mixed （混合重置，默认）
```bash
# 移动分支指针，重置暂存区，但保留工作区更改
git reset HEAD~1
#等同于
git reset --mixed HEAD~1
```
- **最常用**：取消已暂存的文件
- **用途**：重新组织提交内容

###### --hard (硬重置)
```bash
# 彻底重置：移动指针、清空暂存区和工作区
git reset --hard HEAD~1
```
- **危险操作**：会永久丢失未提交的更改
- **用途**：彻底放弃最近的修改

#### 二、rebase （变基）
- 基本变基
```bash
# 将当前分支变基到目标分支
git rebase <目标分支>

# 示例：将feature分支变基到main
git checkout feature
git rebase main
```
交互式变基（整理提交）
```bash
# 整理最近3次提交
git rebase -i HEAD~3

# 常用操作：
# pick - 使用提交
# reword - 使用提交但修改提交信息
# edit - 使用提交但暂停修改
# squash - 将提交合并到前一个提交
# fixup - 类似squash但丢弃提交信息
# drop - 删除提交
```
##### rebase实用场景：
- 同步最新代码
```bash
git fetch origin
git rebase origin/main
```
- 整理凌乱的提交历史
```bash
# 合并多个小提交为一个有意义的提交
git rebase -i HEAD~5
# 然后选择squash或fixup
```
- 解决分支分叉
```bash
# 之前：main --- A --- B
#         \ 
#          feature --- C --- D
# 
git checkout feature
git rebase main
# 
# 之后：main --- A --- B --- C' --- D'
```
##### 重要安全规则
```bash
# 🚫 永远不要对已推送到共享仓库的分支执行 rebase 或 reset --hard
# ✅ 只对本地分支执行这些操作
```
#### 三、分支操作
##### 1）创建分支
```bash
# 创建本地分支
git branch <分支名>

#创建并切换到新分支
git checkout -b <分支名>

#创建并切换到新分支
git switch -c <分支名>

```
##### 2）切换分支
```bash
# 切换到已有分支
git checkout <分支名>

#切换到已有分支
git switch <分支名>

#切换到上一个分支
git checkout -

```
##### 3） 查看分支
```bash
# 列出本地分支
git branch 

# 列出分支及最后一次提交
git branch -v

#显示分支跟踪的远程分支
git branch -vv

#列出所有分支(含远程分支)
git branch -a 
```
##### 4）合并分支
```bash
#将源分支合并到当前分支
git merge <源分支>

#强制非快速合并（生成合并提交）
git merge --no--ff <源分支>

#冲突处理
# 1.编辑冲突文件
# 2.git add <文件>
# 3.git commit
```
##### 5）删除分支
```bash
# 删除已合并的本地分支
git branch -d <分支名>

#强制删除未合并的本地分支
git branch -D <分支名>

#删除远程分支
git push origin --delete <分支名>

```
##### 6）基础操作命令
```bash
# 初始化本地仓库
git init

# 克隆远程仓库到本地
git clone <远程仓库URL>

# 将文件加入暂存区
git add <文件>

# 提交暂存区到本地仓库
git commit -m "消息"

# 查看工作区和暂存区状态
git status

# 查看提交历史
git log

# 提交未暂存的修改
git diff

```
#### 四、远程协作命令
```bash
# 查看远程仓库地址
git remote -v

# 获取远程仓库更新（不合并）
git fetch <远程名>

# 拉取远程分支并合并(fetch + merge)
git pull <远程名> <分支名>

# 推送本地分支到远程
git push <远程名> <分支名>

# 推送并设置上游分支（后续可简写 git push）
git push -u <远程名> <分支名>
```
#### 五、辅助命令
```bash
# 暂存工作区修改（用于切换分支前保存现场）
git stash save "临时保存登录页修改"

# 恢复最近一次暂存的修改
git stash pop

# 创建标签（如版本号）
git tag <便签名>

# 回退到指定提交
git reset --hard <提交哈希>

# 创建撤销提交
git revert <提交哈希>

# Rollback某个文件（未提交时）
git restore <文件名>

# 回退文件到历史版本（已提交到仓库）
git restore --source=<提交哈希> <文件名>

# 从暂存区取消暂存（保留文件区文件）
git restore --staged <文件名>

# 从暂存区删除文件
git rm --cached logs/debug.log # 从暂存区删除 debug.log，保留工作区文件

# 删除暂存区文件并删除工作区文件（彻底删除）
git rm src/old-file.txt  # 删除暂存区和工作区的 old-file.txt


```