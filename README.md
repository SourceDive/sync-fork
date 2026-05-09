# sync-fork

自动同步上游仓库的 tags 和分支到你的 fork。

## 使用

### 1. 在你的 fork 仓库创建 `.github/workflows/sync-fork.yml`

```yaml
name: Sync Fork from Upstream

on:
  schedule:
    - cron: '0 0 * * *'       # 每天 UTC 0:00
  workflow_dispatch:           # 手动触发

jobs:
  sync:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: SourceDive/sync-fork@v1
        with:
          upstream_repo: spring-projects/spring-ai
```

### 2. 参数

| 参数 | 必填 | 默认值 | 说明 |
|---|---|---|---|
| `upstream_repo` | 是 | — | 上游仓库，格式 `owner/repo` |
| `sync_tags` | 否 | `true` | 是否同步 tags |
| `sync_branches` | 否 | `false` | 是否同步分支 |
| `target_branch` | 否 | `main` | 要同步的分支，多个用逗号分隔 |
