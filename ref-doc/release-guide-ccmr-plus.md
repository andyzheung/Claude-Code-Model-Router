# CCMR-Plus v1.2.0 发布指南

## 项目信息

### 项目标识

| 项目 | 值 |
|------|-----|
| **npm 包名** | `@andyzheung/ccmr` |
| **CLI 命令** | `ccmr-plus` |
| **配置目录** | `~/.ccmr-plus/` |
| **配置文件** | `.ccmr-plus.yaml` |
| **版本** | 1.2.0 |
| **作者** | andyzheung |

### 与原版区别

| 项目 | 原版 | CCMR-Plus |
|------|------|-----------|
| npm 包名 | `claude-code-model-router` | `@andyzheung/ccmr` |
| CLI 命令 | `ccmr` | `ccmr-plus` |
| 配置目录 | `~/.claude-gateway/` | `~/.ccmr-plus/` |
| 配置文件 | `.claude-router.yaml` | `.ccmr-plus.yaml` |
| 支持模型 | GLM 4.7, MiniMax M2.1 | **GLM 5.0, MiniMax M2.5** |

---

## 修改文件清单

```
modified:   package.json      (包名、作者、仓库)
modified:   src/config.ts     (配置文件路径)
modified:   src/cli.ts        (配置目录)
modified:   README.md         (所有命令和说明)
```

---

## 配置隔离说明

### 完全隔离的配置系统

```
原版 ccmr:
  ~/.claude-gateway/settings.json
  .claude-router.yaml

CCMR-Plus:
  ~/.ccmr-plus/settings.json        ← 独立配置目录
  .ccmr-plus.yaml                   ← 独立配置文件
```

**两者互不干扰，可以同时安装使用！**

---

## 发布流程

### 1. 准备工作

```bash
# 确认当前分支
git branch  # 应该在 dev 分支

# 确认修改状态
git status
```

### 2. 创建 npm account（如果还没有）

```bash
# 登录 npm
npm login

# 或使用 scope
npm login --scope=@andyzheung
```

### 3. 发布到 npm

```bash
# 构建项目
npm run build

# 发布到 npm（公开包）
npm publish --access public

# 如果是首次发布 scoped 包，需要加 --access public
npm publish --access public
```

### 4. Git 提交

```bash
# 提交修改
git add .
git commit -m "Release v1.2.0: CCMR-Plus with GLM-5 and MiniMax M2.5

- Change npm package to @andyzheung/ccmr
- Change CLI command to ccmr-plus
- Use ~/.ccmr-plus/ config directory (isolated from original)
- Support GLM 5.0 (glm-5)
- Support MiniMax M2.5
- Full backward compatibility with old aliases"

# 推送到远程
git push origin dev
```

---

## 用户使用指南

### 安装

```bash
# 方式一：npx（推荐，无需安装）
npx @andyzheung/ccmr init

# 方式二：全局安装
npm install -g @andyzheung/ccmr

# 更新到最新版本
npm update -g @andyzheung/ccmr
```

### 配置

**步骤 1：初始化配置**
```bash
ccmr-plus init
```

这会生成 `.env` 文件。

**步骤 2：编辑 .env 文件**
```bash
# GLM 5.0
GLM_API_KEY=your_glm_api_key_here

# MiniMax M2.5
MINIMAX_API_KEY=your_minimax_api_key_here

# DeepSeek
DEEPSEEK_API_KEY=your_deepseek_api_key_here

# Kimi
KIMI_API_KEY=your_kimi_api_key_here

# Qwen
QWEN_API_KEY=your_qwen_api_key_here
```

**步骤 3：启动网关**
```bash
ccmr-plus start
```

### 使用

```bash
# 启动 Claude Code（使用第三方模型）
ccmr-plus claude

# 在 Claude Code 中切换模型
/model glm-5           # GLM 5.0
/model minimax-m2.5    # MiniMax M2.5
/model glm-4.7         # 旧版本兼容
/model minimax-m2.1    # 旧版本兼容
```

---

## 验证安装

### 检查版本

```bash
ccmr-plus --version
# 或
npx @andyzheung/ccmr --version
```

### 查看可用模型

```bash
ccmr-plus models
```

### 测试 API

```bash
# 测试 GLM-5
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $GLM_API_KEY" \
  -d '{
    "model": "glm-5",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "你好"}]
  }'
```

---

## 配置文件位置

| 配置类型 | 路径 |
|----------|------|
| 用户配置目录 | `~/.ccmr-plus/` |
| 用户配置文件 | `~/.ccmr-plus/settings.json` |
| 环境变量 | `.env` (项目根目录) |
| 模型配置 | `.ccmr-plus.yaml` (项目根目录，可选) |

---

## 与原版共存

### 同时安装两个版本

```bash
# 安装原版
npm install -g claude-code-model-router

# 安装 CCMR-Plus
npm install -g @andyzheung/ccmr

# 使用原版
ccmr start
ccmr claude

# 使用 CCMR-Plus
ccmr-plus start
ccmr-plus claude
```

### 配置隔离

```
原版:
  ~/.claude-gateway/     ← 原版配置目录
  ~/.claude/             ← Claude Code 默认配置

CCMR-Plus:
  ~/.ccmr-plus/          ← CCMR-Plus 独立配置目录
```

**完全隔离，互不影响！**

---

## 发布检查清单

- [x] 代码修改完成
- [x] 构建成功
- [ ] Git 提交
- [ ] 推送到 GitHub
- [ ] 发布到 npm (`npm publish --access public`)
- [ ] 测试 npm 安装 (`npm install -g @andyzheung/ccmr`)
- [ ] 创建 GitHub Release

---

## GitHub Release 模板

```markdown
## Release v1.2.0 - CCMR-Plus

### 🎉 Fork 版本特性

- npm 包名: `@andyzheung/ccmr`
- CLI 命令: `ccmr-plus`
- 配置目录: `~/.ccmr-plus/`
- 与原版完全隔离，可同时安装使用

### ✨ 新功能

- 支持 **GLM 5.0** (glm-5)
- 支持 **MiniMax M2.5** (MiniMax-M2.5)

### 📝 安装

```bash
npm install -g @andyzheung/ccmr
# 或
npx @andyzheung/ccmr init
```

### 🔧 使用

```bash
ccmr-plus init
ccmr-plus start
ccmr-plus claude
```

### 📚 文档

- GitHub: https://github.com/andyzheung/Claude-Code-Model-Router
- npm: https://www.npmjs.com/package/@andyzheung/ccmr

### 🙏 致谢

Forked from [luwill/Claude-Code-Model-Router](https://github.com/luwill/Claude-Code-Model-Router)
```

---

## 常见问题

### Q: 和原版 ccmr 的区别？

A: CCMR-Plus 是独立的 fork 版本：
- 不同的 npm 包名（`@andyzheung/ccmr`）
- 不同的 CLI 命令（`ccmr-plus`）
- 不同的配置目录（`~/.ccmr-plus/`）
- 支持更新的模型（GLM-5, MiniMax M2.5）

### Q: 可以和原版同时安装吗？

A: 可以！两者配置完全隔离：
```bash
# 原版
ccmr start

# CCMR-Plus
ccmr-plus start
```

### Q: 原版配置需要迁移吗？

A: 不需要。CCMR-Plus 使用独立的配置目录 `~/.ccmr-plus/`，与原版 `~/.claude-gateway/` 完全隔离。

### Q: 如何切换回原版？

A: 直接使用原版命令即可：
```bash
ccmr start
ccmr claude
```
