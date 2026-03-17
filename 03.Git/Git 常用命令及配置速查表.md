# Git 常用命令及配置速查表

## 1. 全局配置 (Configuration)

```bash
# 设置用户信息
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 查看配置
git config --list
git config user.name          # 查看特定项

# 设置默认分支名称 (可选，推荐 main)
git config --global init.defaultBranch main

# 设置行尾符 (Windows 推荐 auto, Linux/Mac 推荐 input)
git config --global core.autocrlf true
```

## 2. 初始化与克隆 (Init & Clone)

```bash
git init                      # 初始化本地仓库
git clone <url>               # 克隆远程仓库
git clone <url> <folder_name> # 克隆到指定文件夹
```

## 3. 基本工作流 (Basic Workflow)

```bash
git status                    # 查看文件状态
git add <file>                # 暂存指定文件
git add .                     # 暂存所有变更 (包括新文件)
git commit -m "Message"       # 提交并添加注释
git commit --amend            # 修改上一次提交 (不推送到远程)
```

## 4. 分支管理 (Branching)

```bash
git branch                    # 列出本地分支
git branch -a                 # 列出所有分支 (含远程)
git branch <name>             # 创建新分支
git checkout <name>           # 切换分支
git checkout -b <name>        # 创建并切换到新分支
git merge <name>              # 合并指定分支到当前分支
git branch -d <name>          # 删除本地分支 (安全删除)
git branch -D <name>          # 强制删除本地分支
```

## 5. 远程同步 (Remote & Sync)

```bash
git remote -v                 # 查看远程仓库地址
git remote add origin <url>   # 添加远程仓库
git fetch origin              # 下载远程更新 (不合并)
git pull origin <branch>      # 下载并合并远程更新
git push origin <branch>      # 推送本地分支到远程
git push -u origin <branch>   # 首次推送并建立关联
git push --force              # 强制推送 (慎用，会覆盖远程历史)
```

## 6. 撤销与重置 (Undo & Reset)

```bash
git restore <file>            # 撤销工作区修改 (未 add)
git restore --staged <file>   # 撤销暂存区修改 (已 add 未 commit)
git reset --hard HEAD         # 彻底重置工作区和暂存区到最新提交 (危险!)
git reset --soft HEAD~1       # 撤销上一次 commit，保留更改在暂存区
git reset --mixed HEAD~1      # 撤销上一次 commit，保留更改在工作区 (默认)
git revert <commit_id>        # 生成一个新提交来撤销某次提交 (安全，推荐用于公共分支)
```

## 7. 查看历史与差异 (Log & Diff)

```bash
git log                       # 查看提交历史
git log --oneline             # 单行显示历史
git log --graph --oneline     # 图形化显示历史
git diff                      # 查看工作区与暂存区的差异
git diff HEAD                 # 查看工作区与最新提交的差异
git show <commit_id>          # 查看某次提交的详情
```

## 8. 临时保存 (Stash)

```bash
git stash                     # 暂存当前工作现场
git stash list                # 查看暂存列表
git stash pop                 # 恢复最近一次暂存并删除记录
git stash apply               # 恢复最近一次暂存但保留记录
git stash drop                # 删除最近一次暂存
git stash clear               # 清空所有暂存
```

## 9. 实用别名 (Alias) - 可选

*添加到 `~/.gitconfig` 的 `[alias]` 部分以简化命令*

```ini
[alias]
    co = checkout
    ci = commit
    st = status
    br = branch
    hist = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short
    last = log -1 HEAD
```

*使用示例：`git st` 等同于 `git status`*
