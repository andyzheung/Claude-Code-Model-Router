# CCMR-Plus GLM-5.2 升级与使用指南

> 本指南对应 **CCMR-Plus v1.2.4**，说明如何将 GLM 升级到 **5.2**（model_id: `glm-5.2`）并在 Claude Code 中使用。
>
> 官方模型文档：https://docs.bigmodel.cn/cn/guide/models/text/glm-5.2

## 目录

- [一、本次升级做了什么](#一本次升级做了什么)
- [二、前置条件](#二前置条件)
- [三、安装或升级到支持 GLM-5.2 的版本](#三安装或升级到支持-glm-52-的版本)
- [四、配置 GLM API Key](#四配置-glm-api-key)
- [五、启动网关](#五启动网关)
- [六、在 Claude Code 中切换到 GLM-5.2](#六在-claude-code-中切换到-glm-52)
- [七、验证 GLM-5.2 是否可用（重要）](#七验证-glm-52-是否可用重要)
- [八、GLM-5.2 模型参数与别名](#八glm-52-模型参数与别名)
- [九、可选：本地覆盖配置](#九可选本地覆盖配置)
- [十、已知限制与注意事项](#十已知限制与注意事项)
- [十一、故障排除](#十一故障排除)
- [十二、常见问题 FAQ](#十二常见问题-faq)

---

## 一、本次升级做了什么

CCMR-Plus 是纯**配置驱动**的模型网关——所有模型定义集中在 `src/config.ts`，路由层（`router.ts`）动态读取配置，不硬编码任何模型。因此升级 GLM 到 5.2 **只改配置、不改路由逻辑**。

本次（v1.2.4）改动点：

| 项目 | 旧值（v1.2.3） | 新值（v1.2.4） |
|------|----------------|----------------|
| `display_name` | GLM 5.0 | **GLM 5.2** |
| `model_id` | `glm-5` | **`glm-5.2`** |
| `context_window` | 200000 (200K) | **1000000 (1M)** |
| `max_tokens` | 128000 | **65536**（对齐官方示例） |
| 新增别名 | — | `glm-5.2`、`glm-52` |

> 旧别名 `glm-5`、`glm-5.0`、`glm-4.7`、`glm-4.6`、`zhipu`、`chatglm` **全部保留**，向后兼容（它们现在都会指向 5.2）。

---

## 二、前置条件

1. **Node.js ≥ 18**
2. **Claude Code 已安装**（`npm install -g @anthropic-ai/claude-code`）
3. **智谱 GLM API Key**：在 https://open.bigmodel.cn/ 控制台获取
   - GLM 5.0 与 5.2 **共用同一个 API Key**，无需重新申请
4. 确认你的智谱账户已开通 **GLM-5.2** 的调用权限（部分新模型可能需要单独申请额度）

---

## 三、安装或升级到支持 GLM-5.2 的版本

> ⚠️ v1.2.4 需要**已发布到 npm** 才能用 `npx`/全局安装方式。如果尚未发布，请使用【方式三：本地源码】。

### 方式一：npx（推荐，无需安装）

```bash
# 初始化配置文件（生成 .env 与 models.yaml 模板）
npx @andyzheung/ccmr@1.2.4 init

# 启动网关
npx @andyzheung/ccmr@1.2.4 start
```

### 方式二：全局安装

```bash
# 安装或升级到 1.2.4
npm install -g @andyzheung/ccmr@1.2.4

# 验证版本
ccmr-plus --version
# 应输出：1.2.4
```

### 方式三：本地源码（版本尚未发布时使用）

```bash
git clone https://github.com/andyzheung/Claude-Code-Model-Router.git
cd Claude-Code-Model-Router
npm install
npm run build          # 生成 dist/

# 方式 A：链接为全局命令
npm link
ccmr-plus --version    # 1.2.4

# 方式 B：直接运行（不链接）
node dist/cli.js start
```

---

## 四、配置 GLM API Key

编辑项目目录下的 `.env` 文件（由 `ccmr-plus init` 生成）：

```bash
# GLM（智谱 AI）- https://open.bigmodel.cn/
GLM_API_KEY=你的智谱_API_Key
```

> `.env` 必须放在**启动网关时的工作目录**。`ccmr-plus start` 会在当前目录读取它。

---

## 五、启动网关

```bash
ccmr-plus start
```

正常启动后会看到 GLM 显示为 **GLM 5.2** 且标记 `[Ready]`：

```
Gateway running at: http://localhost:8080
Default model: deepseek

Available models:
  - glm: GLM 5.2 [Ready]          ← 已是 5.2
  - minimax: MiniMax M2.5 [Ready]
  - deepseek: DeepSeek V3.2 [Ready]
  ...
```

- 若显示 `[No API Key]`：说明 `.env` 里的 `GLM_API_KEY` 未配置或未生效。
- 自定义端口：`ccmr-plus start --port 9000`

---

## 六、在 Claude Code 中切换到 GLM-5.2

新开一个终端启动 Claude Code（连接到网关）：

```bash
ccmr-plus claude
# 或指定端口：ccmr-plus claude --gateway-port 9000
```

进入 Claude Code 后，用以下任意一种方式切换模型：

```bash
# 推荐：用版本别名，明确指定 5.2
/model glm-5.2

# 或用短名称（指向当前最新版本 5.2）
/model glm

# 这些别名也都会指向 5.2（向后兼容）
/model glm-5
/model glm-5.0
/model zhipu
```

切换后用 `/status` 确认当前模型。

---

## 七、验证 GLM-5.2 是否可用（重要）

> 这是本次升级的**关键收尾**：官方文档展示的是 OpenAI 兼容端点，而 CCMR-Plus 走的是 **Anthropic 兼容端点**（`/api/anthropic`）。需实测确认该端点支持 `glm-5.2`。

### 验证 1：直接打智谱 Anthropic 端点（验证端点本身）

```bash
# 先确保已设置 Key
export GLM_API_KEY="你的智谱_API_Key"

curl -X POST https://open.bigmodel.cn/api/anthropic/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $GLM_API_KEY" \
  -d '{
    "model": "glm-5.2",
    "max_tokens": 256,
    "messages": [{"role":"user","content":"你好，请用一句话自我介绍"}]
  }'
```

- ✅ **返回 200 + 正常回复** → Anthropic 端点支持 `glm-5.2`，可继续。
- ❌ **返回 404 / `model not found` / `invalid model`** → 该端点暂不支持 5.2，需联系智谱或换用 OpenAI 端点（需改 `base_url`，超出本指南范围）。

### 验证 2：打 CCMR-Plus 网关（验证整条链路）

前提：网关已 `ccmr-plus start` 运行中。

```bash
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-5.2",
    "max_tokens": 256,
    "messages": [{"role":"user","content":"你好"}]
  }'
```

- 网关会自动把 `glm-5.2` 解析为 `glm` 配置，用 `.env` 里的 `GLM_API_KEY` 转发到智谱。
- ✅ 返回正常回复 → 整条链路（Claude Code → 网关 → 智谱）打通。

> Claude Code 默认使用**流式（stream）**请求。上面的 curl 是非流式验证，仅用于快速确认；流式由网关自动转发，无需额外配置。

---

## 八、GLM-5.2 模型参数与别名

| 项目 | 值 |
|------|-----|
| 短名称 | `glm` |
| `model_id` | `glm-5.2` |
| 提供商 | 智谱 AI（zhipu） |
| Base URL | `https://open.bigmodel.cn/api/anthropic` |
| 鉴权头 | `x-api-key` |
| Context Window | **1M**（1,000,000） |
| Max Output Tokens | 64K（65,536） |
| 流式 / 工具调用 | 支持 / 支持 |

**可用别名（均指向 GLM 5.2）：**

| 别名 | 说明 |
|------|------|
| `glm-5.2`、`glm-52` | 新增，明确指定 5.2 |
| `glm-5`、`glm-5.0` | 兼容旧配置 |
| `glm-4.7`、`glm-4.6` | 兼容旧配置 |
| `zhipu`、`chatglm` | 厂商别名 |

---

## 九、可选：本地覆盖配置

如果不想等官方发版，或想临时调整参数，可在**项目根目录**放置 `models.yaml`，同名模型会覆盖默认配置：

```yaml
# models.yaml
models:
  glm:
    display_name: "GLM 5.2"
    provider: zhipu
    model_id: glm-5.2
    base_url: https://open.bigmodel.cn/api/anthropic
    api_key_env: GLM_API_KEY
    auth_header: x-api-key
    max_tokens: 65536
    context_window: 1000000
```

CCMR-Plus 启动时会自动合并（`models.yaml` > 默认配置）。

---

## 十、已知限制与注意事项

1. **深度思考（thinking）模式暂未支持**
   - GLM-5.2 的卖点是 `thinking: enabled` + `reasoning_effort: max` 深度思考。
   - 但官方文档展示的是 **OpenAI 端点**参数，CCMR-Plus 走 **Anthropic 端点**，该端点是否接受这些参数**尚未验证**。
   - 当前版本（v1.2.4）**不注入 thinking 参数**，GLM-5.2 以普通模式运行。
   - 是否支持思考模式取决于第七节的实测结果；确认端点支持后，后续版本可通过 `extra_body` 配置字段注入（规划中）。

2. **temperature 注意**
   - 官方思考模式示例要求 `temperature: 1.0`。普通模式下 Claude Code 自行传参，通常无需干预。

3. **`glm-5` / `glm-5.0` 别名语义**
   - 它们现在指向 5.2。如果你的脚本依赖「glm-5 = 旧 5.0 行为」，请注意实际调用的是 5.2。

4. **Max Output Tokens**
   - 已按官方示例对齐为 65536。若你的任务需要更大单次输出，请先确认智谱对该模型的真实上限再调整。

---

## 十一、故障排除

| 现象 | 原因 / 解决 |
|------|-------------|
| `[No API Key]` | `.env` 未配置 `GLM_API_KEY`，或不在启动目录 |
| `model not found` / 404 | Anthropic 端点暂不支持 `glm-5.2`，见第七节验证 1 |
| `authentication_error` | API Key 错误或账户未开通 5.2 权限 |
| 端口 8080 被占用 | `ccmr-plus start --port 9000` |
| 切换模型无变化 | 确认是在**网关模式**（`ccmr-plus claude` 启动）而非官方 `claude` |
| 仍显示 GLM 5.0 | 你用的还是旧版本，确认 `ccmr-plus --version` 为 1.2.4，或用本地源码方式 |

---

## 十二、常见问题 FAQ

**Q1：升级后我原来的 `GLM_API_KEY` 还能用吗？**
能。5.0 和 5.2 共用同一个智谱 API Key，无需重新申请。

**Q2：`/model glm` 现在用的是 5.2 吗？**
是的。`glm` 短名称始终指向配置中的最新版本，v1.2.4 起即 5.2。

**Q3：为什么没有自动开启深度思考？**
见第十节。thinking 参数在 Anthropic 端点的支持情况待实测，确认后会在后续版本支持。

**Q4：如何回退到 5.0？**
安装旧版本：`npm install -g @andyzheung/ccmr@1.2.3`；或在 `models.yaml` 中把 `model_id` 覆盖回 `glm-5`。

**Q5：1M 上下文会被自动启用吗？**
`context_window` 是元信息字段，反映模型能力。是否真正用到长上下文取决于你的输入长度与智谱侧的实际承载。

**Q6：支持 Windows 吗？**
支持。所有命令在 Windows / macOS / Linux 上一致。

---

## 相关文档

- [用户使用指南（通用）](../user-guide.md)
- [项目 README](../../README.md)
- 官方模型文档：https://docs.bigmodel.cn/cn/guide/models/text/glm-5.2

## 反馈

- GitHub Issues：https://github.com/andyzheung/Claude-Code-Model-Router/issues
