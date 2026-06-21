# CCMR-Plus 遗留事项跟踪（Pending Items）

> 记录 GLM-5.2 升级（v1.2.4）后**未完成、待跟进**的事项，便于持续跟踪。
>
> 最近更新：2026-06-21（thinking 模式已实现并端到端验证通过）

---

## ✅ 已完成：GLM-5.2 thinking 模式支持（extra_body 字段）

> 原核心遗留，已于 **2026-06-21 实现并端到端验证通过**。以下保留方案与验证记录供回溯。

### 实现内容

采用**通用 `extra_body` 字段**（保持「配置驱动」哲学，不硬编码智谱参数）：

| 文件 | 改动 |
|------|------|
| `src/types.ts` | `ModelConfig` 增加 `extra_body?: Record<string, unknown>` |
| `src/router.ts` `buildRequestBody` | 若 `modelConfig.extra_body` 存在，merge 进 body（仅注入客户端未显式指定的字段，客户端优先） |
| `src/config.ts` glm 配置 | 填 `extra_body: { thinking: { type: 'enabled' }, reasoning_effort: 'max' }`（默认配置 + `generateConfigFile` 模板同步） |
| `src/version.ts`（新增） | 抽取共享 VERSION 常量，`cli.ts` + `server.ts` 复用（顺带修复 server.ts 版本硬编码） |

注入逻辑（`buildRequestBody`，stream / 非 stream 两条路径自动覆盖）：

```ts
if (modelConfig.extra_body) {
  for (const [key, value] of Object.entries(modelConfig.extra_body)) {
    if (!(key in body)) body[key] = value;  // 客户端优先
  }
}
```

### 端点验证结果（2026-06-21，已通过）

智谱 **Anthropic 端点**（`/api/anthropic/v1/messages`）**接受** `thinking` / `reasoning_effort` 参数：

- 普通模式：HTTP 200，正常回复。
- thinking 模式：HTTP 200，思考过程以**标准 Anthropic `thinking` block**（带 `signature`）返回，Claude Code 可原生渲染。
- 未传 `temperature` 未报错（官方 OpenAI 端点示例要求 `temperature:1.0`，Anthropic 端点不强制）。

整链路（Claude Code → 网关 → 智谱）已通过网关 `/v1/messages` 实测确认：返回体含 `{"type":"thinking",...}` block。

### 关键决策

- **客户端优先**：注入逻辑只在客户端未传该字段时注入（已实现）。
- **temperature**：不强制覆盖（端点不强制；遵循客户端优先）。
- **默认开启**：GLM 默认即注入 thinking（已实测端点支持，安全）。想关闭：`models.yaml` 覆盖 `extra_body` 为空。

---

## 📋 其它遗留事项

| # | 事项 | 状态 | 说明 |
|---|------|------|------|
| 1 | `ref-doc/user-guide.md` 内容过期 | ✅ 已更新（2026-06-21） | 已同步到 GLM 5.2 |
| 2 | `src/server.ts` health 版本硬编码 | ✅ 已修复（2026-06-21） | 抽 `src/version.ts` 共享，`/health` 返回 1.2.4 |
| 3 | `npm pkg fix` | ⬜ 待办 | 修 `repository.url normalized` 发布警告（需 bump 版本后发布生效） |
| 4 | GitHub Release v1.2.5 | ⬜ 可选 | 打 tag + release notes |
| 5 | npm token / API Key 撤销 🔒 | ⬜ **高（安全）** | 发布用 token 曾在对话明文暴露（含一个 bypass 2FA），去 npm 网站 → Access Tokens 撤销。**本次验证用的 GLM_API_KEY 也在对话明文暴露，建议一并轮换** |
| 6 | `router.ts` 超 200 行 | ⬜ 暂缓 | 加 extra_body 注入后约 322 行 > CLAUDE.md 硬指标，建议后续拆分（当前暂缓） |

---

## 状态记录

- **2026-06-21**：v1.2.4 发布完成（GLM-5.2 普通模式）。
- **2026-06-21**：thinking 支持（extra_body 字段）**实现并端到端验证通过**；智谱 Anthropic 端点确认支持 thinking 参数。同步修复 server.ts 版本号硬编码、更新 user-guide.md。
- **2026-06-21**：**v1.2.5 已发布到 npm（dist-tag `latest`）**。含 thinking 默认开启、server.ts 版本修复、user-guide 同步、新增 thinking-mode-guide.md。
- **🔒 安全提醒**：本次验证使用的 GLM_API_KEY 已在对话明文暴露，建议轮换。

---

## 相关文档

- [GLM-5.2 使用指南](glm-5.2-usage-guide.md)
- [升级指南](upgrade-guide.md)
- [回退指南](rollback-guide.md)
