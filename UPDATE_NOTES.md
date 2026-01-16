# 更新说明 (v1.0.1)

## 主要变更

### 1. ✅ 修复 settings.json 结构
**问题**: 之前的实现更新的是顶层的 `apiUrl` 和 `apiKey` 字段，但实际 Claude Code 使用的是嵌套的 `env` 对象。

**修复内容**:
- 现在正确更新 `env.ANTHROPIC_BASE_URL`
- 现在正确更新 `env.ANTHROPIC_AUTH_TOKEN`
- 保留其他配置项（如 `enabledPlugins`）

**实际的 settings.json 格式**:
```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your-api-key",
    "ANTHROPIC_BASE_URL": "https://api.url.com"
  },
  "enabledPlugins": {
    "document-skills@anthropic-agent-skills": true,
    "code-review@claude-plugins-official": true,
    "superpowers@superpowers-marketplace": true
  },
  "apiUrl": "",
  "apiKey": ""
}
```

### 2. ✅ 移除 API Key 格式限制
**问题**: 之前要求 API key 必须以 `sk-ant-` 开头，不支持其他 API 提供商。

**修复内容**:
- 移除 `sk-ant-` 前缀要求
- 最小长度从 20 字符降低到 10 字符
- 支持任意格式的 API key

**示例**: 现在可以使用以下格式的 key：
- `sk-ant-xxxxx` (Anthropic 官方)
- `cr_xxxxx` (其他提供商)
- 任何自定义格式

## 修改的文件

### 核心代码
1. **src/core/config.js**
   - `getApiConfig()` - 从嵌套的 env 对象读取配置
   - `updateApiConfig()` - 更新嵌套的 env 对象
   - `createSettings()` - 创建正确的初始结构

2. **src/core/validator.js**
   - `validateApiKey()` - 放宽验证规则

### 文档
3. **DESIGN_SPEC.md** - 更新验证规则和示例
4. **README.md** - 更新 settings.json 示例
5. **IMPLEMENTATION.md** - 更新数据结构说明
6. **CHANGELOG.md** - 记录变更历史
7. **package.json** - 版本升级到 1.0.1

## 使用示例

```bash
# 现在可以添加任意格式的 API key
ccm add aihezu https://cn.aihezu.dev/api cr_b31cbdea1b3fda0a5bfcd7ae11f3c6e5b01b5bc9702a394c522c45a54a4fc886

# 切换配置
ccm use aihezu

# 查看当前配置
ccm show
```

## 验证测试

建议进行以下测试：
1. [ ] `ccm init` - 初始化
2. [ ] `ccm add` - 添加新配置（使用非 sk-ant- 格式的 key）
3. [ ] `ccm use` - 切换配置
4. [ ] 检查 settings.json 文件是否正确更新 env 字段
5. [ ] 验证其他配置（如 enabledPlugins）是否保留

## 向后兼容性

- ✅ 如果 settings.json 不存在，会创建正确格式的文件
- ✅ 如果 settings.json 已存在但缺少 env 对象，会自动创建
- ✅ 保留所有其他配置项，不会覆盖

---

**更新日期**: 2026-01-16
**版本**: 1.0.0 → 1.0.1

---

## 包名更新记录

### 最终包名: @haiyangj/ccs

**发布命名**: `@haiyangj/ccs` (Haiyangj's Claude Config Switcher)

**原因**: 原始包名 `claude-config-manager` 已被其他开发者占用

**命名历史**:
1. ❌ `claude-config-manager` - 已被占用
2. ❌ `claude-profile-switcher` - 已被占用
3. ✅ `@haiyangj/ccs` - 最终选择（作用域包）

**安装方式**:
```bash
# 全局安装（推荐）
npm install -g @haiyangj/ccs

# 从源码安装
git clone https://github.com/haiyangj/ccs.git
cd ccs
npm install
npm link
```

**作用域包的优势**:
- ✅ 不会与他人的包冲突
- ✅ 明确的包归属权
- ✅ 可以发布为私有包（免费）
- ✅ 更好的组织结构

