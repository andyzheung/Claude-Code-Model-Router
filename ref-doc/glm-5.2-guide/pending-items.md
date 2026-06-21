# CCMR-Plus 遗留事项跟踪（Pending Items）

> 记录 GLM-5.2 升级（v1.2.4）后**未完成、待跟进**的事项，便于持续跟踪。
>
> 最近更新：2026-06-21

---

## 🔥 核心遗留：GLM-5.2 thinking 模式支持（extra_body 字段）

### 现状

v1.2.4 已将 GLM 升级到 5.2，但 **以普通模式运行**，未注入 thinking 参数。GLM-5.2 的「深度思考」（`thinking` + `reasoning_effort`）卖点尚未启用。

### 目标

新增通用 `extra_body` 配置字段，让 GLM-5.2 在请求中注入 `thinking: {type: enabled}` + `reasoning_effort: max`，发挥 5.2 的思考能力。

### 前置条件（实施前必须先验证）

智谱 **Anthropic 端点**（`/api/anthropic`）是否接受 `thinking` / `reasoning_effort` 参数。

- 官方文档只在 **OpenAI 端点**（`/api/paas/v4/chat/completions`）演示了这俩参数
- Anthropic 端点是否支持**未经实测**

验证命令：

```bash
curl -X POST https://open.bigmodel.cn/api/anthropic/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $GLM_API_KEY" \
  -d '{"model":"glm-5.2","max_tokens":256,"thinking":{"type":"enabled"},"reasoning_effort":"max","messages":[{"role":"user","content":"你好"}]}'
```

- 200 + 正常回复 → 端点支持，可推进实施
- 报错 / 被忽略 → 端点不支持，方案搁置

### 设计方案（验证通过后实施）

采用**通用 `extra_body` 字段**（不硬编码智谱参数，保持「配置驱动」哲学）：

| 文件 | 改动 |
|------|------|
| `src/types.ts` | `ModelConfig` 增加 `extra_body?: Record<string, unknown>` |
| `src/router.ts` `buildRequestBody` | 若 `modelConfig.extra_body` 存在，merge 进 body（仅注入客户端未显式指定的字段） |
| `src/config.ts` glm 配置 | 填 `extra_body: { thinking: { type: 'enabled' }, reasoning_effort: 'max' }` |

router 注入逻辑示意：

```ts
const body = { ...request };
body.model = modelConfig.model_id;
// 注入模型专属参数（客户端未显式指定时才注入）
if (modelConfig.extra_body) {
  for (const [k, v] of Object.entries(modelConfig.extra_body)) {
    if (!(k in body)) body[k] = v;
  }
}
```

> 改动集中在一处（`buildRequestBody`），stream / 非 stream 两条路径自动覆盖。

### 待决策点

1. **temperature**：思考模式要求 `temperature: 1.0`，是否在 `extra_body` 强制覆盖（会覆盖 Claude Code 传入值）？
2. **客户端优先**：注入逻辑只在客户端未传该字段时注入（上方示意已如此）。
3. **router.ts 拆分**：当前 313 行，超 CLAUDE.md 的 200 行硬指标；加注入逻辑时建议顺手拆分（**当前用户决定暂缓**）。

### 验证方法

注入后：curl 打网关 + 在 Claude Code 实测，确认思考过程（`reasoning_content`）能正常返回并被渲染。

---

## 📋 其它遗留事项

| # | 事项 | 说明 | 优先级 |
|---|------|------|--------|
| 1 | `ref-doc/user-guide.md` 内容过期 | 仍写 GLM 5.0，未更新到 5.2（不影响 npm 包，仅仓库文档） | 低 |
| 2 | `src/server.ts` health 版本硬编码 | `/health` 返回 `version: '1.0.0'`，应改为读 VERSION 常量 | 低 |
| 3 | `npm pkg fix` | 修 `repository.url normalized` 发布警告（需 bump 版本后发布生效） | 低 |
| 4 | GitHub Release v1.2.4 | 打 tag + release notes（可选） | 可选 |
| 5 | npm token 撤销 🔒 | 发布用的两个 token 已在对话明文暴露（含一个 bypass 2FA），建议去 npm 网站 → Access Tokens 撤销 | **高（安全）** |
| 6 | `router.ts` 超 200 行 | 313 行 > CLAUDE.md 硬指标，建议拆分（当前决定暂缓） | 中 |

---

## 状态记录

- **2026-06-21**：v1.2.4 发布完成（GLM-5.2 普通模式）。thinking 支持（extra_body 字段）列为下一步，待 Anthropic 端点 thinking 参数实测通过后实施。

---

## 相关文档

- [GLM-5.2 使用指南](glm-5.2-usage-guide.md)
- [升级指南](upgrade-guide.md)
- [回退指南](rollback-guide.md)
