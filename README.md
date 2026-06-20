[![npm version](https://img.shields.io/npm/v/@andyzheung/ccmr.svg)](https://www.npmjs.com/package/@andyzheung/ccmr)
[![npm downloads](https://img.shields.io/npm/dm/@andyzheung/ccmr.svg)](https://www.npmjs.com/package/@andyzheung/ccmr)


# CCMR-Plus (Claude Code Model Router Plus)

一个轻量级 API 网关，让你在使用 Claude Code 时可以切换到第三方 AI 模型。

> 这是 [claude-code-model-router](https://github.com/luwill/Claude-Code-Model-Router) 的增强版本，支持最新的 GLM-5.2 和 MiniMax M2.5 模型。

支持 Windows、macOS、Linux 跨平台使用。

## 快速开始

### 方式一：使用 npx（推荐）

```bash
# 1. 初始化配置文件
npx @andyzheung/ccmr init

# 2. 编辑 .env 文件，填入 API Keys

# 3. 启动网关
npx @andyzheung/ccmr start

# 4. 新开终端，启动 Claude Code
# 第三方模型（网关模式）：
npx @andyzheung/ccmr claude

# 官方订阅（默认模式）：
claude
```

### 方式二：全局安装

```bash
npm install -g @andyzheung/ccmr

# 然后使用 ccmr-plus 命令
ccmr-plus init
ccmr-plus start

# 启动 Claude Code
ccmr-plus claude    # 第三方模型（网关模式）
claude              # 官方订阅（默认模式）
```

## 命令说明

```bash
# 初始化配置文件
npx @andyzheung/ccmr init

# 启动网关
npx @andyzheung/ccmr start
npx @andyzheung/ccmr start --port 9000  # 指定端口

# 查看可用模型
npx @andyzheung/ccmr models

# 启动 Claude Code（网关模式，使用第三方模型）
npx @andyzheung/ccmr claude
npx @andyzheung/ccmr claude --gateway-port 9000  # 自定义网关端口

# 启动 Claude Code（官方订阅）
claude
```

### Claude Code 原生参数支持

`ccmr-plus claude` 命令完整支持 Claude Code 的原生启动参数：

```bash
# YOLO 模式（跳过所有权限确认）
ccmr-plus claude --dangerously-skip-permissions

# 继续上一次会话
ccmr-plus claude --continue
ccmr-plus claude -c

# YOLO 模式 + 继续上一次会话
ccmr-plus claude --dangerously-skip-permissions --continue

# 恢复指定会话（交互式选择）
ccmr-plus claude --resume
ccmr-plus claude -r

# 恢复指定会话 ID
ccmr-plus claude --resume <session-id>

# 调试模式
ccmr-plus claude --debug
ccmr-plus claude --verbose

# 连接 IDE
ccmr-plus claude --ide

# 指定权限模式
ccmr-plus claude --permission-mode bypassPermissions

# 打印模式（非交互式）
ccmr-plus claude -p "你的问题"
ccmr-plus claude --print --output-format json "你的问题"
```

**支持的完整参数列表：**

| 参数 | 说明 |
|------|------|
| `-c, --continue` | 继续最近的会话 |
| `-r, --resume [id]` | 恢复指定会话或打开会话选择器 |
| `--fork-session` | 恢复时创建新会话 ID |
| `--dangerously-skip-permissions` | 跳过所有权限检查（YOLO 模式） |
| `--permission-mode <mode>` | 权限模式：acceptEdits, bypassPermissions, default, dontAsk, plan |
| `-p, --print` | 打印模式（非交互式） |
| `--output-format <format>` | 输出格式：text, json, stream-json |
| `--model <model>` | 指定模型（覆盖网关路由） |
| `--system-prompt <prompt>` | 自定义系统提示 |
| `--add-dir <dirs...>` | 添加额外目录权限 |
| `-d, --debug` | 调试模式 |
| `--verbose` | 详细输出 |
| `--ide` | 自动连接 IDE |
| `--gateway-port <port>` | 指定网关端口（默认 8080） |

> **提示：** 任何 Claude Code 原生支持的参数都可以直接传递给 `ccmr-plus claude`

## 支持的模型

| 短名称 | 版本别名 | 模型 | 提供商 |
|--------|----------|------|--------|
| `deepseek` | `deepseek-v3.2`, `ds` | DeepSeek V3.2 | DeepSeek |
| `kimi` | `kimi-k2`, `kimi-k2-thinking` | Kimi K2 Thinking | Moonshot |
| `minimax` | `minimax-m2.5`, `minimax-m2.1`, `mm` | MiniMax M2.5 | MiniMax |
| `qwen` | `qwen3-max`, `qwen3` | Qwen3 Max | 阿里云 |
| `glm` | `glm-5.2`, `glm-5`, `glm-4.7`, `zhipu` | GLM 5.2 | 智谱 AI |

### 模型参数

| 模型 | Context Window | Max Output Tokens |
|------|----------------|-------------------|
| DeepSeek V3.2 | 128K | 128K |
| Kimi K2 Thinking | 256K | 32K |
| MiniMax M2.5 | 200K | 128K |
| Qwen3 Max | 256K | 32K |
| GLM 5.2 | 1M | 64K |

## 配置

### 环境变量 (.env)

```bash
DEEPSEEK_API_KEY=sk-xxx    # https://platform.deepseek.com/
KIMI_API_KEY=sk-xxx        # https://www.kimi.com/
MINIMAX_API_KEY=xxx        # https://platform.minimax.io/
QWEN_API_KEY=sk-xxx        # https://dashscope.console.aliyun.com/
GLM_API_KEY=xxx            # https://open.bigmodel.cn/
```

### 配置文件 (models.yaml)

可以自定义模型配置、添加别名等。运行 `ccmr-plus init` 命令会生成模板。

## 使用场景

### 双模式使用（配置完全隔离）

本工具通过独立的配置目录实现完全隔离，让你可以同时使用官方订阅和第三方模型：

```
┌─────────────────────────────────────────────────────────────────┐
│  模式1: 官方订阅（默认）                                          │
│  命令: claude                                                    │
│  配置: ~/.claude/settings.json                                  │
│  用途: 使用 Claude 官方模型（订阅额度）                           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  模式2: 第三方模型（网关）                                        │
│  命令: npx @andyzheung/ccmr claude                              │
│  配置: ~/.ccmr-plus/settings.json                              │
│  用途: 使用第三方 AI 模型（GLM-5.2, MiniMax M2.5 等）              │
└─────────────────────────────────────────────────────────────────┘
```

#### 为什么配置是隔离的？

- **官方模式**使用 `~/.claude/` 配置目录（Claude Code 默认）
- **网关模式**使用 `~/.ccmr-plus/` 配置目录（独立隔离）
- 两个配置目录完全独立，互不干扰
- 在网关模式切换模型不会影响官方模式

#### 使用步骤

**第一步：启动网关**
```bash
npx @andyzheung/ccmr start
```

**第二步：选择使用模式**

**使用官方订阅（终端 A）：**
```bash
claude
```
- 使用官方 Claude 模型（Sonnet, Opus, Haiku）
- 消耗订阅额度
- 配置存储在 `~/.claude/`

**使用第三方模型（终端 B）：**
```bash
npx @andyzheung/ccmr claude
```
- 使用第三方 AI 模型（GLM-5.2, MiniMax M2.5 等）
- 按 API 使用量付费
- 配置存储在 `~/.ccmr-plus/`

#### 跨平台支持

所有命令在 Windows、macOS、Linux 上完全相同，无需修改。

### 在 Claude Code 中切换模型

#### 官方模式（直接 `claude` 启动）

```
/model sonnet     # Claude Sonnet 4.5
/model opus       # Claude Opus 4.5
/model haiku      # Claude Haiku 3.5
```

#### 网关模式（`npx ... claude` 启动）

使用短名称或版本别名切换模型：

```bash
# 使用短名称（向后兼容）
/model deepseek   # 切换到 DeepSeek V3.2
/model qwen       # 切换到 Qwen3 Max
/model glm        # 切换到 GLM 5.2
/model kimi       # 切换到 Kimi K2 Thinking
/model minimax    # 切换到 MiniMax M2.1

# 使用版本别名（明确指定版本）
/model deepseek-v3.2   # DeepSeek V3.2
/model glm-5.2         # GLM 5.2
/model minimax-m2.5    # MiniMax M2.5
/model kimi-k2         # Kimi K2 Thinking
/model qwen3-max       # Qwen3 Max
```

**重要：** 两个模式的配置完全独立，在网关模式切换模型不会影响官方模式！

## API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/v1/messages` | POST | Anthropic Messages API |
| `/v1/models` | GET | 列出可用模型 |
| `/health` | GET | 健康检查 |

## 开发

```bash
# 克隆项目
git clone https://github.com/andyzheung/Claude-Code-Model-Router.git
cd Claude-Code-Model-Router

# 安装依赖
npm install

# 开发模式
npm run dev

# 构建
npm run build

# 本地测试
npm link
ccmr-plus start
```

## 故障排除

### 端口被占用

```bash
# 使用其他端口
npx @andyzheung/ccmr start --port 9000
```

### API Key 错误

1. 检查 .env 文件中的 Key 是否正确
2. 确认账户有余额
3. 运行 `npx @andyzheung/ccmr models` 查看状态

## 更新日志

### v1.2.4
- 更新 GLM 模型至 5.2 版本（model_id: glm-5.2）
- 更新 GLM 上下文窗口至 1M（1000000）
- 新增版本别名 `glm-5.2` 和 `glm-52`
- 保留旧版本别名（glm-5、glm-5.0 等）向后兼容

### v1.2.0
- 更新 MiniMax 模型至 M2.5 版本
- 更新 GLM 模型至 5.0 版本（model_id: glm-5）
- 新增版本别名 `glm-5` 和 `minimax-m2.5`
- 保留旧版本别名向后兼容

### v1.1.0
- 更新 MiniMax 模型至 M2.1 版本
- 更新 GLM 模型至 4.7 版本
- 更新 GLM API 端点至 `https://open.bigmodel.cn/api/anthropic`
- 新增版本别名支持（如 `glm-4.7`、`minimax-m2.1`）
- 优化日志显示，显示具体模型版本
- 更新各模型的 context window 和 max tokens 参数

### v1.0.1
- 添加 Claude Code 原生参数支持

### v1.0.0
- 初始版本发布

## License

MIT
