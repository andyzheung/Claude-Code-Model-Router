# npm 源配置与发布指南

## 一、npm 源说明

### 1.1 官方源 vs 镜像源

| 源 | 地址 | 用途 |
|------|------|------|
| **官方源** | `https://registry.npmjs.org/` | 发布包、获取最新版本 |
| **淘宝镜像** | `https://registry.npmmirror.com/` | 下载包（国内速度快） |

### 1.2 同步机制

```
官方源 ──发布──> npm registry
              │
              └── CDN 同步 ──> 淘宝镜像
```

**重要：**
- 必须发布到官方源
- 淘宝镜像会自动同步（通常几分钟）
- 用户可以从任一源下载

---

## 二、当前源配置检查

### 查看当前源

```bash
npm config get registry
```

**输出示例：**
```
https://registry.npmmirror.com/  # 淘宝镜像
# 或
https://registry.npmjs.org/      # 官方源
```

---

## 三、发布流程

### 3.1 切换到官方源（发布时必须）

```bash
# 临时切换（仅当前命令）
npm login --registry https://registry.npmjs.org/

# 或永久切换
npm config set registry https://registry.npmjs.org/
```

### 3.2 登录 npm

```bash
npm login
```

按提示输入：
- Username: npm 用户名
- Password: 密码（输入时不显示）
- Email: 邮箱
- OTP: 邮箱验证码（如果启用了双重验证）

### 3.3 验证登录

```bash
npm whoami
```

应该显示你的用户名。

### 3.4 发布包

```bash
# 构建
npm run build

# 发布（scoped 包必须加 --access public）
npm publish --access public
```

### 3.5 恢复镜像源（可选）

发布完成后，可以切换回淘宝镜像加速下载：

```bash
npm config set registry https://registry.npmmirror.com/
```

---

## 四、推荐配置：下载与发布分离

### 4.1 理想配置

```bash
# 设置默认下载源为淘宝镜像（快）
npm config set registry https://registry.npmmirror.com/

# 设置你的 scoped 包发布源为官方源
npm config set @andyzheung:registry https://registry.npmjs.org/
```

### 4.2 效果

```bash
# 下载其他包 → 使用淘宝镜像（快）
npm install express

# 下载自己的包 → 使用官方源
npm install @andyzheung/ccmr

# 发布自己的包 → 自动使用官方源
npm publish --access public
```

### 4.3 查看完整配置

```bash
npm config list
```

---

## 五、常见问题

### Q1: 发布时提示 404 错误

**原因：** scoped 包必须加 `--access public`

**解决：**
```bash
npm publish --access public
```

### Q2: 发布时提示 ENEEDAUTH

**原因：** 未登录或登录过期

**解决：**
```bash
npm login
```

### Q3: 淘宝镜像上找不到新发布的包

**原因：** 镜像同步需要时间

**解决：**
- 等待几分钟
- 或临时使用官方源下载：
  ```bash
  npm install @andyzheung/ccmr --registry https://registry.npmjs.org/
  ```

### Q4: 如何知道包是否已发布成功

**验证方法：**
```bash
# 查看包信息
npm view @andyzheung/ccmr

# 或访问官网
# https://www.npmjs.com/package/@andyzheung/ccmr
```

---

## 六、用户下载说明

### 6.1 发布后多久可以下载？

**答案：** 几分钟内

- 官方源：立即可用
- 淘宝镜像：通常 1-5 分钟同步完成

### 6.2 用户下载方式

```bash
# 方式一：npx（自动选择最快的源）
npx @andyzheung/ccmr init

# 方式二：官方源
npm install -g @andyzheung/ccmr --registry https://registry.npmjs.org/

# 方式三：淘宝镜像
npm install -g @andyzheung/ccmr --registry https://registry.npmmirror.com/
```

### 6.3 推荐用户配置

建议用户也配置源分离：

```bash
# 下载用淘宝镜像（快）
npm config set registry https://registry.npmmirror.com/

# 你的包自动从官方源获取（如果设置了 scope registry）
npm config set @andyzheung:registry https://registry.npmjs.org/
```

---

## 七、发布检查清单

- [ ] 切换到官方源：`npm config set registry https://registry.npmjs.org/`
- [ ] 登录 npm：`npm login`
- [ ] 验证登录：`npm whoami`
- [ ] 构建项目：`npm run build`
- [ ] 发布包：`npm publish --access public`
- [ ] 验证发布：`npm view @andyzheung/ccmr`
- [ ] 测试下载：`npx @andyzheung/ccmr --version`
- [ ] （可选）恢复镜像源：`npm config set registry https://registry.npmmirror.com/`

---

## 八、快速参考

### 发布流程（一键复制）

```bash
# 1. 切换到官方源
npm config set registry https://registry.npmjs.org/

# 2. 登录
npm login

# 3. 构建并发布
npm run build
npm publish --access public

# 4. 验证
npm view @andyzheung/ccmr

# 5. （可选）恢复镜像源
npm config set registry https://registry.npmmirror.com/
```

### 用户下载（一键复制）

```bash
# 推荐：使用 npx（自动选择源）
npx @andyzheung/ccmr init

# 或全局安装
npm install -g @andyzheung/ccmr

# 使用
ccmr-plus init
ccmr-plus start
```

---

## 九、相关文档

- npm 官方文档：https://docs.npmjs.com/
- 淘宝镜像：https://npmmirror.com/
- CCMR-Plus 下载指南：`ref-doc/download-verification-guide.md`
- CCMR-Plus 发布指南：`ref-doc/release-guide-ccmr-plus.md`
