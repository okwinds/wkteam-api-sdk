# 任务总结：wkteam-api-sdk（全量 endpoint SDK 落地）

## Goal / Scope

目标：在离线 API 文档（`docs/api/`）与工程规格（`docs/specs/engineering-spec/`）约束下，交付一套可用的 TypeScript SDK，并确保接口不漏、可离线回归。

范围：
- ✅ SDK 核心能力：配置、鉴权注入、URL 拼接、GET query、POST JSON、超时、重试（默认保守）、并发限制、错误模型
- ✅ 全量 endpoint wrapper：基于离线文档解析出的 **127** 个 endpoint
- ✅ 覆盖性护栏：catalog ↔ SDK manifest 一致性校验 + method/path contract smoke
- ✅ 构建与质量：lint / unit tests / build 通过

非范围：
- ❌ 真实环境的集成验证（需要你提供 `baseUrl` 与 `Authorization`）
- ❌ 上游平台的权限模型/RBAC 推断（SDK 不做权限判断）

## Key Decisions（Trade-offs）

- 选择 **codegen**（编译期生成）而非纯手写封装：
  - 优点：127 endpoints 不漏、可维护、可自动回归
  - 代价：需要维护生成器与生成产物（但可复现）
- `baseUrl` / `Authorization` 不做猜测，全部外置为配置项：
  - 优点：适配你当前“未知真实 baseUrl/鉴权格式”的事实
  - 代价：需要你在真实环境自行提供并做集成 smoke
- 方法命名策略：优先使用 `path` 推导（如 `/member/login` → `memberLogin`），回退到 operationId：
  - 优点：调用体验更友好
  - 代价：若上游 path 变更，方法名会随之变化（但仍有 operationId 稳定索引可追溯）

相关 ADR：`docs/specs/engineering-spec/00_Overview/DECISION_LOG.md`

## Code Changes（概要）

- 工程化：
  - `package.json`：vitest/tsup/biome/zod（固定版本）
  - `tsconfig.json`、`tsup.config.ts`、`vitest.config.ts`、`biome.json`
- SDK 核心：
  - `src/sdk/core-client.ts`：统一请求管线（超时/重试/错误/并发/解析）
  - `src/sdk/client.ts`：挂载 `client.api.*`（由 codegen 生成）
  - `src/sdk/errors.ts`、`src/sdk/transport.ts`、`src/sdk/config.ts`
- Codegen：
  - `tools/generate_sdk.mjs`：从 `api_catalog.json` 生成 `src/generated/**`
  - `src/generated/**`：endpointDefs/validators/api tree/manifest/operationIndex
- 覆盖性与回归测试：
  - `test/api-coverage.test.ts`：catalog ↔ manifest 对齐、wrapper 存在性、method/path contract smoke（遍历 127 endpoint）
  - 其他测试：错误映射、鉴权头注入、重试、并发限制、GET query

## Test Plan & Results（离线回归证据）

执行命令：
- `node tools/generate_sdk.mjs`
- `npm test`
- `npm run lint`
- `npm run build`

结果：
- ✅ `npm test` 全绿（含 127 endpoint contract smoke）
- ✅ `npm run lint` 通过（Biome）
- ✅ `npm run build` 通过（ESM/CJS/types）

## Known Issues / Risks

- 需要真实环境闭环的点仍存在（但已全部外置配置并写入规格/文档）：
  - 真实 `baseUrl` 形态（是否有网关前缀 path）
  - `Authorization` scheme（是否需要前缀、失效表现、刷新方式）
  - 限流/风控/幂等机制（上游是否支持）
- `npm install` 输出了若干依赖审计告警（dev 依赖链路）。建议后续在你确定可升级策略后再做 `npm audit fix`（避免引入破坏性升级）。

## Next Steps

1) 你提供真实 `baseUrl` 与 `Authorization` 后，我可以补：
   - `test:integration`（仅挑选低副作用/只读 endpoint 做 smoke）
   - README 中的更贴近真实环境的示例与排错指引
2) 若你希望更强类型（响应 data 结构更准确）：
   - 扩展 `tools/generate_api_artifacts.mjs` 对“返回参数表/示例 JSON”的解析，生成 `data` 的更精确 schema（仍保持可回退 `unknown`）。
