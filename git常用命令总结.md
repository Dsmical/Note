## git常用命令总结

1. **配置 Git**
   - `git config --global user.name "你的名字"`：设置用户名。
   - `git config --global user.email "你的邮箱"`：设置邮箱地址。
2. **创建仓库**
   - `git init`：在当前目录初始化一个新的 Git 仓库。
3. **克隆仓库**
   - `git clone [URL]`：克隆（下载）一个远程仓库到本地。
4. **查看状态**
   - `git status`：查看当前仓库的状态，包括更改、未跟踪的文件等。
5. **添加和提交更改**
   - `git add [文件名]`：将指定文件添加到暂存区。
   - `git add .`：将所有更改的文件添加到暂存区。
   - `git commit -m "提交信息"`：提交暂存区的更改到仓库。
6. **推送更改**
   - `git push origin [分支名]`：将本地的更改推送到远程仓库的指定分支。
7. **拉取更新**
   - `git pull`：从远程仓库获取最新版本并合并到本地。
8. **分支管理**
   - `git branch`：查看所有分支。
   - `git branch [分支名]`：创建一个新分支。
   - `git checkout [分支名]`：切换到指定分支。
   - `git merge [分支名]`：合并指定分支到当前分支。
9. **查看历史**
   - `git log`：查看提交历史。
10. **撤销更改**
    - `git checkout -- [文件名]`：撤销对文件的修改。
    - `git reset --hard [commit哈希]`：重置到指定的提交。
11. **处理远程仓库**
    - `git remote add origin [URL]`：添加一个新的远程仓库。
    - `git remote -v`：查看所有远程仓库。

## 流程

**首次使用git的操作配置 Git**

- `git config --global user.name "你的名字"`：设置用户名。
- `git config --global user.email "你的邮箱"`：设置邮箱地址。

1. `git init`
2. `git add [文件名]`或者`git add .`
3. `git commit -m "提交信息"`
4. `git remote add origin [你的远程仓库URL]`
5. `git push -u origin main` 这里的 `master` 或 `main` 是你的远程仓库的主分支名称,`-u` 参数（或 --set-upstream）的作用是将本地的 main 分支与远程仓库的 main 分支关联起来。这意味着，在这次操作之后，你可以只用 git push 或 git pull 命令在这两个分支之间同步更改.

## 常见错误解决

1. **合并不相关的历史**:
   - 运行命令 `git pull origin main --allow-unrelated-histories`。
2. **创建并切换到 `main` 分支可以使用命令**
   + `git checkout -b main`
3. 待完善。。