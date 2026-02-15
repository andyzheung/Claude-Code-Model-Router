# v1.2.0 发布与使用指南

## 一、修改完整性检查

### ✅ 已完成的修改

| 文件 | 修改状态 | 说明 |
|------|----------|------|
| `src/config.ts` | ✅ 完成 | MiniMax M2.5, GLM-5 配置及别名 |
| `README.md` | ✅ 完成 | 模型列表、参数、示例、更新日志 |
| `package.json` | ✅ 完成 | 版本号 1.1.0 → 1.2.0 |
| `npm run build` | ✅ 成功 | TypeScript 编译通过 |

### 📊 修改统计

```
modified:   README.md        (+11, -6)
modified:   package.json     (+1, -1)
modified:   src/config.ts    (+6, -2)
new file:   ref-doc/
```

---

## 二、发布流程

### 方式一：发布到 npm（正式发布）

```bash
# 1. 确认当前在 dev 分支
git branch

# 2. 提交修改
git add src/config.ts README.md package.json
git commit -m "Release v1.2.0: Update GLM to 5.0 and MiniMax to M2.5

- Update MiniMax model_id to MiniMax-M2.5
- Update GLM model_id to glm-5 (official uses lowercase)
- Add version aliases: glm-5, minimax-m2.5
- Keep backward compatibility with old aliases
- Update documentation and changelog"

# 3. 推送到远程
git push origin dev

# 4. 合并到 main 分支
git checkout main
git merge dev
git push origin main

# 5. 创建发布标签
git tag -a v1.2.0 -m "Release v1.2.0: GLM 5.0 and MiniMax M2.5"
git push origin v1.2.0

# 6. 发布到 npm
npm publish

# 7. 创建 GitHub Release
# 访问: https://github.com/luwill/Claude-Code-Model-Router/releases/new
# 标签: v1.2.0
# 标题: Release v1.2.0 - GLM 5.0 & MiniMax M2.5
```

### 方式二：本地测试（发布前验证）

```bash
# 1. 本地全局链接
npm link

# 2. 验证版本
ccmr --version
# 预期输出包含 1.2.0

# 3. 查看可用模型
ccmr models

# 4. 启动网关测试
ccmr start
```

---

## 三、用户使用指南

### 3.1 安装/更新

```bash
# 使用 npx（无需安装）
npx claude-code-model-router start

# 全局安装
npm install -g claude-code-model-router@latest

# 更新到最新版本
npm update -g claude-code-model-router
```

### 3.2 配置 API Keys

编辑 `.env` 文件：

```bash
# GLM 5.0
GLM_API_KEY=your_glm_api_key

# MiniMax M2.5
MINIMAX_API_KEY=your_minimax_api_key
```

### 3.3 启动网关

```bash
# 使用默认端口 8080
npx claude-code-model-router start

# 或使用自定义端口
npx claude-code-model-router start --port 9000

# 使用 ccmr 命令（全局安装后）
ccmr start
```

### 3.4 在 Claude Code 中使用

```bash
# 启动 Claude Code（网关模式）
npx claude-code-model-router claude

# 或使用 ccmr 命令
ccmr claude
```

### 3.5 切换模型

在 Claude Code 中使用 `/model` 命令：

```bash
# 使用短名称（默认最新版本）
/model glm        # GLM 5.0
/model minimax    # MiniMax M2.5

# 使用版本别名（明确指定版本）
/model glm-5           # GLM 5.0
/model glm-4.7         # GLM 4.7（旧版本兼容）
/model minimax-m2.5    # MiniMax M2.5
/model minimax-m2.1    # MiniMax M2.1（旧版本兼容）
```

---

## 四、版本兼容性

### 向后兼容

✅ **完全向后兼容** - 旧版本用户无需修改配置

| 旧别名 | 新别名 | 说明 |
|--------|--------|------|
| `glm-4.7` | `glm-5` | 两者都可用 |
| `minimax-m2.1` | `minimax-m2.5` | 两者都可用 |

### 环境变量

环境变量名称保持不变：

```bash
GLM_API_KEY        # GLM 5.0 和 GLM 4.7 共用
MINIMAX_API_KEY    # MiniMax M2.5 和 M2.1 共用
```

---

## 五、快速测试

### 测试 GLM-5

```bash
# 1. 启动网关
ccmr start

# 2. 新终端测试 API
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $GLM_API_KEY" \
  -d '{
    "model": "glm-5",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Hello, what version are you?"}]
  }'
```

### 测试 MiniMax M2.5

```bash
curl -X POST http://localhost:8080/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $MINIMAX_API_KEY" \
  -d '{
    "model": "minimax-m2.5",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Hello, what version are you?"}]
  }'
```

---

## 六、更新日志（用于 GitHub Release）

```markdown
## Release v1.2.0 - GLM 5.0 & MiniMax M2.5

### 🎉 新功能
- 支持 **GLM 5.0** 模型（智谱 AI 最新版本）
- 支持 **MiniMax M2.5** 模型（MiniMax 最新版本）

### 📝 变更
- GLM model_id: `GLM-4.7` → `glm-5`（官方使用小写）
- MiniMax model_id: `MiniMax-M2.1` → `MiniMax-M2.5`

### ✨ 新增别名
- `/model glm-5` - 使用 GLM 5.0
- `/model glm-5.0` - 使用 GLM 5.0
- `/model minimax-m2.5` - 使用 MiniMax M2.5

### 🔄 向后兼容
- 保留 `glm-4.7` 和 `minimax-m2.1` 别名
- 环境变量名称保持不变

### 📚 文档
- [GLM 官方文档](https://docs.bigmodel.cn/cn/coding-plan/tool/claude)
- [MiniMax 官方文档](https://platform.minimaxi.com/docs/coding-plan/claude-code)
```

---

## 七、发布检查清单

- [x] 代码修改完成
- [x] 构建成功 (`npm run build`)
- [ ] Git 提交并推送到 dev
- [ ] 合并到 main 分支
- [ ] 创建 Git 标签 v1.2.0
- [ ] 发布到 npm
- [ ] 创建 GitHub Release
- [ ] 更新项目主页 README（如有）
- [ ] 通知用户更新

---

## 八、常见问题

### Q1: 更新后原有配置是否需要修改？

**A:** 不需要。环境变量和配置文件格式保持不变，直接更新即可。

### Q2: 如何确认使用了新版本？

**A:** 在 Claude Code 中输入 `/status` 查看当前模型信息。

### Q3: 旧版本别名是否仍然可用？

**A:** 是的。`glm-4.7` 和 `minimax-m2.1` 仍然可用，指向对应版本的模型配置。

### Q4: API Key 是否需要更换？

**A:** 不需要。`GLM_API_KEY` 和 `MINIMAX_API_KEY` 继续使用，但请确保你的账户支持新版本模型。

---

## 九、参考文档

- 详细修改记录: `ref-doc/changelog-v1.2.0.md`
- 官方文档分析: `ref-doc/new-model-support-analysis.md`
- 灵活配置方案: `ref-doc/flexible-config-solution.md`
