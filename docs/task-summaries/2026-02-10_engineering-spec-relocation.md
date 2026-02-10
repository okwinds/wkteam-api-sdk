# 任务总结：工程规格目录迁移（`engineering-spec/` → `docs/specs/engineering-spec/`）

## Goal / Scope

依据当前仓库 `AGENTS.md` 的目录约定（Spec/设计文档目录为 `docs/specs/`），将仓库根目录的 `engineering-spec/` 迁移到规范位置：

- 迁移目标：`docs/specs/engineering-spec/`
- 级联更新所有引用（文档、工具脚本、测试锚点、README/DOCS_INDEX）
- 保持“可复现 + 离线回归门槛（TDD Gate）”不回归

## Key Decisions

- 工程规格作为 **docs/specs 体系的一部分**：保留目录名 `engineering-spec`，以便识别“主工程规格”，但不再占用仓库根目录。
- 工具与测试统一锚定新的 catalog 路径：`docs/specs/engineering-spec/02_Technical_Design/schemas/api_catalog.json`。
- 本地构建/依赖产物（`node_modules/`、`dist/`）仍不作为仓库存档内容：验证完成后删除，需要时可命令重建。

## Changes

### Directory Move

- `engineering-spec/` → `docs/specs/engineering-spec/`

### Cascading Updates

- 根目录入口与索引：
  - `README.md`
  - `DOCS_INDEX.md`
  - `docs/worklog.md`
- TypeScript SDK：
  - `src/typescript-sdk/tools/generate_api_artifacts.mjs`（输出路径调整到 `docs/specs/engineering-spec/**`）
  - `src/typescript-sdk/tools/generate_sdk.mjs`（读取 catalog 路径与 repoRoot anchor 更新）
  - `src/typescript-sdk/test/api-coverage.test.ts`（coverage gate 读取路径更新）
  - `src/typescript-sdk/tools/README.md`、`src/typescript-sdk/README.md`
- Python SDK：
  - `src/python-sdk/tools/generate_sdk.py`（repoRoot anchor 与读取路径更新）
  - `src/python-sdk/tests/test_api_coverage_gate.py`
  - `src/python-sdk/README.md`
- 工程规格正文中涉及路径引用的段落同步更新（避免“点击路径失效/误导”）。

## Test Plan & Results（离线回归）

### TypeScript SDK

```bash
cd src/typescript-sdk
npm i
npm run generate:api
npm run generate:sdk
npm test
npm run build
```

Results：
- `14 passed`
- `npm run build` 成功（产物 `src/typescript-sdk/dist/**` 可重建，不入库）

### Python SDK

```bash
python3.11 src/python-sdk/tools/generate_sdk.py
PYTHONPATH=src/python-sdk python3.11 -m pytest -q src/python-sdk/tests
```

Results：
- `7 passed`
- codegen 读取新 catalog 路径成功，生成 `manifest.py/generated_api.py/operation_index.py`（127 endpoints）

### Cleanup (keep repo clean)

```bash
rm -r src/typescript-sdk/node_modules src/typescript-sdk/dist
```

## Known Issues / Risks

- 若外部脚本仍硬编码旧路径 `engineering-spec/**`，需要按本总结更新为 `docs/specs/engineering-spec/**`。

## Next Steps

- 若希望进一步减少路径重复，可在工程规格中把“仓库根目录前缀”改成相对链接（例如 `./02_Technical_Design/...`），但需要一次性全量一致性调整。
