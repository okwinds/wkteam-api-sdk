# 任务总结：Repo 卫生与目录收敛（移除构建产物/依赖缓存）

## Goal / Scope

用户反馈“目录结构管理差、`dist` 是什么、Node 资源不应在根目录、不同技术栈源码应各自收敛”，本次对仓库做一次可复现的目录卫生收敛：

- SDK 源码保持在 `src/` 下，并按技术栈分目录：`src/python-sdk/`、`src/typescript-sdk/`
- 清理不应长期驻留的本地构建产物/依赖缓存：`node_modules/`、`dist/`、pytest cache
- 统一入口文档对“生成器/测试位置”的表述，避免误导为根目录存在 `tools/`、`test/`

## Key Decisions

- `node_modules/` 与 `dist/` 不作为仓库交付物：通过 `npm i` / `npm run build` 可重建。
- 目录约定收敛：主工程规格在 `docs/specs/engineering-spec/`，离线权威输入在 `docs/api/`。

## Code / Docs Changes

- 清理目录：
  - `src/typescript-sdk/node_modules/`
  - `src/typescript-sdk/dist/`
  - `src/python-sdk/.pytest_cache/`
  - Python `__pycache__/`
- 文档更新：
  - `README.md`：修正“生成器/测试所在位置”的表述，并解释 `node_modules/`、`dist/` 的性质。
  - `docs/worklog.md`：追加本次变更记录（命令 + 结果）。

## Test Plan & Results（离线回归）

本次变更以“目录卫生/文档修正”为主，但仍需按 DoD 复跑离线回归：

- TypeScript SDK：
  - `cd src/typescript-sdk && npm i`
  - `cd src/typescript-sdk && npm run generate:api`
  - `cd src/typescript-sdk && npm run generate:sdk`
  - `cd src/typescript-sdk && npm test`
- Python SDK：
  - `python3.11 src/python-sdk/tools/generate_sdk.py`
  - `PYTHONPATH=src/python-sdk python3.11 -m pytest -q src/python-sdk/tests`

**Results**
- TypeScript：`14 passed`，并且 `npm run build` 成功生成 `src/typescript-sdk/dist/**`（构建产物，可重建，不入库）。
- Python：`7 passed`

说明：为保持仓库目录干净，本次验证完成后已删除本地 `src/typescript-sdk/node_modules/` 与 `src/typescript-sdk/dist/`，需要时按命令重建即可。

## Known Issues / Risks

- 若你的本地脚本/IDE 仍引用旧路径 `wkteam-api-wen-dang2-sdk/`，需要手工切换到 `wkteam-api-sdk/`（本次已删除旧目录名的兼容别名，以避免歧义）。

## Next Steps

- 若需要进一步“仓库入口更易用”，可在 `README.md` 增加一段最小 Repo Map（docs/spec/src）与常用命令矩阵（TS/Python）。
