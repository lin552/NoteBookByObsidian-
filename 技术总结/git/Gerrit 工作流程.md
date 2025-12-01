---
创建时间: "2025-11-12 11:03:40"
作者: wangxiaoming
tags:
---


#### 一、多人开发流程
```bash
# 1.开始工作前先同步
git fetch origin
git rebase origin/6.0.0_xpdev

# 2.进行开发修改

# 3.提交更改
git add .
git commit -m "你的提交信息"

# 4.推送前再次同步（避免期间有人提交）
git fetch origin
git rebase origin/6.0.0_xpdev

# 5.解决可能出现的冲突
git rebase --continue 
git rebase --abort

# 查看操作记录，找到Rebase前的提交哈希
git reflog

# 重置到Rebase前的状态
git reset --hard HEAD@{1}

# 6.推送到Gerrit
git push origin HEAD:refs/for/6.0.0_xpdev
```
#### 二、可配置内容
- 设置git配置自动变基：
```bash
# 设置pull时自动使用rebase
git config --global pull.rebase true

# 或者为特定分支设置
git config branch.6.0.0_xpdev.rebase true
```
- 使用便携命令：
```bash
# 创建别名，一键同步和rebase
git config --global alias.sync '!git fetch origin && git rebase origin/6.0.0_xpdev'

# 之后只需运行
git sync
```
#### 三、情况处理
- rebase时出现冲突
```bash
# 查看冲突文件
git status

#解决冲突后
git add .
git rebase --continue

# 如果冲突太复杂想放弃
git rebase --abort

```