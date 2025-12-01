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

#### git  revert

#### git pull

#### git fetch