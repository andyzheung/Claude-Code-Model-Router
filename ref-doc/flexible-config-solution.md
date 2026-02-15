# 灵活配置方案设计

## 1. 问题分析

### 当前问题

1. **硬编码配置**: 每次更新模型都需要修改 `src/config.ts` 中的 `DEFAULT_CONFIG`
2. **发布周期长**: 需要重新构建、发布 npm 包
3. **用户等待时间长**: 新模型发布后，用户需要等待官方更新才能使用
4. **维护成本高**: 每次更新都要修改多处代码

### 期望目标

- ✅ 用户无需修改代码即可添加新模型
- ✅ 支持通过配置文件动态添加模型
- ✅ 提供模型配置模板和验证
- ✅ 保持向后兼容性

---

## 2. 解决方案设计

### 2.1 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                      用户配置层                              │
│  ~/.claude-router/models.yaml (用户自定义模型)               │
│  项目根目录/models.yaml (项目特定模型)                        │
└─────────────────────────────────────────────────────────────┘
                            ↓ merge
┌─────────────────────────────────────────────────────────────┐
│                     内置模型库                                │
│  内置模型配置 (无需修改代码，通过外部文件维护)                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    配置加载器                                 │
│  ConfigManager.loadConfig()                                  │
│  - 支持多配置源                                              │
│  - 配置合并策略                                              │
│  - 配置验证                                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 实施方案

### 方案 A: 外置模型注册表（推荐）

#### 3.1 目录结构调整

```
claude-code-model-router/
├── src/
│   ├── config.ts           # 核心配置逻辑
│   └── ...
├── models/                 # 新增：模型注册表目录
│   ├── index.yaml          # 主索引（仅包含目录信息）
│   ├── builtin/            # 内置模型（随 npm 包发布）
│   │   ├── deepseek.yaml
│   │   ├── glm.yaml
│   │   ├── minimax.yaml
│   │   ├── kimi.yaml
│   │   └── qwen.yaml
│   └── custom/             # 用户自定义模型（可选）
│       └── .gitkeep
├── ~/.claude-router/       # 用户配置目录
│   └── models/             # 用户自定义模型
│       └── my-custom-model.yaml
```

#### 3.2 模型配置文件格式

**models/builtin/glm-5.yaml**
```yaml
# GLM 5.0 模型配置
display_name: "GLM 5.0"
provider: zhipu
model_id: glm-5
base_url: https://open.bigmodel.cn/api/anthropic
api_key_env: GLM_API_KEY
auth_header: x-api-key
supports_streaming: true
supports_tools: true
max_tokens: 128000
context_window: 200000

# 版本别名
aliases:
  - glm-5.0
  - zhipu
```

**models/builtin/minimax-m2.5.yaml**
```yaml
# MiniMax M2.5 模型配置
display_name: "MiniMax M2.5"
provider: minimax
model_id: MiniMax-M2.5
base_url: https://api.minimaxi.com/anthropic
api_key_env: MINIMAX_API_KEY
auth_header: x-api-key
supports_streaming: true
supports_tools: true
max_tokens: 128000
context_window: 200000

# 版本别名
aliases:
  - minimax-m2.5
  - minimax-m2.1
  - mm
```

#### 3.3 核心代码修改

