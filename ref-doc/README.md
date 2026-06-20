# CCMR-Plus 文档目录

本文档目录包含 CCMR-Plus 项目的所有相关文档。

---

## 📚 文档分类

### 🌟 用户文档

| 文档 | 说明 | 适用人群 |
|------|------|----------|
| **[user-guide.md](user-guide.md)** | 完整的用户使用指南 | 最终用户 |
| **[glm-5.2-usage-guide.md](glm-5.2-guide/glm-5.2-usage-guide.md)** | GLM-5.2 升级与使用指南 | 用户、维护者 |
| **[download-verification-guide.md](download-verification-guide.md)** | 下载、安装和验证指南 | 用户、开发者 |

### 🔧 开发者文档

| 文档 | 说明 | 适用人群 |
|------|------|----------|
| **[fork-modifications-summary.md](fork-modifications-summary.md)** | Fork 版本的修改总结 | 开发者 |
| **[new-model-support-analysis.md](new-model-support-analysis.md)** | GLM-5 和 MiniMax M2.5 支持分析 | 开发者 |
| **[flexible-config-solution.md](flexible-config-solution.md)** | 灵活配置方案设计 | 开发者 |
| **[npm-source-configuration.md](npm-source-configuration.md)** | npm 源配置与发布指南 | 开发者 |

### 📦 发布文档

| 文档 | 说明 | 适用人群 |
|------|------|----------|
| **[release-guide-ccmr-plus.md](release-guide-ccmr-plus.md)** | CCMR-Plus 发布指南 | 维护者 |
| **[changelog-v1.2.0.md](changelog-v1.2.0.md)** | v1.2.0 版本更新记录 | 维护者、用户 |

---

## 🚀 快速导航

### 我是用户，想使用 CCMR-Plus

👉 阅读 **[user-guide.md](user-guide.md)**

包含：
- 安装步骤
- 配置 API Keys
- 使用方法
- 常见问题

### 我是开发者，想了解修改内容

👉 阅读 **[fork-modifications-summary.md](fork-modifications-summary.md)**

包含：
- 与原版的区别
- 代码修改清单
- 配置隔离说明

### 我想发布新版本

👉 阅读 **[release-guide-ccmr-plus.md](release-guide-ccmr-plus.md)**

包含：
- 发布流程
- npm 源配置
- 发布检查清单

### 我想了解如何支持新模型

👉 阅读 **[new-model-support-analysis.md](new-model-support-analysis.md)** 和 **[flexible-config-solution.md](flexible-config-solution.md)**

---

## 📖 文档说明

### 用户指南重点内容

1. **安装方式**
   - npx 方式（推荐）
   - 全局安装方式

2. **配置 API Keys**
   - GLM 5.0
   - MiniMax M2.5
   - 其他模型

3. **使用方法**
   - 启动网关
   - 切换模型
   - 与原版共存

4. **端口配置** ⚠️
   - 默认端口：8080
   - 与原版同时使用需使用不同端口
   - 示例：`ccmr-plus start --port 8081`

### 与原版共存配置

```
原版 ccmr:
  命令: ccmr
  端口: 8080 (默认)
  配置: ~/.claude-gateway/

CCMR-Plus:
  命令: ccmr-plus
  端口: 8081 (需指定)
  配置: ~/.ccmr-plus/
```

**同时使用示例：**
```bash
# 终端 1：原版
ccmr start

# 终端 2：CCMR-Plus
ccmr-plus start --port 8081
ccmr-plus claude --gateway-port 8081
```

---

## 🔄 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.2.2 | 2025-02-15 | 修复 VERSION 硬编码问题，更新 CLI 命令为 ccmr-plus |
| v1.2.1 | 2025-02-15 | 移除冲突的 bin 命令 |
| v1.2.0 | 2025-02-15 | 更新 GLM 到 5.0，MiniMax 到 M2.5 |

---

## 📞 获取帮助

- **GitHub Issues**: https://github.com/andyzheung/Claude-Code-Model-Router/issues
- **npm 页面**: https://www.npmjs.com/package/@andyzheung/ccmr

---

## 🙏 致谢

Forked from [claude-code-model-router](https://github.com/luwill/Claude-Code-Model-Router)
