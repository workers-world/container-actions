# workers-world/container-actions

镜像仓可复用的 GitHub Actions：**Release PR 上 docker smoke（不 push）**，**仅 master 推 GHCR**。

**不要** `@master`，pin `actions/vX.Y.Z`（见 [manifest/actions-bundle.yaml](manifest/actions-bundle.yaml)）。

Release PR / auto-merge / sync-default 仍用 [worker-actions](https://github.com/workers-world/worker-actions)；本仓只负责 Docker 轨。

## 轨

```text
push dev_* → worker-ci（开 Release PR）+（PR 上）container docker-smoke
         → 门禁绿 auto-merge → master
         → container-build-push → GHCR :latest
         → 可选 repository_dispatch → cpt1 / dld1 全量 deploy
```

## Caller（镜像仓）

见 [templates/](templates/) 与 [docs/container-ci.md](docs/container-ci.md)。

```yaml
# .github/workflows/ci.yml — 并列 job
jobs:
  release-pr:
    uses: workers-world/worker-actions/.github/workflows/worker-ci.yml@actions/v0.2.26
    # … secrets / with: materialize_sdk: false, run_tests: false, sync_packages_lock: false
  docker-smoke:
    permissions:
      contents: read
    uses: workers-world/container-actions/.github/workflows/container-ci.yml@actions/v0.1.0
    with:
      image: ghcr.io/workers-world/your_image

# .github/workflows/build-image.yml — 仅 master
on:
  push:
    branches: [master]
jobs:
  build-push:
    permissions:
      contents: read
      packages: write
    uses: workers-world/container-actions/.github/workflows/container-build-push.yml@actions/v0.1.0
    secrets:
      DISPATCH_TOKEN: ${{ secrets.GHA_TOKEN }}
    with:
      image: ghcr.io/workers-world/your_image
      emit_dispatch: true
      dispatch_type: your_image_pushed
      dispatch_repos: workers-world/your-access-layer
```

Branch protection：Release PR required checks 须含 **docker-smoke**（或 reusable 展开后的 job 名）。

## Workflows

| Workflow | 用途 |
|----------|------|
| `container-ci.yml` | 门面：PR 上调 smoke |
| `container-docker-smoke.yml` | `docker build` + load，不 push |
| `container-build-push.yml` | master 推 GHCR；可选 dispatch |
| `release-actions-bundle.yml` | 打 `actions/v*` |

## 调用方配置（名称，不含值）

| 名称 | 类型 | 用途 |
|------|------|------|
| `GHA_TOKEN` | Secret | Release PR / sync-default / 跨仓 `repository_dispatch`（`DISPATCH_TOKEN`） |
| `RELEASE_BOT_*` | 同 worker-actions | 本仓打 tag（含 workflows pin） |
| GHCR | `GITHUB_TOKEN` + `packages: write` | build-push 本仓镜像 |

## 首 tag

建仓后：合入 `master` → `release-actions-bundle` 打出 `actions/v0.1.0`（或 `workflow_dispatch` 指定 `0.1.0`）。在 tag 存在前，镜像仓 caller 的 `uses …@actions/v0.1.0` 会解析失败——先发本仓再迁 caller。
