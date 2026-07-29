# NewtSpeak

**开源的 [Discord](https://discord.com) / [KOOK](https://www.kookapp.cn) 替代方案** — 自托管组队语音与社区协作软件。

数据在你自己的服务器上：多服务器（Guild）、文字频道、语音/舞台通话、屏幕共享、细粒度权限、贴图表情、好友私信，以及机器人与 AI 运维扩展。

> 英文关键词（便于搜索）：`Discord alternative` · `self-hosted Discord` · `open source team voice` · `Discord-like` · `KOOK alternative` · `WebRTC voice chat`

## 为什么做 NewtSpeak

| 你需要… | NewtSpeak |
|---------|-----------|
| 像 Discord 一样组队语音 + 文字 | ✅ 多服务器、频道、语音、舞台 |
| 数据与权限完全自控 | ✅ 自托管，不经过第三方云 |
| 开源可审计、可二次开发 | ✅ 源码开放（双许可见各组件仓） |
| 机器人 / 自动化 | ✅ 官方 Bot SDK + Agent CLI / MCP |

## 核心能力

- **服务器与频道**：创建/加入多个互相独立的服务器；文本 / 语音 / 分类 / 舞台频道  
- **组队语音**：WebRTC + SFU，支持大房舞台、屏幕共享、连接热迁移  
- **权限 (RBAC)**：角色、频道覆盖、与 Discord 对齐的权限位模型  
- **消息与搜索**：收发、反应、附件、全站搜索、Bot 卡片与流式消息  
- **社交**：好友、隐私、私信/群、通知收件箱  
- **开放生态**：机器人 API（类 Discord Bot）+ CLI / MCP 给 AI 运维  

## 组件一览

| 组件 | 仓库 | 作用 |
|------|------|------|
| 桌面 / Web 客户端 | [Newt-Desktop](https://github.com/NewtSpeak/Newt-Desktop) | 用户端（对标 Discord 客户端体验） |
| 控制面 | [Newt-Server](https://github.com/NewtSpeak/Newt-Server) | 账号、权限、消息、语音调度、Bot API |
| 媒体面 SFU | [Newt-SFU](https://github.com/NewtSpeak/Newt-SFU) | 实时语音与屏幕流转发 |
| 机器人 SDK | [NewtBotSdk](https://github.com/NewtSpeak/NewtBotSdk) | JS / Go / Python / Rust |
| Agent CLI | [Newt-Agent](https://github.com/NewtSpeak/Newt-Agent) | 命令行 + MCP（AI 宿主） |

本仓库是 **统一发布中心**：不放运行时源码，只汇总各版本 **安装包 / 二进制 / changelog**。

## 下载

打开 **[Releases](https://github.com/NewtSpeak/NewtSpeak/releases)**，按版本号（如 `v0.1.0`）获取：

| 产物前缀 | 说明 |
|----------|------|
| `newt-desktop-*` | 桌面安装包（Windows / macOS / Linux） |
| `newt-desktop-web-*.zip` | 纯 Web 静态包 |
| `newt-server-*` | 控制面服务端二进制 |
| `newt-sfu-*` | 媒体面 SFU 二进制 |
| `SHA256SUMS-*` | 校验和 |

## 最小自托管组合

1. **newt-server** + PostgreSQL + 反向代理（如 Caddy）  
2. **newt-sfu**（语音；可与 Server 同机或独立）  
3. **桌面客户端** 或 Web 静态包  

部署文档：

- [Server 部署](./docs/deploy-server.md)  
- [SFU 部署](./docs/deploy-sfu.md)  
- [生态总览](./docs/MONOREPO.md) · [文档索引](./docs/INDEX.md)  

## 与 Discord 的关系（定位说明）

NewtSpeak **不是** Discord 官方产品或插件，而是独立的开源实现，在产品能力上对标 Discord / KOOK 的组队语音与社区体验，并强调：

- 可私有化部署  
- 协议与权限可自研扩展  
- 机器人与 AI 运维一等公民  

适合：游戏公会、工作室、开源社区、内网团队等需要「类 Discord 语音房」且数据不出境的场景。

## 版本约定

- 标签：`vMAJOR.MINOR.PATCH`  
- 源码仓与本发布仓使用相同版本 tag  
- 源码仓发版后，CI 可同步产物到本仓同版本 Release  

维护者说明：各源码仓配置 `RELEASE_HUB_TOKEN`（对本仓有 Contents 写权限）后可自动同步。可选 `RELEASE_NOTES.md`。

## 许可证

商业授权与双许可条款见 **各源码仓** `LICENSE*`，不以本仓替代。

---

**Star** 本组织与各组件仓，并打上 `discord` / `discord-alternative` 等 Topic，便于更多需要「开源 Discord 替代 / 自托管组队语音」的用户发现 NewtSpeak。
