# 升级指南：从 1.2.3 切换到 1.2.4 并使用 GLM-5.2

> 面向**现有 CCMR-Plus 1.2.3 用户**，平滑升级到 1.2.4 并启用 GLM-5.2。
>
> 新用户请直接看 [GLM-5.2 使用指南](glm-5.2-usage-guide.md)。

## 升级前后对比

| 项目 | v1.2.3 | v1.2.4 |
|------|--------|--------|
| GLM model_id | `glm-5` | **`glm-5.2`** |
| GLM 上下文窗口 | 200K | **1M** |
| 新增别名 | — | `glm-5.2`、`glm-52` |
| API Key | `GLM_API_KEY` | 不变（复用） |
| 配置目录 | `~/.ccmr-plus/` | 不变 |

**无需重新申请 API Key，无需迁移配置。** 升级是平滑的。

---

## 第 1 步：升级包

你是全局安装（1.2.3），直接覆盖升级：

```bash
npm install -g @andyzheung/ccmr@1.2.4
```

若用淘宝镜像且尚未同步到 1.2.4，临时指定官方源：

```bash
npm install -g @andyzheung/ccmr@1.2.4 --registry https://registry.npmjs.org/
```

## 第 2 步：验证升级

```bash
ccmr-plus --version
# 应显示 1.2.4

ccmr-plus models
# glm 应显示 "GLM 5.2"
```

## 第 3 步：启动并使用 GLM-5.2

```bash
# 启动网关
ccmr-plus start

# 新终端启动 Claude Code（连接网关）
ccmr-plus claude
```

在 Claude Code 中切换模型：

```bash
/model glm-5.2     # 明确指定 5.2（推荐）
/model glm         # 或用短名称（1.2.4 起自动指向 5.2）
```

用 `/status` 确认当前模型。

## 第 4 步：验证 5.2 生效

网关启动后，curl 打本地网关（无需 Key，网关用 `.env` 里的）：

```bash
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -d '{"model":"glm-5.2","max_tokens":100,"messages":[{"role":"user","content":"你好"}]}'
```

返回正常回复即生效。

---

## 可选：用 models.yaml 自定义

想调整 5.2 参数（如 max_tokens），在工作目录放 `models.yaml` 覆盖：

```yaml
models:
  glm:
    max_tokens: 32768   # 按需调整
```

---

## 升级后出问题？

查看 [回退指南](rollback-guide.md)：
- **只回退模型到 5.0**（`models.yaml` 覆盖 `model_id: glm-5`，不动包，秒级）
- 或**整个包降级到 1.2.3**（`npm install -g @andyzheung/ccmr@1.2.3`）

> ⚠️ 1.2.3 没有 `glm-5.2`/`glm-52` 别名，回退后用 `/model glm` 或 `/model glm-5`。

---

## 相关文档

- [GLM-5.2 使用指南（完整）](glm-5.2-usage-guide.md)
- [回退指南](rollback-guide.md)
- [遗留事项跟踪](pending-items.md)
