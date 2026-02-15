# 版本 v1.2.0 修改记录

## 修改概述

本次更新主要支持 **GLM 5.0** 和 **MiniMax M2.5** 两个新模型，同时保持向后兼容性。

---

## 修改文件清单

| 文件 | 修改类型 | 说明 |
|------|----------|------|
| `src/config.ts` | 修改 | 更新模型配置和别名 |
| `README.md` | 修改 | 更新模型列表和文档 |
| `package.json` | 修改 | 版本号升级至 1.2.0 |

---

## 详细修改内容

### 1. src/config.ts

#### 1.1 更新 MiniMax 模型配置 (第 41-52 行)

```diff
  minimax: {
-   display_name: 'MiniMax M2.1',
+   display_name: 'MiniMax M2.5',
    provider: 'minimax',
-   model_id: 'MiniMax-M2.1',
+   model_id: 'MiniMax-M2.5',
    base_url: 'https://api.minimaxi.com/anthropic',
    api_key_env: 'MINIMAX_API_KEY',
    auth_header: 'x-api-key',
    supports_streaming: true,
    supports_tools: true,
    max_tokens: 128000,
    context_window: 200000,
  },
```

#### 1.2 更新 GLM 模型配置 (第 65-76 行)

```diff
  glm: {
-   display_name: 'GLM 4.7',
+   display_name: 'GLM 5.0',
    provider: 'zhipu',
-   model_id: 'GLM-4.7',
+   model_id: 'glm-5',  // 注意：官方使用小写
    base_url: 'https://open.bigmodel.cn/api/anthropic',
    api_key_env: 'GLM_API_KEY',
    auth_header: 'x-api-key',
    supports_streaming: true,
    supports_tools: true,
    max_tokens: 128000,
    context_window: 200000,
  },
```

#### 1.3 添加 MiniMax 版本别名 (第 85-87 行)

```diff
    'minimax-m2': 'minimax',
    'minimax-m2.1': 'minimax',
+   'minimax-m2.5': 'minimax',
    'mm': 'minimax',
```

#### 1.4 添加 GLM 版本别名 (第 91-94 行)

```diff
    'glm-4.7': 'glm',
    'glm-4.6': 'glm',
+   'glm-5': 'glm',
+   'glm-5.0': 'glm',
    'zhipu': 'glm',
```

#### 1.5 更新 generateConfigFile() 模板

```diff
  minimax:
-   display_name: "MiniMax M2.1"
+   display_name: "MiniMax M2.5"
    provider: minimax
-   model_id: MiniMax-M2.1
+   model_id: MiniMax-M2.5
```

```diff
  glm:
-   display_name: "GLM 4.7"
+   display_name: "GLM 5.0"
    provider: zhipu
-   model_id: GLM-4.7
+   model_id: glm-5
```

```diff
 aliases:
   ds: deepseek
   deepseek-v3.2: deepseek
   mm: minimax
   minimax-m2.1: minimax
+  minimax-m2.5: minimax
   kimi-k2: kimi
   qwen3-max: qwen
   glm-4.7: glm
+  glm-5: glm
```

---

### 2. README.md

#### 2.1 更新模型列表表格 (第 127-134 行)

```diff
## 支持的模型

| 短名称 | 版本别名 | 模型 | 提供商 |
|--------|----------|------|--------|
| `deepseek` | `deepseek-v3.2`, `ds` | DeepSeek V3.2 | DeepSeek |
| `kimi` | `kimi-k2`, `kimi-k2-thinking` | Kimi K2 Thinking | Moonshot |
-| `minimax` | `minimax-m2.1`, `mm` | MiniMax M2.1 | MiniMax |
+| `minimax` | `minimax-m2.5`, `minimax-m2.1`, `mm` | MiniMax M2.5 | MiniMax |
| `qwen` | `qwen3-max`, `qwen3` | Qwen3 Max | 阿里云 |
-| `glm` | `glm-4.7`, `zhipu` | GLM 4.7 | 智谱 AI |
+| `glm` | `glm-5`, `glm-4.7`, `zhipu` | GLM 5.0 | 智谱 AI |
```

