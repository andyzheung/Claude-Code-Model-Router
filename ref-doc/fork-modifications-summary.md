# CCMR-Plus 修改总结 v1.2.0

## 一、项目重命名与隔离

### 1.1 npm 包信息

| 项目 | 原版 | CCMR-Plus |
|------|------|-----------|
| 包名 | `claude-code-model-router` | `@andyzheung/ccmr` |
| CLI 命令 | `ccmr` | `ccmr-plus` |
| 作者 | luwill | andyzheung |
| 仓库 | github.com/luwill/... | github.com/andyzheung/... |

### 1.2 配置隔离

| 配置项 | 原版 | CCMR-Plus |
|--------|------|-----------|
| 配置目录 | `~/.claude-gateway/` | `~/.ccmr-plus/` |
| 配置文件 | `.claude-router.yaml` | `.ccmr-plus.yaml` |

---

## 二、代码修改清单

### 2.1 package.json

```diff
{
-  "name": "claude-code-model-router",
+  "name": "@andyzheung/ccmr",
   "version": "1.2.0",
+  "description": "... (Forked with GLM-5 and MiniMax M2.5 support)",
   "bin": {
-    "ccmr": "dist/cli.js",
+    "ccmr-plus": "dist/cli.js",
     "claude-code-model-router": "dist/cli.js"
   },
   "keywords": [
     ...
+    "glm",
+    "glm-5",
+    "minimax",
+    "minimax-m2.5",
   ],
-  "author": "luwill",
+  "author": "andyzheung",
   "repository": {
-    "url": "https://github.com/luwill/Claude-Code-Model-Router.git"
+    "url": "https://github.com/andyzheung/Claude-Code-Model-Router.git"
   },
   ...
}
```

### 2.2 src/cli.ts

```diff
   const env = {
     ...process.env,
-    CLAUDE_CONFIG_DIR: path.join(homeDir, '.claude-gateway'),
+    CLAUDE_CONFIG_DIR: path.join(homeDir, '.ccmr-plus'),
     ANTHROPIC_BASE_URL: `http://localhost:${gatewayPort}`,
   };
```

### 2.3 src/config.ts

```diff
   const possiblePaths = [
     configPath,
     path.join(process.cwd(), 'models.yaml'),
     path.join(process.cwd(), 'config', 'models.yaml'),
-    path.join(process.cwd(), '.claude-router.yaml'),
+    path.join(process.cwd(), '.ccmr-plus.yaml'),
   ].filter(Boolean) as string[];
```

```diff
 export function generateConfigFile(): string {
-  return `# Claude Code Model Router Configuration
-# Place this file as models.yaml or .claude-router.yaml in your project root
+  return `# CCMR-Plus Configuration
+# Place this file as models.yaml or .ccmr-plus.yaml in your project root
```

### 2.4 README.md

全部更新为：
- `npx @andyzheung/ccmr` 替代 `npx claude-code-model-router`
- `ccmr-plus` 替代 `ccmr`
- `~/.ccmr-plus/` 替代 `~/.claude-gateway/`
- `.ccmr-plus.yaml` 替代 `.claude-router.yaml`

---

## 三、模型更新

### 3.1 GLM-5

```diff
   glm: {
-    display_name: 'GLM 4.7',
+    display_name: 'GLM 5.0',
     provider: 'zhipu',
-    model_id: 'GLM-4.7',
+    model_id: 'glm-5',  // 官方使用小写
     ...
   }
```

### 3.2 MiniMax M2.5

```diff
   minimax: {
-    display_name: 'MiniMax M2.1',
+    display_name: 'MiniMax M2.5',
     provider: 'minimax',
-    model_id: 'MiniMax-M2.1',
+    model_id: 'MiniMax-M2.5',
     ...
   }
```

---

## 四、修改文件汇总

```
modified:   package.json
modified:   src/cli.ts
modified:   src/config.ts
modified:   README.md
```

---

## 五、验证结果

### 5.1 构建测试

```bash
$ npm run build
> @andyzheung/ccmr@1.2.0 build
> tsc

✅ 构建成功
```

### 5.2 包名验证

```bash
$ npm pack --dry-run
npm notice
npm notice 📦 @andyzheung/ccmr@1.2.0
npm notice === Tarball Contents ===
...
```

---

## 六、发布步骤

### 6.1 准备

```bash
# 确认在 dev 分支
git branch

# 查看修改
git status
```

### 6.2 提交

```bash
git add .
git commit -m "Release v1.2.0: CCMR-Plus with GLM-5 and MiniMax M2.5"
git push origin dev
```

### 6.3 发布 npm

```bash
# 构建并发布
npm run build
npm publish --access public
```

---

## 七、用户使用

### 安装

```bash
# npx 方式
npx @andyzheung/ccmr init

# 全局安装
npm install -g @andyzheung/ccmr
```

### 使用

```bash
ccmr-plus init
ccmr-plus start
ccmr-plus claude
```

### 配置文件位置

```
~/.ccmr-plus/          ← 配置目录
~/.ccmr-plus/settings.json  ← Claude Code 配置
.env                   ← API Keys
.ccmr-plus.yaml        ← 模型配置（可选）
```

---

## 八、与原版共存

```bash
# 可以同时安装两个版本

# 原版
ccmr start
ccmr claude

# CCMR-Plus
ccmr-plus start
ccmr-plus claude
```

**配置完全隔离，互不干扰！**

---

## 九、参考文档

| 文档 | 说明 |
|------|------|
| `ref-doc/release-guide-ccmr-plus.md` | 完整发布指南 |
| `ref-doc/changelog-v1.2.0.md` | 详细修改记录 |
| `ref-doc/new-model-support-analysis.md` | 模型更新分析 |
