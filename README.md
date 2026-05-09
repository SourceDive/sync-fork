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

## 参数

| 参数 | 必填 | 默认值 | 说明 |
|---|---|---|---|
| `upstream_repo` | 是 | — | 上游仓库，格式 `owner/repo` |
| `github_token` | 是 | — | PAT，需 `repo` + `workflow` 权限 |
| `sync_tags` | 否 | `true` | 是否同步 tags |
| `sync_branches` | 否 | `false` | 是否同步分支 |
| `target_branch` | 否 | `main` | 要同步的分支，多个用逗号分隔 |
