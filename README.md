# alipay-notify

**免费提供公网异步通知地址**，让本地开发环境也能实时接收支付宝异步通知（`notify_url`）。无需自建服务、无需公网 IP、无需内网穿透，注册即用。

```
支付宝 ──POST──► 云端中继 ──SSE──► 本地 CLI 实时拉取
```

## 这是什么

这是一个标准的 **AI Agent Skill**。

安装到任何支持 Skill 机制的 AI 编程工具后，Agent 会自动完成全部操作：注册 → 获取 notify_url → 实时监听 → 查看原始报文 → 本地验签。**你不需要手动输入任何命令。**

### 兼容的 AI 编程工具

| 工具 | 支持情况 |
|------|----------|
| Claude Code | ✅ 原生支持 |
| Cursor | ✅ 原生支持 |
| 其他支持 Skill / 自定义指令的 Agent | ✅ 只要能读取 SKILL.md 并执行终端命令即可 |

> 本 Skill 遵循标准的 SKILL.md 规范，不依赖特定工具的私有 API。任何能读取 Markdown 指令并执行 shell 命令的 AI Agent 均可使用。

## 安装

### Claude Code

```bash
# 在项目根目录
mkdir -p .claude/skills
git clone https://github.com/zhangke091/alipay-notify .claude/skills/alipay-notify
```

### Cursor

```bash
mkdir -p .cursor/skills
git clone https://github.com/zhangke091/alipay-notify .cursor/skills/alipay-notify
```

### 手动安装

将本仓库内容复制到你所用 AI 工具的 skills 目录即可，结构保持：

```
skills/alipay-notify/
├── SKILL.md
├── README.md
└── scripts/
    └── cli.py
```

## 前置条件

- Python 3.6+（macOS / Linux 自带）
- 验签（可选）需额外安装：`pip install cryptography`
- **无需部署服务端** — 中继服务已在云端运行

## 使用

安装完成后，在 AI Agent 对话中说：

> 帮我接收支付宝异步通知

Agent 会自动完成注册、给你 `notify_url`、监听通知、验签等全部流程。

## 仓库结构

| 文件 | 说明 |
|------|------|
| `SKILL.md` | Agent 指令文件（Agent 自动读取，无需人工操作） |
| `scripts/cli.py` | CLI 工具，纯 Python 3 标准库，约 800 行 |
| `README.md` | 本文件 |

## 限制

- **仅限联调 / 沙箱 / 内部调试**，不可用于生产环境
- 每个 IP 限注册 1 次
- 通知保留 1 天，每租户最多 200 条

## License

MIT
