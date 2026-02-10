# Task Summary — Repo Layout Refactor（多 SDK 收敛）

## 1) Goal / Scope

- Goal：将仓库调整为多技术栈 SDK 共存，但源代码不散落，且 Node/测试/工具不放在仓库根目录。
- In Scope：
  - Python SDK 目录规范化为 `src/python-sdk/`
  - TypeScript SDK 目录规范化为 `src/typescript-sdk/`，并将 Node 资源（package/node_modules/tools/test/config）收敛进去
  - 修复 codegen 与测试对 repo root 的定位（避免 `process.cwd()` 假设）
  - 更新文档索引与工程规格中的路径引用
- Out of Scope：
  - 不修改 API contract、endpoint 清单与业务逻辑
  - 不做新的线上集成测试
- Constraints：遵守 `AGENTS.md`（Spec-Driven + TDD gate + worklog + index）。

---

## 2) Context（背景与触发）

- 背景：仓库已交付 TS/Python SDK，但目录层级与资源分布不一致。
- 触发问题：用户要求“源码集中在 `src/`，不同技术栈各自目录，同级清晰；根目录不放 node 资源；测试/工具随 SDK 一起归类；文档索引需准确”。

---

## 3) Spec / Contract（文档契约）

- Contract：`docs/specs/engineering-spec/00_Overview/REPO_LAYOUT.md`
- Acceptance Criteria：见 `docs/specs/engineering-spec/00_Overview/REPO_LAYOUT.md`
- Test Plan：离线回归（pytest + vitest）

---

## 4) Implementation（实现说明）

### 4.1 Key Decisions

- Decision：把 Node 工具脚本放在 `src/typescript-sdk/tools/`，并让脚本通过“向上查找 anchor 文件”定位 repo root。
  - Why：避免依赖执行目录（`process.cwd()`）造成路径漂移，满足目录收敛后仍可一键生成与回归。

### 4.2 Code Changes（按文件列）

- `docs/specs/engineering-spec/00_Overview/REPO_LAYOUT.md`：结构重构规格。
- `src/typescript-sdk/tools/generate_api_artifacts.mjs`：改为可从 `src/typescript-sdk/` 正确定位 repo root。
- `src/typescript-sdk/tools/generate_sdk.mjs`：改为可从 `src/typescript-sdk/` 正确定位 repo root，并输出到 `src/typescript-sdk/src/generated/`。
- `src/typescript-sdk/test/api-coverage.test.ts`：改为 anchor 定位 repo root（不依赖 cwd）。
- `DOCS_INDEX.md`、`README.md`：更新路径与使用命令。

---

## 5) Verification（验证与测试结果）

### Unit / Offline Regression（必须）

- Python：
  - `python3.11 src/python-sdk/tools/generate_sdk.py`
  - `PYTHONPATH=src/python-sdk python3.11 -m pytest -q src/python-sdk/tests`
  - 结果：`7 passed`
- TypeScript：
  - `cd src/typescript-sdk && npm test`
  - 结果：`14 passed`

---

## 6) Results（交付结果）

- 交付物列表：
  - `src/typescript-sdk/`：TypeScript SDK（含 tools/test/config）
  - `src/python-sdk/`：Python SDK（含 tools/tests）
  - `docs/specs/engineering-spec/00_Overview/REPO_LAYOUT.md`：结构规范
- 如何使用/验收：
  - TS：`cd src/typescript-sdk && npm run generate:api && npm run generate:sdk && npm test`
  - Python：`python3.11 src/python-sdk/tools/generate_sdk.py && PYTHONPATH=src/python-sdk python3.11 -m pytest -q src/python-sdk/tests`

---

## 7) Known Issues / Follow-ups

- 已知问题：`src/typescript-sdk/node_modules/` 体积较大，建议在实际 git 仓库中不提交，并通过 `npm ci` 复现安装。
- 后续建议：若要进一步“彻底 monorepo 化”，可引入 workspace（npm workspaces/pnpm），但需在 spec 中明确并同步文档。

---

## 8) Doc Index Update

- 已在 `DOCS_INDEX.md` 登记：是
