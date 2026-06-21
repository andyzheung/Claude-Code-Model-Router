# CCMR-Plus 回退指南（GLM-5.2 ↔ GLM-5.0 / v1.2.4 ↔ v1.2.3）

> 当 v1.2.4 的 GLM-5.2 出现异常，或包本身有问题时，按本指南快速回退。
>
> 两种方式：**只回退模型（推荐）** 或 **整个包降级**。

## 何时需要回退

- GLM-5.2 调用报错、响应异常、质量下降 → 想临时回到稳定的 GLM 5.0
- v1.2.4 包本身有 bug → 想降级到 v1.2.3

---

## 方式 A：只回退 GLM 模型到 5.0（推荐，最轻量）

**不动 npm 包**（仍是 1.2.4），仅用本地 `models.yaml` 把 GLM 的 `model_id` 覆盖回 `glm-5`。

### 步骤

1. 在**启动 ccmr-plus 的工作目录**创建（或编辑）`models.yaml`：

```yaml
models:
  glm:
    display_name: "GLM 5.0"
    provider: zhipu
    model_id: glm-5              # 覆盖回 5.0
    base_url: https://open.bigmodel.cn/api/anthropic
    api_key_env: GLM_API_KEY
    auth_header: x-api-key
    max_tokens: 128000
    context_window: 200000
```

2. 重启网关：`ccmr-plus start`

3. 验证：`ccmr-plus models`（glm 应显示 GLM 5.0），或 curl 打网关测试。

### 恢复回 5.2

删除 `models.yaml` 里的 glm 覆盖（或整个文件），或把 `model_id` 改回 `glm-5.2`，重启即可。

---

## 方式 B：整个包降级到 v1.2.3

适合 v1.2.4 包本身有问题、想完全回到旧版本。

```bash
npm install -g @andyzheung/ccmr@1.2.3
ccmr-plus --version        # 应显示 1.2.3
```

如果用淘宝镜像且未同步，指定官方源：

```bash
npm install -g @andyzheung/ccmr@1.2.3 --registry https://registry.npmjs.org/
```

npx 方式：

```bash
npx @andyzheung/ccmr@1.2.3 start
```

---

## 回退后的注意事项

| 事项 | 说明 |
|------|------|
| 别名差异 | v1.2.3 **没有** `glm-5.2` / `glm-52` 别名，请用 `/model glm` 或 `/model glm-5` |
| 配置不受影响 | `~/.ccmr-plus/`、`.env` 不随版本切换变化，API Key、会话都保留 |
| models.yaml 覆盖 | 若方式 A 留了覆盖文件，降级到 1.2.3（方式 B）后要改回 `glm-5` 或删除，否则 1.2.3 用 `glm-5.2` 会报错 |
| 当前模型 | 回退后 `/model glm` 走的是 GLM 5.0 |

---

## 验证回退成功

```bash
# 1. 确认版本（方式 B 时）
ccmr-plus --version

# 2. 确认模型列表
ccmr-plus models

# 3. 启动后 curl 测试 GLM
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -d '{"model":"glm","max_tokens":100,"messages":[{"role":"user","content":"你好"}]}'
```

---

## 相关文档

- [GLM-5.2 升级与使用指南](glm-5.2-usage-guide.md)
- [项目 README](../../README.md)
