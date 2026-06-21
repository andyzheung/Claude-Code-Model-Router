# CCMR-Plus 用户使用指南

## 目录

- [什么是 CCMR-Plus](#什么是-ccmr-plus)
- [与原版的区别](#与原版的区别)
- [安装](#安装)
- [配置](#配置)
- [使用](#使用)
- [常见问题](#常见问题)

---

## 什么是 CCMR-Plus

CCMR-Plus (Claude Code Model Router Plus) 是一个轻量级 API 网关，让你在使用 Claude Code 时可以切换到第三方 AI 模型。

**核心特性：**
- ✅ 支持 **GLM 5.0**（智谱 AI 最新版本）
- ✅ 支持 **MiniMax M2.5**（MiniMax 最新版本）
- ✅ 支持其他模型：DeepSeek V3.2、Kimi K2、Qwen3 Max
- ✅ 完全兼容 Claude Code 的所有功能
- ✅ 与原版完全隔离，可同时使用

---

## 与原版的区别

| 特性 | 原版 ccmr | CCMR-Plus |
|------|-----------|-----------|
| npm 包名 | `claude-code-model-router` | `@andyzheung/ccmr` |
| CLI 命令 | `ccmr` | `ccmr-plus` |
| GLM 版本 | GLM 4.7 | **GLM 5.0** |
| MiniMax 版本 | MiniMax M2.1 | **MiniMax M2.5** |
| 配置目录 | `~/.claude-gateway/` | `~/.ccmr-plus/` |
| 配置文件 | `.claude-router.yaml` | `.ccmr-plus.yaml` |

**两者可以同时安装使用，互不干扰！**

---

## 安装

### 方式一：npx（推荐，无需安装）

```bash
# 初始化配置
npx @andyzheung/ccmr init

# 启动网关
npx @andyzheung/ccmr start

# 启动 Claude Code（新终端）
npx @andyzheung/ccmr claude
```

### 方式二：全局安装

```bash
# 安装
npm install -g @andyzheung/ccmr

# 验证安装
ccmr-plus --version
```

---

## 配置

### 步骤 1：初始化配置

```bash
ccmr-plus init
```

这会在当前目录创建两个文件：
- `.env` - API Keys 配置
- `.ccmr-plus.yaml` - 模型配置（可选）

### 步骤 2：配置 API Keys

编辑 `.env` 文件，填入你的 API Keys：

```bash
# GLM 5.0 - https://open.bigmodel.cn/
GLM_API_KEY=your_glm_api_key_here

# MiniMax M2.5 - https://platform.minimax.io/
MINIMAX_API_KEY=your_minimax_api_key_here

# DeepSeek V3.2 - https://platform.deepseek.com/
DEEPSEEK_API_KEY=your_deepseek_api_key_here

# Kimi K2 - https://www.kimi.com/
KIMI_API_KEY=your_kimi_api_key_here

# Qwen3 Max - https://dashscope.console.aliyun.com/
QWEN_API_KEY=your_qwen_api_key_here
```

**获取 API Key：**
- GLM: https://open.bigmodel.cn/
- MiniMax: https://platform.minimax.io/
- DeepSeek: https://platform.deepseek.com/
- Kimi: https://www.kimi.com/
- Qwen: https://dashscope.console.aliyun.com/

---

## 使用

### 基本使用流程

```bash
# 1. 启动网关
ccmr-plus start

# 输出示例：
# ============================================================
#   Claude Code Model Router
# ============================================================
#
# Gateway running at: http://localhost:8080
# Default model: deepseek
#
# Available models:
#   - deepseek: DeepSeek V3.2 [Ready]
#   - glm: GLM 5.0 [Ready]
#   - minimax: MiniMax M2.5 [Ready]
#
# Press Ctrl+C to stop the gateway.
# ============================================================

# 2. 新开终端，启动 Claude Code
ccmr-plus claude

# 3. 在 Claude Code 中切换模型
/model glm-5           # 使用 GLM 5.0
/model minimax-m2.5    # 使用 MiniMax M2.5
/model deepseek        # 使用 DeepSeek
```

### 在 Claude Code 中切换模型

#### 使用短名称（默认最新版本）

```bash
/model glm        # GLM 5.0
/model minimax    # MiniMax M2.5
/model deepseek   # DeepSeek V3.2
/model kimi       # Kimi K2 Thinking
/model qwen       # Qwen3 Max
```

#### 使用版本别名（明确指定版本）

```bash
/model glm-5           # GLM 5.0
/model glm-5.0         # GLM 5.0
/model glm-4.7         # GLM 4.7（兼容旧版本）
/model minimax-m2.5    # MiniMax M2.5
/model minimax-m2.1    # MiniMax M2.1（兼容旧版本）
```

### 其他命令

```bash
# 查看所有可用模型
ccmr-plus models

# 使用自定义端口启动网关
ccmr-plus start --port 9000

# 查看帮助
ccmr-plus --help
```

---

## 支持的模型

| 模型 | 版本别名 | Context Window | Max Output |
|------|----------|----------------|------------|
| GLM 5.0 | `glm-5`, `glm-5.0` | 200K | 128K |
| MiniMax M2.5 | `minimax-m2.5` | 200K | 128K |
| DeepSeek V3.2 | `deepseek`, `ds` | 128K | 128K |
| Kimi K2 | `kimi`, `kimi-k2` | 256K | 32K |
| Qwen3 Max | `qwen`, `qwen3-max` | 256K | 32K |

---

## 高级用法

### 指定网关端口

```bash
# 启动网关时指定端口
ccmr-plus start --port 9000

# 启动 Claude Code 时指定端口
ccmr-plus claude --gateway-port 9000
```

### 与原版同时使用

**⚠️ 端口冲突警告：** 两者默认端口都是 8080，需要使用不同端口！

```bash
# 终端 1：使用原版（默认端口 8080）
ccmr start

# 终端 2：使用 CCMR-Plus（使用端口 8081）
ccmr-plus start --port 8081

# 终端 3：启动 Claude Code 连接到 CCMR-Plus
ccmr-plus claude --gateway-port 8081
```

**配置完全隔离：**
- 原版：`~/.claude-gateway/`、端口 `8080`（默认）
- CCMR-Plus：`~/.ccmr-plus/`、端口 `8081`（需指定）

---

## 常见问题

### Q1: 如何确认安装成功？

```bash
ccmr-plus --version
# 应该显示：1.2.2
```

### Q2: API Key 在哪里配置？

编辑项目目录下的 `.env` 文件：

```bash
# GLM 5.0
GLM_API_KEY=your_key_here

# MiniMax M2.5
MINIMAX_API_KEY=your_key_here
```

### Q3: 端口 8080 被占用怎么办？

```bash
# 使用其他端口
ccmr-plus start --port 9000
```

### Q4: 如何查看当前使用的模型？

在 Claude Code 中输入：

```bash
/status
```

### Q5: 与原版 ccmr 冲突吗？

**不冲突！** 两者完全隔离：
- 不同的命令（`ccmr` vs `ccmr-plus`）
- 不同的配置目录
- 可以同时安装使用

### Q6: 如何卸载？

```bash
# 卸载全局安装
npm uninstall -g @andyzheung/ccmr

# 删除配置（可选）
rm -rf ~/.ccmr-plus
rm .env
rm .ccmr-plus.yaml
```

### Q7: 更新到最新版本

```bash
# 更新全局安装
npm update -g @andyzheung/ccmr

# 或重新安装
npm install -g @andyzheung/ccmr@latest
```

### Q8: 配置文件位置？

| 配置项 | 位置 |
|--------|------|
| 用户配置目录 | `~/.ccmr-plus/` |
| Claude Code 配置 | `~/.ccmr-plus/settings.json` |
| 环境变量 | 项目目录下的 `.env` |
| 模型配置 | 项目目录下的 `.ccmr-plus.yaml`（可选） |

### Q9: 支持 Claude Code 的所有参数吗？

是的！CCMR-Plus 完全支持 Claude Code 的所有原生参数：

```bash
ccmr-plus claude --dangerously-skip-permissions
ccmr-plus claude --continue
ccmr-plus claude --resume
ccmr-plus claude --print "你的问题"
# ... 等等
```

### Q10: GLM-5 和 GLM-4.7 有什么区别？

GLM-5 是智谱 AI 的最新版本，性能更强：
- 更好的代码生成能力
- 更长的上下文支持
- 更快的响应速度

---

## 支持与反馈

- **GitHub**: https://github.com/andyzheung/Claude-Code-Model-Router
- **npm**: https://www.npmjs.com/package/@andyzheung/ccmr
- **问题反馈**: https://github.com/andyzheung/Claude-Code-Model-Router/issues

---

## 致谢

CCMR-Plus 是 [claude-code-model-router](https://github.com/luwill/Claude-Code-Model-Router) 的增强版本，感谢原作者的开源贡献。
