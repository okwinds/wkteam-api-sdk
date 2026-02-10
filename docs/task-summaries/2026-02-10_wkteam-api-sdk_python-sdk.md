# Task Summary — Python SDK（Python 3.11）交付

## 1) Goal / Scope

- Goal：在同一仓库中交付 `wkteam api-wen-dang2` 的 **Python 3.11 SDK**，并满足“接口全覆盖 + 离线可回归”。
- In Scope：
  - 新增 `src/python-sdk/` 子项目（打包配置 + README）
  - 以 `docs/specs/engineering-spec/02_Technical_Design/schemas/api_catalog.json` 为输入 codegen：
    - 生成 `wkteam_api_sdk/manifest.py`（operationId/endpointDefs）
    - 生成 `wkteam_api_sdk/generated_api.py`（`client.api.*` 全量 wrapper）
  - 实现 sync/async core client（统一错误模型、分类重试、hook、脱敏、可注入 transport）
  - pytest 离线回归（AT-008 覆盖性 gate + method/path contract smoke + retry 行为）
- Out of Scope：
  - 未做真实环境集成测试（缺少可用 baseUrl/Authorization）
  - 未对每个 endpoint 的返回结构做强类型建模（离线文档信息不完整，避免伪精确）
- Constraints：
  - 遵守 `AGENTS.md`：Spec-Driven + TDD + worklog/summary + 索引更新
  - 代码集中在一个源代码文件夹：`src/python-sdk/wkteam_api_sdk/`

---

## 2) Context（背景与触发）

- 背景：仓库已交付 TypeScript SDK；用户要求“再按规格文档交付 Python 3.11 SDK”，并可离线开发使用。
- 触发问题（Symptoms）：无 Python SDK、无 Python 覆盖性护栏、无可复现的 Python codegen。
- 影响范围（Impact）：Python 侧无法直接集成 wkteam API；无法复用离线文档成果进行本地开发。

---

## 3) Spec / Contract（文档契约）

- Contract：
  - 权威输入：`docs/api/api-wen-dang2/` 与 `docs/specs/engineering-spec/02_Technical_Design/schemas/api_catalog.json`
  - 覆盖性门槛（AT-008）：catalog 的 `operationId` 集合 == Python SDK manifest 的 `operation_ids` 集合
- Acceptance Criteria：
  - 生成并可调用 127 个 endpoint wrapper（不漏）
  - core client 支持：鉴权注入、超时、分类重试、统一错误、hook、脱敏
- Test Plan：
  - pytest 离线回归（fake transport）：覆盖性 gate + 全量 endpoint smoke + retry 行为
- 风险与降级：
  - 未接入真实环境：不对 baseUrl/Authorization scheme 做猜测；全部以配置项暴露

---

## 4) Implementation（实现说明）

### 4.1 Key Decisions（关键决策与 trade-offs）

- Decision：使用 `httpx` 并提供 sync/async 双 client（ADR-006）。
  - Why：同一套实现覆盖 sync/async；transport 便于注入与 mock；符合“工程可用”。
  - Trade-off：依赖略重；需要固定版本与说明。
  - Alternatives：requests（仅 sync）、aiohttp（仅 async）。
- Decision：codegen wrapper 的 method 命名默认基于 `path`，冲突时回退到（去模块前缀的）`operationId`。
  - Why：提高可读性，同时保证同模块方法名唯一，避免覆盖。
  - Trade-off：部分方法名较长，但可追溯、稳定。

### 4.2 Code Changes（按文件列）

- `docs/specs/engineering-spec/02_Technical_Design/PYTHON_SDK_ARCHITECTURE.md`：Python SDK 架构与覆盖性门槛。
- `src/python-sdk/tools/generate_sdk.py`：Python codegen（生成 wrappers/manifest/operation_index）。
- `src/python-sdk/wkteam_api_sdk/core_client.py`：核心请求管线（sync/async）。
- `src/python-sdk/wkteam_api_sdk/errors.py`：统一错误模型与可序列化上下文。
- `src/python-sdk/wkteam_api_sdk/generated_api.py`、`src/python-sdk/wkteam_api_sdk/manifest.py`、`src/python-sdk/wkteam_api_sdk/operation_index.py`：生成产物。
- `src/python-sdk/tests/`：pytest 离线回归（覆盖性 gate + 全量 smoke + retry 行为）。

---

## 5) Verification（验证与测试结果）

### Unit / Offline Regression（必须）

- 命令：
  - `python3.11 src/python-sdk/tools/generate_sdk.py`
  - `PYTHONPATH=src/python-sdk python3.11 -m pytest -q src/python-sdk/tests`
- 结果：
  - `7 passed`

### Integration（可选）

- 开关（env）：未实现（缺少真实环境参数）
- 结果：跳过

### Scenario / Regression Guards（强烈建议）

- 新增护栏：
  - AT-008 覆盖性 gate（不漏接口）
  - methodName collision 防回归（通过全量 smoke 覆盖）

---

## 6) Results（交付结果）

- 交付物列表：
  - Python SDK：`src/python-sdk/wkteam_api_sdk/`
  - 生成器：`src/python-sdk/tools/generate_sdk.py`
  - 离线回归：`src/python-sdk/tests/`
- 如何使用/如何验收：
  - 按 `src/python-sdk/README.md` 初始化并创建 client
  - 运行 `python3.11 src/python-sdk/tools/generate_sdk.py` 生成/更新 wrappers
  - 运行 `PYTHONPATH=src/python-sdk python3.11 -m pytest -q src/python-sdk/tests` 验证离线回归

---

## 7) Known Issues / Follow-ups

- 已知问题：
  - 本次未做真实环境集成 smoke（缺 baseUrl/Authorization）。
  - Python 侧强类型建模未覆盖返回结构（避免伪精确），后续可按真实返回逐步增强。
- 后续建议：
  - 增加 `WKTEAM_INTEGRATION=1` 的 Python 集成 smoke（只读接口优先）。
  - 若发现某些接口成功判定并非 `code==1000`，需要在 spec/catalog 增加 endpoint 级覆盖并在 core 中支持。

---

## 8) Doc Index Update

- 已在 `DOCS_INDEX.md` 登记：是
