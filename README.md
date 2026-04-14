# alipay-notify

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Python 3.6+](https://img.shields.io/badge/python-3.6+-green.svg)](https://www.python.org)
[![GitHub release](https://img.shields.io/github/v/release/zhangke091/alipay-notify)](https://github.com/zhangke091/alipay-notify/releases)

**免费提供公网异步通知地址**，让本地开发环境也能实时接收支付宝异步通知（`notify_url`）。
无需自建服务、无需公网 IP、无需内网穿透，注册即用。服务端支持 HTTPS（TLS 1.3）+ 域名访问。

### 适用场景

- 本地联调支付宝支付（手机网站支付、电脑网站支付、当面付等）
- 沙箱环境回调调试
- CI/CD 环境回调验证
- 任何没有公网 IP 又需要接收 `notify_url` 回调的场景

```
支付宝 ──POST──► 云端中继 ──SSE──► 本地 CLI 实时拉取
                            └───► 浏览器查看器（dev.html）
```

---

## 快速上手（1 分钟）

不需要 AI Agent，直接在终端跑 3 条命令就能收到支付宝异步通知：

```bash
# 1. 注册，获取你的专属 notify_url（服务地址向管理员获取）
python3 scripts/cli.py register --server https://www.opensupport.cc --name my-dev

# 输出示例：
# ✓ 注册成功！
# notify_url
# ┌──────────────────────────────────────────────────────────────────┐
# │ https://www.opensupport.cc/notify/<your-token>                  │
# └──────────────────────────────────────────────────────────────────┘

# 2. 把 notify_url 传入支付宝下单接口（不是在开放平台控制台配置）
#    Java:   request.setNotifyUrl("<输出的 notify_url>");
#    Python: client.page_execute(request, notify_url="<输出的 notify_url>")

# 3. 开始监听，支付成功后通知实时到达
python3 scripts/cli.py listen
```

收到通知后终端输出：

```
  ┃ 新通知 #1
  ┃ 时间       2026-04-12 23:55:01
  ┃ 订单号     TEST202604122347040ccf52
  ┃ 交易号     2026041222001100001038276505
  ┃ 金额       0.01
  ┃ 状态       TRADE_SUCCESS ✓
```

更多命令：

```bash
python3 scripts/cli.py list              # 查询通知列表
python3 scripts/cli.py get 1             # 查看通知详情
python3 scripts/cli.py export 1          # 导出原始报文
python3 scripts/cli.py verify 1          # RSA2 验签（需 pip install cryptography）
python3 scripts/cli.py ack 1             # 确认通知，停止支付宝重试
```

**浏览器查看器**：访问 `https://www.opensupport.cc/dev.html`，用 API Key 登录后可实时查看通知、金额、状态，支持原始报文复制和 ACK 确认。

---

## AI Agent 用法

这也是一个标准的 **AI Agent Skill**。安装到支持 Skill 的 AI 编程工具后，对 Agent 说一句话就行：

> 帮我接收支付宝异步通知

Agent 会自动完成注册、获取 notify_url、监听、验签等全部流程。**你不需要手动输入任何命令。**

### 兼容的 AI 编程工具

| 工具 | 支持情况 |
|------|----------|
| Claude Code | ✅ 原生支持 |
| Cursor | ✅ 原生支持 |
| 其他支持 Skill / 自定义指令的 Agent | ✅ 只要能读取 SKILL.md 并执行终端命令即可 |

> 本 Skill 遵循标准的 SKILL.md 规范，不依赖特定工具的私有 API。

## 安装

### 作为 AI Skill 安装

```bash
# Claude Code
mkdir -p .claude/skills
git clone https://github.com/zhangke091/alipay-notify .claude/skills/alipay-notify

# Cursor
mkdir -p .cursor/skills
git clone https://github.com/zhangke091/alipay-notify .cursor/skills/alipay-notify
```

### 独立使用（不需要 AI 工具）

```bash
git clone https://github.com/zhangke091/alipay-notify
cd alipay-notify
python3 scripts/cli.py register --server https://www.opensupport.cc --name my-dev
```

## 前置条件

- Python 3.6+（macOS / Linux 自带）
- 验签（可选）需额外安装：`pip install cryptography`
- **无需部署服务端** — 中继服务已在 `https://www.opensupport.cc` 运行

## 数据安全

| 机制 | 说明 |
|------|------|
| **传输加密** | 服务端支持 HTTPS（TLS 1.3），Let's Encrypt 证书自动续期，所有 API 调用和通知接收均加密传输 |
| **云端只做转发** | 中继服务接收支付宝 POST，原样存储 `raw_body`，通过 SSE 推送到你的本地 CLI 或浏览器 |
| **租户完全隔离** | 每个开发者独立 token + API Key，无法访问他人数据 |
| **凭证安全** | 注册凭证仅在首次注册时返回，重复注册不会泄露已有凭证 |
| **验签在本地** | 支付宝公钥只保存在你本地的 `.alipay-notify.json`，云端不存任何密钥 |
| **自动清除** | 通知保留 1 天后自动删除，每租户最多 200 条 |
| **仅限联调** | 不可用于生产环境，仅供开发调试使用 |

## 仓库结构

| 文件 | 说明 |
|------|------|
| `SKILL.md` | Agent 指令文件（Agent 自动读取，无需人工操作） |
| `scripts/cli.py` | CLI 工具，纯 Python 3 标准库，约 800 行 |
| `README.md` | 本文件 |

## 限制

- **仅限联调 / 沙箱 / 内部调试**，不可用于生产环境
- 每个 IP 限注册 1 次，凭证丢失请联系管理员
- 通知保留 1 天，每租户最多 200 条

## License

MIT