**src/config.ts - 模型发现器**
```typescript
import fs from 'node:fs';
import path from 'node:path';

export class ModelRegistry {
  private models: Map<string, ModelConfig> = new Map();
  private aliases: Map<string, string> = new Map();

  constructor() {
    this.loadBuiltinModels();
    this.loadUserModels();
  }

  private loadBuiltinModels(): void {
    const builtinDir = path.join(__dirname, '../models/builtin');
    this.loadModelsFromDirectory(builtinDir);
  }

  private loadUserModels(): void {
    const userDirs = [
      path.join(process.cwd(), 'models'),
      path.join(os.homedir(), '.claude-router/models'),
    ];

    for (const dir of userDirs) {
      if (fs.existsSync(dir)) {
        this.loadModelsFromDirectory(dir);
      }
    }
  }

  private loadModelsFromDirectory(dir: string): void {
    const files = fs.readdirSync(dir).filter(f => f.endsWith('.yaml'));

    for (const file of files) {
      const filePath = path.join(dir, file);
      const content = fs.readFileSync(filePath, 'utf-8');
      const model = yaml.load(content) as ModelConfig & { aliases?: string[] };

      // 使用文件名（不含扩展名）作为模型 key
      const modelKey = path.basename(file, '.yaml');

      this.models.set(modelKey, {
        display_name: model.display_name,
        provider: model.provider,
        model_id: model.model_id,
        base_url: model.base_url,
        api_key_env: model.api_key_env,
        auth_header: model.auth_header,
        supports_streaming: model.supports_streaming ?? true,
        supports_tools: model.supports_tools ?? true,
        max_tokens: model.max_tokens,
        context_window: model.context_window,
      });

      // 注册别名
      if (model.aliases) {
        for (const alias of model.aliases) {
          this.aliases.set(alias, modelKey);
        }
      }

      // 模型名称本身也是别名
      this.aliases.set(modelKey, modelKey);
    }
  }

  getModel(name: string): ModelConfig | undefined {
    const resolved = this.aliases.get(name);
    return resolved ? this.models.get(resolved) : undefined;
  }

  listModels(): Record<string, ModelConfig> {
    return Object.fromEntries(this.models);
  }
}
```

#### 3.4 CLI 新增命令

```bash
# 列出所有可用模型
ccmr list-models

# 添加新模型（交互式）
ccmr add-model

# 验证模型配置
ccmr validate-model models/custom/my-model.yaml

# 删除用户模型
ccmr remove-model my-model
```

---

### 方案 B: 配置继承与模板（高级）

#### 3.5 模板机制

**models/templates/provider-template.yaml**
```yaml
# 提供商通用配置模板
{{model_id}}:
  display_name: "{{display_name}}"
  provider: "{{provider}}"
  model_id: "{{model_id}}"
  base_url: "{{base_url}}"
  api_key_env: "{{api_key_env}}"
  auth_header: x-api-key
  supports_streaming: true
  supports_tools: true
  max_tokens: {{max_tokens}}
  context_window: {{context_window}}
```

#### 3.6 配置验证

**src/config-validator.ts**
```typescript
import { z } from 'zod';

export const ModelConfigSchema = z.object({
  display_name: z.string().min(1),
  provider: z.string().min(1),
  model_id: z.string().min(1),
  base_url: z.string().url(),
  api_key_env: z.string().regex(/^[A-Z_]+$/),
  auth_header: z.string().default('x-api-key'),
  supports_streaming: z.boolean().default(true),
  supports_tools: z.boolean().default(true),
  max_tokens: z.number().positive().optional(),
  context_window: z.number().positive().optional(),
  aliases: z.array(z.string()).optional(),
});

export function validateModelConfig(config: unknown): {
  valid: boolean;
  errors?: string[];
} {
  const result = ModelConfigSchema.safeParse(config);
  return {
    valid: result.success,
    errors: result.success ? undefined : result.error.errors.map(e => e.message),
  };
}
```

---

## 4. 配置加载优先级

```
1. 用户目录模型 (~/.claude-router/models/*.yaml)  ← 最高优先级
2. 项目目录模型 (./models/*.yaml)
3. 内置模型库 (dist/models/builtin/*.yaml)        ← 最低优先级
```

**合并规则:**
- 同名模型：高优先级覆盖低优先级
- 别名：所有别名合并，冲突时高优先级获胜
- 配置：浅合并，非嵌套结构

---

## 5. 使用示例

### 5.1 用户添加自定义模型

**方法一：创建配置文件**
```bash
# 1. 创建用户模型目录
mkdir -p ~/.claude-router/models

# 2. 创建模型配置文件
cat > ~/.claude-router/models/my-provider.yaml << 'EOF'
display_name: "My Provider Model"
provider: myprovider
model_id: my-model-v1
base_url: https://api.myprovider.com/v1
api_key_env: MY_PROVIDER_API_KEY
auth_header: Authorization
max_tokens: 4096
context_window: 32000
aliases:
  - my-model
  - mm
EOF

# 3. 重启网关
ccmr start

# 4. 使用新模型
/model my-model
```

