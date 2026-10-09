# container-actions 发版

- 合入 `master` 且改动 `.github/workflows/**` → `release-actions-bundle` 打 `actions/vX.Y.Z`
- 嵌套 `uses: workers-world/container-actions/…@actions/v*` 在打 tag 前对齐
- manifest 经带 `[skip actions-release]` 的 backfill PR 回填
- Caller **禁止** `@master`
