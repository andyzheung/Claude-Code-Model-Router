# 新模型支持方案分析：GLM-5 和 MiniMax M2.5

## 1. 官方文档关键信息汇总

### 1.1 GLM-5 (智谱 AI)

**官方文档:** https://docs.bigmodel.cn/cn/coding-plan/tool/claude

| 参数 | 值 | 来源 |
|------|-----|------|
| model_id | `glm-5` | 文档明确说明："手动修改模型为 'glm-5'" |
| base_url | `https://open.bigmodel.cn/api/anthropic` | 保持不变 |
| API Key 环境变量 | `GLM_API_KEY` | 保持不变 |
| 认证头 | `x-api-key` | 保持不变 |

**文档引用:**
> "套餐已支持的用户使用 GLM-5，需要在自定义配置（如 Claude Code 中的 `~/.claude/settings.json`）中，手动修改模型为 'glm-5'。"

**其他可用模型:**
- `glm-4.5-air` (Haiku 对应)
- `glm-4.7` (Sonnet/Opus 对应)

---

### 1.2 MiniMax M2.5

**官方文档:** https://platform.minimaxi.com/docs/coding-plan/claude-code

| 参数 | 值 | 来源 |
|------|-----|------|
| model_id | `MiniMax-M2.5` | 文档配置示例 |
| base_url | `https://api.minimaxi.com/anthropic` | 文档配置示例 |
| API Key 环境变量 | `MINIMAX_API_KEY` | 保持不变 |
| 认证头 | `x-api-key` (通过 `ANTHROPIC_AUTH_TOKEN`) | 文档配置 |
| timeout | `3000000` ms (50分钟) | 文档配置 |

**文档配置示例:**
```json
{
  "ANTHROPIC_BASE_URL": "https://api.minimaxi.com/anthropic",
  "ANTHROPIC_AUTH_TOKEN": "MINIMAX_API_KEY",
  "API_TIMEOUT_MS": "3000000",
  "ANTHROPIC_MODEL": "MiniMax-M2.5"
}
```

**注意:** 官方文档中的 base_url 带尾部斜杠 `/anthropic/`，但项目中当前使用的是 `/anthropic`（不带尾部斜杠），建议保持当前项目的格式统一。

---

## 2. 需要修改的代码

### 2.1 更新 src/config.ts

```typescript
// 位置: 第 41-52 行

minimax: {
  display_name: 'MiniMax M2.5',     // M2.1 → M2.5
  provider: 'minimax',
  model_id: 'MiniMax-M2.5',         // MiniMax-M2.1 → MiniMax-M2.5
  base_url: 'https://api.minimaxi.com/anthropic',
  api_key_env: 'MINIMAX_API_KEY',
  auth_header: 'x-api-key',
  supports_streaming: true,
  supports_tools: true,
  max_tokens: 128000,              // 待官方确认具体值
  context_window: 200000,          // 待官方确认具体值
},
```

```typescript
// 位置: 第 65-76 行

glm: {
  display_name: 'GLM 5.0',         // 4.7 → 5.0
  provider: 'zhipu',
  model_id: 'glm-5',               // GLM-4.7 → glm-5 (注意大小写)
  base_url: 'https://open.bigmodel.cn/api/anthropic',
  api_key_env: 'GLM_API_KEY',
  auth_header: 'x-api-key',
  supports_streaming: true,
  supports_tools: true,
  max_tokens: 128000,              // 待官方确认具体值
  context_window: 200000,          // 待官方确认具体值
},
```

### 2.2 添加版本别名 (src/config.ts)

```typescript
// 位置: 第 78-95 行

aliases: {
  // ... 现有别名 ...

  // MiniMax 新增
  'minimax-m2.5': 'minimax',       // 新增
  'minimax-m2.1': 'minimax',       // 保留向后兼容

  // GLM 新增
  'glm-5': 'glm',                  // 新增
  'glm-5.0': 'glm',                // 新增
  'glm-4.7': 'glm',                // 保留向后兼容
},
```

### 2.3 更新 generateConfigFile() 函数

```typescript
// 位置: 第 234-242 行

minimax:
  display_name: "MiniMax M2.5"     # 更新
  model_id: MiniMax-M2.5           # 更新
```

```typescript
// 位置: 第 254-262 行

glm:
  display_name: "GLM 5.0"          # 更新
  model_id: glm-5                  # 更新 (注意小写)
```

### 2.4 更新 README.md

```markdown
| `minimax` | `minimax-m2.5`, `minimax-m2.1`, `mm` | MiniMax M2.5 | MiniMax |
| `glm` | `glm-5`, `glm-4.7`, `zhipu` | GLM 5.0 | 智谱 AI |
```

### 2.5 更新 package.json 版本

```json
{
  "version": "1.2.0"
}
```

---

## 3. 修改清单汇总

| 文件 | 修改内容 | 行数 |
|------|----------|------|
| `src/config.ts` | 更新 minimax model_id 为 `MiniMax-M2.5` | ~44 |
| `src/config.ts` | 更新 glm model_id 为 `glm-5` | ~68 |
| `src/config.ts` | 添加 `minimax-m2.5` 和 `glm-5` 别名 | ~78-95 |
| `src/config.ts` | 更新 generateConfigFile() 模板 | ~234-262 |
| `README.md` | 更新模型列表表格 | ~127-134 |
| `package.json` | 更新版本号至 1.2.0 | ~3 |

---

## 4. 待确认事项

虽然官方文档已提供主要信息，但仍需确认：

### 4.1 技术参数
| 参数 | GLM-5 | MiniMax M2.5 |
|------|-------|--------------|
| max_tokens | ? | ? |
| context_window | ? | ? |

> 当前暂使用 128000 / 200000，待官方文档更新后调整

### 4.2 兼容性测试
- [ ] GLM-5 API 是否完全兼容 Anthropic Messages API
- [ ] MiniMax M2.5 streaming 是否正常
- [ ] Tool calling 是否支持

---

## 5. 实施建议

由于官方文档已明确 model_id 和 base_url，可以立即进行以下修改：

**Phase 1: 立即修改（低风险）**
1. 更新 model_id
2. 添加版本别名
3. 更新文档

**Phase 2: 测试验证**
1. 本地构建测试
2. API 调用测试
3. 向后兼容性测试

**Phase 3: 发布**
1. 更新 CHANGELOG
2. 发布 npm v1.2.0

---

## 6. 关键注意事项

### 6.1 GLM model_id 格式
- 官方文档使用 `glm-5`（全小写）
- 项目当前使用 `GLM-4.7`（大写）
- **建议按官方文档修改为 `glm-5`**

### 6.2 MiniMax base_url
- 官方文档: `https://api.minimaxi.com/anthropic/` (带斜杠)
- 项目当前: `https://api.minimaxi.com/anthropic` (不带斜杠)
- **建议保持当前格式**（router.ts 会自动处理尾部斜杠）

---

## 7. 参考链接

- GLM 官方文档: https://docs.bigmodel.cn/cn/coding-plan/tool/claude
- MiniMax 官方文档: https://platform.minimaxi.com/docs/coding-plan/claude-code
- 项目地址: https://github.com/luwill/Claude-Code-Model-Router
