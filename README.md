# sync-fork

自动同步上游仓库的 tags 和分支到你的 fork。

## 前置条件

需要一个 GitHub Personal Access Token (PAT)，权限勾选 `repo` + `workflow`。

创建后存入 fork 仓库的 Secrets，变量名 `SYNC_FORK_TOKEN`。

Settings → Secrets and variables → Actions → New repository secret

## 使用

在你的 fork 仓库创建 `.github/workflows/sync-fork.yml`：

```yaml
name: Sync Fork from Upstream
on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

# 避免定时任务与手动触发并发执行导致互相覆盖
concurrency:
  group: sync-fork-${{ github.ref }}
  cancel-in-progress: false

jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      actions: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: SourceDive/sync-fork@v1.1
        with:
          upstream_repo: spring-projects/spring-ai
          github_token: ${{ secrets.SYNC_FORK_TOKEN }}
```

> 启用分支同步时，请务必保留 `fetch-depth: 0`，否则 `merge`/`rebase` 无法获得完整历史。

## 参数

| 参数 | 必填 | 默认值 | 说明 |
|---|---|---|---|
| `upstream_repo` | 是 | — | 上游仓库，格式 `owner/repo` |
| `github_token` | 是 | — | PAT，需 `repo` + `workflow` 权限 |
| `sync_tags` | 否 | `true` | 是否同步 tags（增量推送，仅推送 fork 上缺失的 tag）|
| `tag_force` | 否 | `false` | 为 `true` 时额外推送 SHA 不一致的同名 tag（强制覆盖）|
| `sync_branches` | 否 | `false` | 是否同步分支 |
| `target_branch` | 否 | （空）| 要同步的分支，多个用逗号分隔；留空则自动探测上游默认分支 |
| `branch_sync_mode` | 否 | `merge` | 分支同步方式：`merge` / `rebase` / `force` |
| `git_user_name` | 否 | `github-actions[bot]` | `merge`/`rebase` 生成提交所用用户名 |
| `git_user_email` | 否 | `...github-actions[bot]...` | `merge`/`rebase` 生成提交所用邮箱 |

## Tag 同步说明

tag 采用**增量推送**：先用 `git ls-remote` 取 fork 上已有的 tag，只推送本地（含上游）存在而 fork 缺失的 tag，稳定态下不再每次全量推送。开启 `tag_force` 时，会额外推送同名但 SHA 不同的 tag（强制覆盖）。

## 分支同步说明

- `merge`（默认）/ `rebase`：会把上游合并进 fork 的同名分支，**保留 fork 自己的提交**；若产生冲突会让任务失败而不是静默覆盖。
- `force`：用上游分支强制覆盖 fork 同名分支，会**丢弃 fork 上的自有提交**（适合纯镜像场景）。
