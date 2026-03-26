# Git 分支同步工作流

本仓库 origin 指向 Gitee 镜像仓库，Gitee 会自动同步 GitHub 上游更新。
个人定制修改保存在 `swordup_study` 分支，`main` 分支保持与上游一致。

## 初始设置（仅执行一次）

```bash
# 1. 基于 main 创建个人分支
git checkout -b swordup_study

# 2. 提交当前修改
git add -A
git commit -m "feat: 自定义修改 - pm2集成、暗色模式默认、移除ja语言"

# 3. 推送到 Gitee
git push -u origin swordup_study

# 4. 切回 main，丢弃修改，保持与上游一致
git checkout main
git checkout .
```

## 日常同步流程（每次上游更新后执行）

```bash
# 1. 拉取 Gitee 上已自动同步的最新 main
git checkout main
git pull origin main

# 2. 切到个人分支，rebase 到最新 main
git checkout swordup_study
git rebase main

# 3. 推送（rebase 改写了提交历史，需要强制推送覆盖远程旧历史）
git push --force-with-lease origin swordup_study
```

## rebase 后提示与远程已分叉（diverged）

执行 `git status` 若出现：

```text
Your branch and 'origin/swordup_study' have diverged,
and have N and M different commits each, respectively.
```

**原因**：rebase 会生成**新的 commit hash**，本地历史与远程上仍保留的、rebase **之前**的提交已经对不上号，Git 会认为两边各自多出一截提交，即「分叉」。

**处理**（仅适用于**个人分支** `swordup_study`，且确认没有别人刚往远程推了你必须保留的提交时）：

```bash
git push --force-with-lease origin swordup_study
```

`--force-with-lease` 比 `--force` 更安全：若远程在你上次 `fetch` 之后又有了新提交，会**拒绝推送**，避免误覆盖他人工作。

若被拒绝，先 `git fetch origin`，再查看 `git log origin/swordup_study` 与本地差异，决定是再次 rebase、合并，还是与协作者确认。

**不要**在这种情况用普通 `git pull` 了事：容易在已 rebase 的本地历史上再叠一层合并提交，历史会更乱；个人分支以「本地为准推上去」时，应用上述强制推送（并遵守下文注意事项）。

## 处理 rebase 冲突

如果 rebase 过程中遇到冲突：

```bash
# 1. 查看冲突文件
git status

# 2. 手动编辑冲突文件，解决冲突后标记为已解决
git add <冲突文件>

# 3. 继续 rebase
git rebase --continue

# 如果想放弃本次 rebase，回到 rebase 前的状态
git rebase --abort
```

## 注意事项

- `--force-with-lease` / `--force` 仅用于**个人分支**；rebase 会生成新的 commit hash，必须用强制推送才能让远程与本地一致
- **永远不要对 `main` 分支使用 `--force`**
- 日常开发请始终在 `swordup_study` 分支上操作
