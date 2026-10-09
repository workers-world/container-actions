# 镜像仓 Container CI

## 分工

| 能力 | 仓 |
|------|-----|
| Release PR、verify（可关 test）、auto-merge、sync-default | [worker-actions](https://github.com/workers-world/worker-actions) |
| Docker smoke（PR）、GHCR build-push（master only） | **container-actions**（本仓） |
| 全量 Wrangler Container deploy | 访问层 Worker（cpt1 / dld1）自有 `deploy-container.yml` |

## 必须

1. **禁止**在 `push` → `dev_*` 上 push GHCR。
2. Caller pin `actions/vX.Y.Z`，禁止 `@master`。
3. Branch protection：Release PR 将 docker-smoke 设为 required check。

## 模板文件

- [templates/ci.yml](../templates/ci.yml)
- [templates/build-image.yml](../templates/build-image.yml)
- [templates/sync-default-branch.yml](../templates/sync-default-branch.yml)

复制到镜像仓 `.github/workflows/`，替换 `IMAGE` / `dispatch_*`。
