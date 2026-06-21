# CCMR-Plus GLM-5.2 深度思考（thinking）模式使用指南

> 本指南说明 GLM-5.2 的**深度思考（thinking）模式**如何在 CCMR-Plus 中使用、验证、调参与关闭。
>
> 适用版本：**v1.2.5+**（thinking 模式默认开启）。
>
> 官方模型文档：https://docs.bigmodel.cn/cn/guide/models/text/glm-5.2

## 目录

- [一、什么是 thinking 模式](#一什么是-thinking-模式)
- [二、默认行为](#二默认行为)
- [三、工作原理](#三工作原理)
- [四、验证 thinking 是否生效](#四验证-thinking-是否生效)
- [五、调整思考强度](#五调整思考强度)
- [六、关闭 thinking 模式](#六关闭-thinking-模式)
- [七、注意事项](#七注意事项)
- [八、常见问题 FAQ](#八常见问题-faq)

---

## 一、什么是 thinking 模式

GLM-5.2 支持「深度思考」：在给出最终回答前，模型先进行一段**可见的推理过程**（thinking），再输出结论。

启用后，响应里会多出一个 `thinking` 内容块，例如：

```json
{
  "content": [
    {
      "type": "thinking",
      "thinking": "1. 分析请求...\n2. 确定约束...\n3. 生成答案...",
      "signature": "7a3987723726414a98704bcd"
    },
    {
      "type": "text",
      "text": "最终回答"
    }
  ]
}
```

这是**标准 Anthropic `thinking` block** 格式（带 `signature` 字段），Claude Code 可原生识别与渲染。

---

## 二、默认行为

从 **v1.2.5 起，GLM-5.2 默认开启 thinking 模式**，无需任何额外配置。

默认注入的参数（定义在 `src/config.ts` 的 glm 配置 `extra_body` 字段）：

```yaml
extra_body:
  thinking:
    type: enabled
  reasoning_effort: max
```

即：思考开启 + 最大思考强度。

---

## 三、工作原理

CCMR-Plus 是纯**配置驱动**的网关，thinking 通过通用的 `extra_body` 机制实现，**不硬编码任何智谱专属逻辑**：

1. **配置层**（`src/config.ts`）：glm 模型带 `extra_body: { thinking: {...}, reasoning_effort: 'max' }`。
2. **路由层**（`src/router.ts` `buildRequestBody`）：转发请求前，把 `extra_body` 里的字段**合并进请求体**——但只在**客户端未显式指定**该字段时才注入（**客户端优先**）。
3. **流式 / 非流式**：两条路径都调用 `buildRequestBody`，自动覆盖。

注入逻辑：

```ts
if (modelConfig.extra_body) {
  for (const [key, value] of Object.entries(modelConfig.extra_body)) {
    if (!(key in body)) body[key] = value;  // 客户端优先
  }
}
```

> 因为是「客户端优先」，所以如果你在 Claude Code 里手动开启了 extended thinking（客户端自带 `thinking` 参数），路由不会覆盖你的设置。

---

## 四、验证 thinking 是否生效

### 方法 1：直接打网关（推荐）

前提：网关已启动（`ccmr-plus start`），`.env` 已配 `GLM_API_KEY`。

```bash
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm",
    "max_tokens": 512,
    "messages": [{"role":"user","content":"用一句话证明你会思考"}]
  }'
```

- ✅ 返回的 `content` 数组里**包含 `type: "thinking"` 的块** → thinking 生效。
- ❌ 只有 `type: "text"` 块、没有 thinking 块 → thinking 未注入（检查 `models.yaml` 是否覆盖关掉了）。

### 方法 2：在 Claude Code 里观察

切换到 GLM 后（`/model glm`），提问一个需要推理的问题。Claude Code 会把 thinking 过程作为模型推理展示出来（而非普通输出）。

---

## 五、调整思考强度

`reasoning_effort` 控制思考强度，可选值（参考智谱官方）：

| 值 | 含义 | 适用场景 |
|------|------|----------|
| `low` | 轻量思考 | 简单问答、追求低延迟 |
| `medium` | 中等 | 一般任务 |
| `high` | 较强 | 复杂推理 |
| `max` | 最大（**默认**） | 最难推理、代码、数学 |

想改强度，在**启动网关的工作目录**放 `models.yaml` 覆盖：

```yaml
models:
  glm:
    extra_body:
      thinking:
        type: enabled
      reasoning_effort: medium   # 改成你想要的级别
```

重启网关（`ccmr-plus start`）生效。

---

## 六、关闭 thinking 模式

思考模式会增加 token 消耗与响应延迟，某些场景（如纯翻译、简单补全）你可能想关掉。

在 `models.yaml` 里把 `extra_body` 覆盖为空对象即可：

```yaml
models:
  glm:
    extra_body: {}   # 关闭 thinking，回到普通模式
```

重启网关生效。删除该覆盖（或整个 `models.yaml`）即恢复默认开启。

---

## 七、注意事项

1. **Token 消耗增加**
   thinking 过程本身消耗 output tokens。默认 `max` 强度下，简单问题的输出 token 也可能明显增多。计费敏感时考虑调低 `reasoning_effort` 或关闭。

2. **响应延迟增加**
   思考需要时间。首 token 延迟会比普通模式高，属正常现象。

3. **`max_tokens` 与 thinking**
   思考过程占用 `max_tokens` 预算。CCMR-Plus 默认上限 65536，一般够用。若任务很长，可在 `models.yaml` 调高（先确认智谱对该模型的真实上限）。

4. **temperature**
   智谱官方 OpenAI 端点示例要求 thinking 时 `temperature: 1.0`。实测 **Anthropic 端点不强制**，CCMR-Plus 不覆盖客户端 temperature。若遇到异常可尝试显式传 `temperature: 1.0`。

5. **客户端优先**
   若 Claude Code 自身传了 `thinking` 参数，路由不覆盖。如需强制使用网关配置，请确保客户端未传该参数。

6. **别名不受影响**
   `glm-5`、`glm-5.0`、`zhipu` 等别名都指向当前 GLM 5.2 配置，因此 thinking 对它们同样生效。

---

## 八、常见问题 FAQ

**Q1：thinking 是 GLM 专属的吗？会影响其它模型吗？**
不会。`extra_body` 是 GLM 配置里才有的字段，其它模型（DeepSeek、MiniMax 等）没有配置，行为不变。

**Q2：关掉 thinking 后还能用 GLM 5.2 的 1M 上下文吗？**
能。thinking 与上下文窗口是两回事，关闭 thinking 不影响 1M 上下文能力。

**Q3：为什么我的请求没有返回 thinking 块？**
按顺序排查：① `models.yaml` 是否覆盖关掉了 `extra_body`；② 客户端是否自己传了 `thinking` 参数（可能类型不是 enabled）；③ 问题是否过于简单导致模型未触发思考（可换 `reasoning_effort: max` 或更难的题试试）。

**Q4：thinking 会泄露给最终用户吗？**
在 Claude Code 中，thinking 作为模型推理过程展示。是否对终端用户可见取决于你的前端如何渲染 Anthropic thinking block。

**Q5：能在请求里临时覆盖 reasoning_effort 吗？**
可以。由于「客户端优先」，请求体里直接带 `reasoning_effort` 字段会优先于网关默认配置（前提是上游端点接受该字段）。

---

## 相关文档

- [GLM-5.2 升级与使用指南](glm-5.2-usage-guide.md)
- [升级指南](upgrade-guide.md)
- [回退指南](rollback-guide.md)
- [遗留事项跟踪](pending-items.md)
- 官方模型文档：https://docs.bigmodel.cn/cn/guide/models/text/glm-5.2
