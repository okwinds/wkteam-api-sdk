# Testing Strategy

目标：确保 SDK “可用”、可回归、可在无外网/无真实账号的情况下完成最小离线回归；在具备环境时可执行在线集成测试。

本仓库包含两套 SDK：
- TypeScript：`src/typescript-sdk/test/`（Vitest）
- Python：`src/python-sdk/tests/`（pytest）

## 分层

### Unit（离线回归，必须）

覆盖：
- URL/path 拼装、Query/Form/JSON 编码
- Header（包含 `Authorization`）注入逻辑与 token 生命周期状态机
- 请求/响应的结构化解析（`code/message/data` 等）
- 错误分类（网络错误、HTTP 非 2xx、业务 `code != 1000`、解析失败）
- 重试/超时/幂等等策略（不实际发请求，使用 mock transport）

要求：
- 不依赖外网
- 不依赖真实账号、真实 token
- 可在 CI 本地稳定运行

### Integration（可选，环境具备时运行）

覆盖：
- 真实 HTTP 连接到目标 baseUrl（可配置）
- 登录流程（获取 token）
- 选取少量无副作用/可控副作用的接口做冒烟（smoke）

要求：
- 使用 `.env`（不提交）/ `.env.example`（提交）管理配置
- 严格避免泄露 token/个人数据到日志

### E2E / Scenario（建议）

覆盖关键业务链路：
- 登录 → 初始化通讯录 → 发消息 → 收消息（回调） → 撤回消息
- 群相关操作（创建/邀请/踢人等）以最小可控集为主

## 质量门禁（Definition of Done for SDK Implementation）

- 所有 endpoint wrapper 都必须有最少 1 条单测覆盖（结构/参数/错误处理）
- 对“高风险”接口（发送消息、群管理）必须有场景回归用例
- 集成测试在无环境时允许跳过，但必须有显式 skip 机制与说明
