# OpenClaw 配置备份仓库

本仓库用于安全地备份 OpenClaw 配置到 GitHub，敏感凭证已通过环境变量分离。

---

## 目录

- [包含的文件](#包含的文件)
- [排除的敏感文件](#排除的敏感文件)
- [快速恢复指南](#快速恢复指南)
- [详细恢复步骤](#详细恢复步骤)
- [配置说明](#配置说明)
- [故障排查](#故障排查)
- [安全须知](#安全须知)

---

## 包含的文件

| 文件/目录 | 说明 |
|-----------|------|
| `CLAUDE.md` | 项目文档和架构指南 |
| `extensions/feishu-openclaw-plugin/` | 飞书/Lark 插件源代码 |
| `.gitignore` | Git 忽略规则 |
| `.env.example` | 环境变量模板 |
| `openclaw.example.json` | 配置模板（不含敏感信息） |
| `README.md` | 本说明文档 |

## 排除的敏感文件

以下文件包含敏感信息，已被 `.gitignore` 排除：

| 文件/目录 | 原因 |
|-----------|------|
| `openclaw.json` | 包含 API Keys、App Secret、Auth Token |
| `credentials/` | 凭证文件 |
| `devices/` | 设备身份数据 |
| `identity/` | 用户身份数据 |
| `memory/` | 会话记忆数据 |
| `logs/` | 运行时日志 |
| `agents/` | 包含凭证的 Agent 配置 |
| `.claude/` | Claude Code 本地设置 |
| `workspace/` | 有独立的 git 仓库 |

---

## 快速恢复指南

```bash
# 1. 安装 OpenClaw
npm install -g openclaw

# 2. Clone 仓库到 ~/.openclaw/
git clone <your-repo-url> ~/.openclaw

# 3. 创建 .env 文件
cd ~/.openclaw
cp .env.example .env
# 编辑 .env 填入你的 API Keys

# 4. 创建 openclaw.json
cp openclaw.example.json openclaw.json

# 5. 启动网关
openclaw gateway run
```

---

## 详细恢复步骤

### 步骤 1: 准备工作

确保已安装 Node.js 和 OpenClaw:

```bash
# 检查 Node.js
node --version

# 安装 OpenClaw (如果未安装)
npm install -g openclaw

# 检查 OpenClaw 版本
openclaw --version
```

### 步骤 2: Clone 仓库

```bash
# 如果 ~/.openclaw/ 不存在
git clone <your-repo-url> ~/.openclaw

# 如果要合并到现有目录
cd ~/.openclaw
git init
git remote add origin <your-repo-url>
git pull origin main
```

### 步骤 3: 配置环境变量

复制并编辑 `.env` 文件:

```bash
cp .env.example .env
```

编辑 `.env` 填入真实的 API Keys:

```bash
# ===========================================
# 模型提供商
# ===========================================

# 阿里云 DashScope - 用于 qwen3.5-plus 模型
DASHSCOPE_API_KEY="sk-your-actual-api-key-here"

# ===========================================
# Web 搜索
# ===========================================

# Kimi Web Search API Key
KIMI_SEARCH_API_KEY="your-kimi-api-key-here"

# ===========================================
# 飞书/Lark 频道
# ===========================================

# 飞书 App Secret (用于 WebSocket 连接)
FEISHU_APP_SECRET="your-actual-app-secret-here"

# ===========================================
# 网关认证
# ===========================================

# 网关 token 用于本地认证
OPENCLAW_GATEWAY_TOKEN="your-gateway-token-here"
```

### 步骤 4: 创建主配置文件

```bash
cp openclaw.example.json openclaw.json
```

编辑 `openclaw.json` 需要修改的字段:

| 字段 | 位置 | 说明 |
|------|------|------|
| `channels.feishu.appId` | 第 86 行 | 你的飞书应用 ID |
| `agents.defaults.workspace` | 第 35 行 | 工作区路径 (如有需要) |
| `plugins.load.paths` | 第 127 行 | 插件加载路径 |

其他敏感值已通过环境变量引用，如 `${DASHSCOPE_API_KEY}`，系统会自动从 `.env` 读取。

### 步骤 5: 验证安装

```bash
# 检查配置文件是否有效
openclaw config get channels.feishu

# 启动网关
openclaw gateway run
```

### 步骤 6: 飞书配对 (如使用飞书频道)

1. 在飞书聊天中发送消息给机器人
2. 机器人会回复一个配对码
3. 在终端执行:
   ```bash
   openclaw pairing approve feishu <配对码>
   ```

---

## 配置说明

### 环境变量说明

| 变量名 | 用途 | 获取方式 |
|--------|------|----------|
| `DASHSCOPE_API_KEY` | 阿里云通义千问 API | [阿里云控制台](https://dashscope.console.aliyun.com/) |
| `KIMI_SEARCH_API_KEY` | Kimi 搜索 API | Kimi 开放平台 |
| `FEISHU_APP_SECRET` | 飞书应用密钥 | [飞书开放平台](https://open.feishu.cn/) |
| `OPENCLAW_GATEWAY_TOKEN` | 网关认证 Token | 自动生成或自定义 |

### openclaw.json 关键配置

```json
{
  "models": {
    // AI 模型配置
  },
  "channels": {
    "feishu": {
      // 飞书频道配置
    }
  },
  "gateway": {
    // 网关服务配置
  },
  "plugins": {
    // 插件管理配置
  }
}
```

---

## 故障排查

### 常见问题

#### 1. 无法启动网关

```bash
# 检查端口是否被占用
lsof -i :36789

# 修改端口
openclaw config set gateway.port 36790
```

#### 2. 飞书连接失败

```bash
# 检查配置
openclaw config get channels.feishu

# 运行诊断
openclaw feishu-diagnose
```

#### 3. 环境变量未生效

```bash
# 检查 .env 文件格式
cat .env

# 确保没有空格，格式为 KEY="value"
# 重启网关
openclaw gateway run
```

### 日志位置

```bash
# 查看日志
ls -la ~/.openclaw/logs/
```

---

## 备份流程

### 创建备份

```bash
cd ~/.openclaw

# 更新 git 仓库
git add .
git commit -m "Backup: $(date +%Y-%m-%d)"

# 推送到 GitHub
git push origin main
```

### 自动备份

可以创建定时任务定期备份配置。

---

## 安全须知

⚠️ **重要安全警告**

| 操作 | 说明 |
|------|------|
| ❌ 不要提交 `.env` 到 GitHub | 包含真实 API Keys |
| ❌ 不要提交 `openclaw.json` 到 GitHub | 包含敏感配置 |
| ✅ 使用 `.env.example` 作为模板 | 不含真实凭证 |
| ✅ 定期轮换 API Keys | 提高安全性 |
| ✅ 使用私有仓库或公共仓库 | 本仓库设计为公共仓库安全 |

### 如果意外泄露

1. 立即在相应平台撤销 API Key
2. 生成新的 API Key
3. 更新 `.env` 文件
4. 如果是仓库泄露，考虑清理 git 历史

---

## 从备份恢复检查清单

- [ ] 已安装 OpenClaw
- [ ] 已 clone 仓库到 `~/.openclaw/`
- [ ] 已创建 `.env` 并填入 API Keys
- [ ] 已创建 `openclaw.json`
- [ ] 已更新飞书 `appId`
- [ ] `openclaw gateway run` 成功启动
- [ ] 飞书配对完成 (如使用)

---

## License

Same as OpenClaw project license.
