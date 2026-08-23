# WorkBuddy 账户全景仪表盘

一个**本地运行**的 WorkBuddy / CodeBuddy 账户调试仪表盘。读取本机客户端登录态，调用官方接口，把账户额度、积分套餐、签到、JWT 有效期、AI 记忆画像、本机环境等信息集中展示在一个网页里，方便自查与排错。

> ⚠️ 本项目**仅读取你本机 WorkBuddy 客户端已保存的登录态**（位于 `~/Library/Application Support/CodeBuddyExtension/Data/Public/auth/workbuddy-desktop.info`），不会向你索取账号密码，也不会把任何凭证上传到除官方接口以外的第三方。**请只在自己信任的机器上运行，并不要把包含登录态的文件提交到任何仓库。**

---

## 功能

- **概览卡片**：权益赠送包剩余 / 体验版剩余 / 全部剩余 / 登录态剩余天数
- **账户档案**：昵称、UID、UIN、账号类型、手机号、是否管理员（取自本机登录态）
- **登录态有效期**：解析 JWT，显示签发者 / 过期时间 / 剩余天数（进度环）
- **积分套餐明细**：调用 `/billing/meter/get-user-resource`，按官方口径区分「权益赠送包」与「体验版」（CapacityType=4），并附**到期时间轴**（30 天内红色预警）
- **每日签到**：调用 `/billing/meter/checkin-activity-status`，展示活动状态、连续天数、今日签到、最近记录
- **AI 记忆画像**：调用 `/api/memory/profile`，按 `## 标题` 折叠展示（解决长文本不直观问题）
- **本机环境**：客户端版本、构建号、安装大小、系统 / 架构 / Node
- **一键导出报告**：生成 Markdown 快照，可复制粘贴到备忘或文档
- **左侧边栏**：当前账号、已登记账号、本地数据目录列表（可点击切换）

---

## 快速开始

需要 Node.js 18+（macOS 桌面端路径为硬编码默认值，Windows / Linux 可自行改 `credits-api.js` 与 `debug-server.mjs` 中的 `authCandidates()`）。

```bash
# 1. 克隆
git clone https://github.com/xmgzxmgz/workbuddy-account-dashboard.git
cd workbuddy-account-dashboard

# 2. 启动本地调试服务（默认端口 8765）
npm start
# 或： node debug-server.mjs

# 3. 浏览器打开
open http://localhost:8765
```

首次打开会自动读本机登录态并拉取全部数据。点击「刷新全部」可重新拉取，「仅查积分」只请求额度接口，「导出报告」生成可复制的快照。

### 命令行模式（无需网页）

`credits-api.js` 也支持纯命令行使用：

```bash
node credits-api.js status   # 查询签到状态
node credits-api.js quota    # 查询积分额度（区分赠送包 / 体验版）
node credits-api.js checkin  # 执行今日签到（幂等，已签则跳过）
node credits-api.js token    # 仅打印本机登录态摘要（token 默认脱敏）
```

---

## 接口说明（均为官方接口，仅用本机登录态 Bearer Token 调用）

| 用途 | 方法 | 路径 |
| --- | --- | --- |
| 积分套餐 | POST | `https://copilot.tencent.com/billing/meter/get-user-resource` |
| 签到状态 | GET | `https://copilot.tencent.com/v2/billing/meter/checkin-activity-status` |
| 执行签到 | POST | `https://copilot.tencent.com/v2/billing/meter/daily-checkin` |
| 记忆画像 | GET | `https://copilot.tencent.com/api/memory/profile` |

---

## 安全与隐私

- **本地优先**：所有数据读取与接口调用都发生在你本机，调试服务只监听 `127.0.0.1`（localhost）。
- **不打包凭证**：仓库 `.gitignore` 忽略 `*.info`、`vault/`、`*.poc`、`*.enc` 等任何可能含登录态的文件。
- **脱敏**：命令行 `token` 子命令默认只打印前若干字符与长度，不输出完整 Token。
- **登录态会过期**：JWT 通常约 90 天有效，失效后需重新登录 WorkBuddy 客户端再运行本工具。

---

## 已知限制

- 账号切换：本机只保存当前登录态的 Token，侧边栏点非当前账号仅能展示本机登记档案，无法读取该账号的远程额度 / JWT（需先在 WorkBuddy 客户端内切换登录）。
- 平台：默认路径针对 macOS 桌面端；Windows / Linux 用户需修改 `authCandidates()` 中的路径。

---

## 许可

MIT