**方法二：使用 CLI 命令（交互式）**
```bash
ccmr add-model

? 模型名称（短名称）: myprovider
? 显示名称: My Provider Model
? 提供商标识: myprovider
? Model ID: my-model-v1
? API 端点: https://api.myprovider.com/v1
? API Key 环境变量: MY_PROVIDER_API_KEY
? 认证头名称: Authorization
? 最大 tokens: 4096
? 上下文窗口: 32000
? 添加别名（用逗号分隔）: my-model,mm

✅ 模型已添加到 ~/.claude-router/models/myprovider.yaml
```

### 5.2 使用特定版本

```bash
# 使用短名称（默认最新）
/model glm

# 使用版本别名
/model glm-5
/model minimax-m2.5

# 用户自定义模型
/model my-model
```

---

## 6. 迁移计划

### 阶段 1: 准备工作
1. 创建 `models/` 目录结构
2. 将现有模型配置迁移到独立 YAML 文件
3. 保持现有 `DEFAULT_CONFIG` 作为 fallback

### 阶段 2: 功能开发
1. 实现 `ModelRegistry` 类
2. 实现模型发现和加载逻辑
3. 添加配置验证功能

### 阶段 3: CLI 增强
1. 实现 `ccmr add-model` 命令
2. 实现 `ccmr list-models` 命令
3. 实现 `ccmr validate-model` 命令

### 阶段 4: 文档和测试
1. 编写用户文档
2. 添加集成测试
3. 发布新版本

---

## 7. 向后兼容性

**完全向后兼容：**

- 现有配置文件 `models.yaml` 仍然有效
- 环境变量配置方式不变
- 使用短名称切换模型的方式不变
- 旧版本无需迁移即可继续使用

---

## 8. 文件清单

### 新增文件

| 文件 | 说明 |
|------|------|
| `src/model-registry.ts` | 模型注册表核心逻辑 |
| `src/config-validator.ts` | 配置验证器 |
| `src/commands/add-model.ts` | add-model 命令 |
| `src/commands/list-models.ts` | list-models 命令 |
| `src/commands/validate-model.ts` | validate-model 命令 |
| `models/index.yaml` | 模型索引（可选） |
| `models/builtin/*.yaml` | 内置模型配置文件 |
| `models/templates/*.yaml` | 模型配置模板 |

### 修改文件

| 文件 | 修改内容 |
|------|----------|
| `src/config.ts` | 集成 ModelRegistry |
| `src/cli.ts` | 添加新命令 |
| `README.md` | 添加自定义模型文档 |
| `.gitignore` | 忽略用户自定义模型 |

---

## 9. 优势对比

| 特性 | 当前方案 | 新方案 |
|------|----------|--------|
| 添加新模型 | 需要修改代码 | 只需添加 YAML 文件 |
| 发布周期 | 等待 npm 更新 | 用户自定义立即生效 |
| 维护成本 | 高（每次更新改代码） | 低（只需更新 YAML） |
| 灵活性 | 低 | 高 |
| 向后兼容 | 完全兼容 | 完全兼容 |
| 配置验证 | 无 | 有（Zod schema） |

---

## 10. 开发工作量估算

| 任务 | 预计工时 |
|------|----------|
| 创建目录结构和模型文件 | 2h |
| 实现 ModelRegistry 类 | 4h |
| 实现配置验证 | 2h |
| 实现 CLI 命令 | 4h |
| 编写文档 | 2h |
| 测试和调试 | 4h |
| **总计** | **18h** |

---

## 11. 后续增强方向

1. **模型市场**: 社区贡献模型配置的集中仓库
2. **配置热更新**: 无需重启即可加载新模型
3. **模型测试**: 自动测试新模型 API 兼容性
4. **版本管理**: 支持同一提供商的多个版本
5. **Web UI**: 可视化配置管理界面
