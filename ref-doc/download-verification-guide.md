# CCMR-Plus 下载与验证指南

## 一、发布后下载时效

### ⚡ 立即可用

发布到 npm 后，**通常几秒钟内**就可以在全球 CDN 上下载使用。

```
发布完成 ──(几秒)──> npm CDN 同步 ──> 全球可下载
```

---

## 二、下载方式

### 方式一：npx（推荐，无需安装）

**最简单的方式，直接运行，无需安装：**

```bash
# 初始化配置
npx @andyzheung/ccmr init

# 启动网关
npx @andyzheung/ccmr start

# 启动 Claude Code
npx @andyzheung/ccmr claude
```

### 方式二：全局安装

```bash
# 安装
npm install -g @andyzheung/ccmr

# 验证安装
ccmr-plus --version

# 使用
ccmr-plus init
ccmr-plus start
ccmr-plus claude
```

### 方式三：本地开发

```bash
# 克隆仓库
git clone https://github.com/andyzheung/Claude-Code-Model-Router.git
cd Claude-Code-Model-Router

# 安装依赖
npm install

# 构建
npm run build

# 本地链接
npm link

# 使用
ccmr-plus init
ccmr-plus start
```

---

## 三、验证安装

### 3.1 检查版本

```bash
# npx 方式
npx @andyzheung/ccmr --version

# 全局安装方式
ccmr-plus --version
```

**预期输出：**
```
1.2.0
```

### 3.2 查看可用模型

```bash
# 确保 .env 文件已配置 API Keys
ccmr-plus models
```

**预期输出：**
```
Available models:
  - deepseek: DeepSeek V3.2 [Ready]
  - kimi: Kimi K2 Thinking [No API Key]
  - minimax: MiniMax M2.5 [Ready]
  - qwen: Qwen3 Max [No API Key]
  - glm: GLM 5.0 [Ready]
```

### 3.3 测试网关

```bash
# 终端 1：启动网关
ccmr-plus start

# 终端 2：测试 API
curl http://localhost:8080/health
```

**预期输出：**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "default_model": "deepseek",
  "models": {
    "deepseek": "available",
    "glm": "available",
    "minimax": "available",
    ...
  }
}
```

---

## 四、配置 API Keys

### 4.1 初始化配置

```bash
ccmr-plus init
```

这会创建 `.env` 文件：

```bash
# Claude Code Model Router - API Keys
# Fill in your API keys below

# DeepSeek - https://platform.deepseek.com/
DEEPSEEK_API_KEY=

# Kimi - https://www.kimi.com/
KIMI_API_KEY=

# MiniMax - https://platform.minimax.io/
MINIMAX_API_KEY=

# Qwen - https://dashscope.console.aliyun.com/
QWEN_API_KEY=

# GLM - https://open.bigmodel.cn/
GLM_API_KEY=
```

### 4.2 填入 API Keys

编辑 `.env` 文件，填入你的 API Keys：

```bash
# GLM 5.0
GLM_API_KEY=your_glm_api_key_here

# MiniMax M2.5
MINIMAX_API_KEY=your_minimax_api_key_here
```

---

## 五、完整使用流程

### 步骤 1：安装

```bash
# 选择一种方式
npx @andyzheung/ccmr init
# 或
npm install -g @andyzheung/ccmr && ccmr-plus init
```

### 步骤 2：配置 API Keys

编辑 `.env` 文件：

```bash
GLM_API_KEY=your_key
MINIMAX_API_KEY=your_key
```

### 步骤 3：启动网关

```bash
ccmr-plus start
```

**输出示例：**
```
============================================================
  Claude Code Model Router
============================================================

Gateway running at: http://localhost:8080
Default model: deepseek

Available models:
  - deepseek: DeepSeek V3.2 [Ready]
  - glm: GLM 5.0 [Ready]
  - minimax: MiniMax M2.5 [Ready]

Press Ctrl+C to stop the gateway.
============================================================
```

### 步骤 4：启动 Claude Code

```bash
# 新开终端
ccmr-plus claude
```

### 步骤 5：切换模型

在 Claude Code 中：

```bash
/model glm-5           # GLM 5.0
/model minimax-m2.5    # MiniMax M2.5
```

---

## 六、快速测试命令

### 测试 GLM-5

```bash
# 确保 GLM_API_KEY 已设置
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "glm-5",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "你好，请用一句话介绍你自己"}]
  }'
```

### 测试 MiniMax M2.5

```bash
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "minimax-m2.5",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Hello, introduce yourself"}]'
  }'
```

---

## 七、故障排除

### 问题 1：命令找不到

```bash
# 错误
bash: ccmr-plus: command not found

# 解决
# 使用 npx 方式，或重新安装
npm install -g @andyzheung/ccmr
```

### 问题 2：版本不是 1.2.0

```bash
# 检查当前版本
ccmr-plus --version

# 强制更新到最新版本
npm update -g @andyzheung/ccmr

# 或清除缓存后重装
npm cache clean --force
npm install -g @andyzheung/ccmr@latest
```

### 问题 3：API Key 无效

```bash
# 1. 检查 .env 文件是否存在
ls -la .env

# 2. 检查环境变量是否加载
echo $GLM_API_KEY

# 3. 重新初始化
rm .env
ccmr-plus init
```

### 问题 4：端口被占用

```bash
# 使用其他端口
ccmr-plus start --port 9000

# 或查看占用进程
lsof -i :8080
```

---

## 八、npm 验证命令

### 查看包信息

```bash
# 查看包信息
npm view @andyzheung/ccmr

# 查看所有版本
npm view @andyzheung/ccmr versions

# 查看最新版本
npm view @andyzheung/ccmr version
```

### 验证下载

```bash
# 下载但不安装（测试包是否存在）
npm pack --dry-run @andyzheung/ccmr

# 实际下载 tarball
npm pack @andyzheung/ccmr
# 会生成: andyzheung-ccmr-1.2.0.tgz
```

---

## 九、卸载

```bash
# 全局卸载
npm uninstall -g @andyzheung/ccmr

# 删除配置（可选）
rm -rf ~/.ccmr-plus
rm .env
rm .ccmr-plus.yaml
```

---

## 十、获取帮助

```bash
# 查看帮助
ccmr-plus --help

# 查看可用命令
ccmr-plus --help

# 查看特定命令帮助
ccmr-plus start --help
ccmr-plus claude --help
```

---

## 十一、发布后验证清单

发布后请按以下步骤验证：

- [ ] `npm view @andyzheung/ccmr` 能看到包信息
- [ ] `npx @andyzheung/ccmr --version` 显示 1.2.0
- [ ] `npm install -g @andyzheung/ccmr` 安装成功
- [ ] `ccmr-plus init` 生成配置文件
- [ ] `ccmr-plus start` 网关启动成功
- [ ] `ccmr-plus models` 显示模型列表
- [ ] API 调用测试通过
