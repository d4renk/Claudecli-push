# Claude Notify

![License](https://img.shields.io/badge/license-MIT-blue.svg)

为 [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) 提供智能任务完成通知的 Hook 工具。当 Claude Code 执行长时间任务或需要用户确认时，自动通过你喜欢的渠道推送通知。

## ✨ 功能特性

- **全场景监听**：
  - `UserPromptSubmit` - 记录任务开始
  - `Notification` - 提示需要用户确认/操作
  - `Stop` - 自动检测任务成功或失败状态
  - `PreCompact` - 上下文过长警告
- **智能通知**：
  - 自动计算任务耗时
  - 仅在任务超过设定阈值（默认 3 分钟）时推送，避免打扰
- **多平台支持**：支持 20+ 种推送服务，包括：
  - **移动端**：Bark (iOS)、钉钉、飞书、企业微信、Telegram
  - **桌面端**：Server 酱、PushDeer、PushPlus
  - **其他**：SMTP 邮件、自定义 Webhook 等

---

## 🚀 快速开始

### 前置要求

- **操作系统**：Linux 或 WSL2 (Windows Subsystem for Linux)
- **环境**：Node.js 14+

### 安装步骤

1. **克隆项目**
   ```bash
   git clone https://github.com/d4renk/claude-notify.git
   cd claude-notify
   ```

2. **运行安装脚本**
   此脚本会自动配置 Claude Code 的 Hooks 设置 (`~/.claude/settings.json`)。
   ```bash
   bash install.sh
   ```

3. **配置环境变量**
   在您的 shell 配置文件（如 `~/.bashrc` 或 `~/.zshrc`）中添加推送服务的配置。

   *示例（Bark iOS）：*
   ```bash
   export BARK_PUSH=https://api.day.app/YOUR_KEY
   ```

4. **验证安装**
   ```bash
   # 1. 加载新的环境变量
   source ~/.bashrc  # 或 source ~/.zshrc

   # 2. 发送测试通知
   node send-notify.js "测试标题" "恭喜！Claude Notify 配置成功。"
   ```

---

## ⚙️ 配置说明

所有配置均通过环境变量进行管理。

### 1. 推送服务配置（必选其一）

请至少配置一种推送服务，否则无法接收通知。

#### 推荐服务

| 服务名称 | 环境变量配置示例 | 获取方式 |
| :--- | :--- | :--- |
| **Bark (iOS)** | `export BARK_PUSH=https://api.day.app/YOUR_KEY` | [App Store](https://apps.apple.com/cn/app/bark-customed-notifications/id1403753865) |
| **Server 酱** | `export PUSH_KEY=YOUR_SERVER_CHAN_KEY` | [官网](https://sct.ftqq.com) |
| **Telegram** | `export TG_BOT_TOKEN=YOUR_TOKEN`<br>`export TG_USER_ID=YOUR_ID` | [@BotFather](https://t.me/botfather) |
| **钉钉机器人** | `export DD_BOT_TOKEN=YOUR_TOKEN`<br>`export DD_BOT_SECRET=YOUR_SECRET` | 群设置 → 智能群助手 |
| **飞书机器人** | `export FSKEY=YOUR_KEY` | 飞书开放平台 |

<details>
<summary><strong>点击查看其他 15+ 种支持的服务</strong></summary>

| 服务名称 | 必需变量 | 可选变量 |
| :--- | :--- | :--- |
| **PushDeer** | `DEER_KEY` | `DEER_URL` |
| **PushPlus** | `PUSH_PLUS_TOKEN` | `PUSH_PLUS_USER` 等 |
| **企业微信** | `QYWX_KEY` (机器人) 或 `QYWX_AM` (应用) | - |
| **WxPusher** | `WXPUSHER_APP_TOKEN` | `WXPUSHER_UIDS` |
| **Gotify** | `GOTIFY_URL`, `GOTIFY_TOKEN` | `GOTIFY_PRIORITY` |
| **SMTP 邮件** | `SMTP_SERVER`, `SMTP_EMAIL`, `SMTP_PASSWORD` | - |
| **自定义 Webhook** | `WEBHOOK_URL` | `WEBHOOK_METHOD`, `WEBHOOK_BODY` |

*(更多详细配置参数请参考项目根目录下的 `.env.example` 文件)*
</details>

### 2. 高级选项（可选）

您可以根据需要调整以下默认行为：

| 环境变量 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `CLAUDE_NOTIFY_LONG_TASK_SECONDS` | `180` | **长任务阈值 (秒)**。<br>只有耗时超过此时间的任务才会触发“任务完成”通知。 |
| `CLAUDE_NOTIFY_TITLE_PREFIX` | `"Claude Code"` | **通知标题前缀**。 |
| `HITOKOTO` | `false` | **启用一言**。<br>设为 `true` 可在通知中附加一句随机名言。 |
| `CLAUDE_NOTIFY_DEBUG` | `false` | **调试模式**。<br>设为 `true` 会将详细日志写入 `hook.log`。 |
| `CLAUDE_NOTIFY_STATE_DIR` | `~/.claude/claude-notify-state` | **状态存储目录**。 |

---

## 🔍 工作原理

Claude Notify 作为一个 Hook 挂载在 Claude Code 的生命周期中：

1.  **开始 (`UserPromptSubmit`)**：用户按下回车，脚本记录当前时间戳。
2.  **执行中**：Claude Code 执行任务...
3.  **结束/中断 (`Stop` / `Notification`)**：
    *   脚本计算 `(当前时间 - 开始时间)`。
    *   如果耗时 > `CLAUDE_NOTIFY_LONG_TASK_SECONDS` (默认 180秒)，则推送通知。
    *   如果遇到需要用户确认的 `Notification` 事件，无论耗时多久，立即推送。

---

## 📂 文件结构

```text
claude-notify/
├── install.sh              # 自动安装脚本
├── uninstall.sh            # 卸载脚本
├── claude_notify_hook.js   # 核心 Hook 逻辑
├── notify.js               # 推送服务适配层
├── send-notify.js          # 命令行测试工具
└── README.md               # 说明文档
```

## 🗑️ 卸载

如果需要移除 Claude Notify，只需运行卸载脚本：

```bash
bash uninstall.sh
```
此操作会清理 `~/.claude/settings.json` 中的相关 Hook 配置。

---

## 🔗 相关文档

- [Claude Code Hooks 官方指南](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/hooks) (外部链接)
- [本地 Hooks 完整指南](./Claude%20Code%20Hooks%20完整指南.md)
- [更新日志](./CHANGELOG.md)