#### 2.2 更新模型参数表格 (第 136-144 行)

```diff
### 模型参数

| 模型 | Context Window | Max Output Tokens |
|------|----------------|-------------------|
| DeepSeek V3.2 | 128K | 128K |
| Kimi K2 Thinking | 256K | 32K |
-| MiniMax M2.1 | 200K | 128K |
+| MiniMax M2.5 | 200K | 128K |
| Qwen3 Max | 256K | 32K |
-| GLM 4.7 | 200K | 128K |
+| GLM 5.0 | 200K | 128K |
```

#### 2.3 更新版本别名使用示例 (第 240-246 行)

```diff
 # 使用版本别名（明确指定版本）
 /model deepseek-v3.2   # DeepSeek V3.2
-/model glm-4.7         # GLM 4.7
+/model glm-5           # GLM 5.0
-/model minimax-m2.1    # MiniMax M2.1
+/model minimax-m2.5    # MiniMax M2.5
 /model kimi-k2         # Kimi K2 Thinking
 /model qwen3-max       # Qwen3 Max
```

#### 2.4 添加更新日志 (第 295-302 行)

```diff
 ## 更新日志

+### v1.2.0
+- 更新 MiniMax 模型至 M2.5 版本
+- 更新 GLM 模型至 5.0 版本（model_id: glm-5）
+- 新增版本别名 `glm-5` 和 `minimax-m2.5`
+- 保留旧版本别名向后兼容
+
 ### v1.1.0
 - 更新 MiniMax 模型至 M2.1 版本
```

---

### 3. package.json

#### 3.1 版本号升级 (第 3 行)

```diff
 {
   "name": "claude-code-model-router",
-  "version": "1.1.0",
+  "version": "1.2.0",
```

---

## 向后兼容性

✅ **完全向后兼容**

- 旧版本别名 `glm-4.7` 和 `minimax-m2.1` 仍然可用
- 使用旧别名的用户无需修改配置
- 环境变量名称保持不变

---

## 使用方式

### 通过短名称切换（默认使用最新版本）

```bash
/model glm        # 使用 GLM 5.0
/model minimax    # 使用 MiniMax M2.5
```

### 通过版本别名切换（明确指定版本）

```bash
/model glm-5         # 使用 GLM 5.0
/model glm-4.7       # 使用 GLM 4.7（旧版本兼容）
/model minimax-m2.5  # 使用 MiniMax M2.5
/model minimax-m2.1  # 使用 MiniMax M2.1（旧版本兼容）
```

---

## 测试建议

1. **基本功能测试**
   ```bash
   ccmr start
   ccmr models  # 检查模型列表
   ```

2. **API 调用测试**
   ```bash
   # 测试 GLM-5
   curl -X POST http://localhost:8080/v1/messages \
     -H "Content-Type: application/json" \
     -d '{"model": "glm-5", "max_tokens": 100, "messages": [{"role": "user", "content": "Hello"}]}'

   # 测试 MiniMax M2.5
   curl -X POST http://localhost:8080/v1/messages \
     -H "Content-Type: application/json" \
     -d '{"model": "minimax-m2.5", "max_tokens": 100, "messages": [{"role": "user", "content": "Hello"}]}'
   ```

3. **向后兼容性测试**
   ```bash
   # 确保旧别名仍然可用
   /model glm-4.7
   /model minimax-m2.1
   ```

---

## 发布检查清单

- [x] 代码修改完成
- [ ] 本地构建测试 (`npm run build`)
- [ ] 本地功能测试
- [ ] 更新 CHANGELOG.md
- [ ] 创建 Git tag
- [ ] 发布到 npm
- [ ] 更新 GitHub Release

---

## 相关文档

- 官方分析文档: `ref-doc/new-model-support-analysis.md`
- GLM 官方文档: https://docs.bigmodel.cn/cn/coding-plan/tool/claude
- MiniMax 官方文档: https://platform.minimaxi.com/docs/coding-plan/claude-code
