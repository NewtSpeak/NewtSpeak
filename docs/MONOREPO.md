# OwlSpeak

自托管的社区协作平台（对标 Discord / KOOK）：多服务器、文本/语音、细粒度 RBAC、舞台、屏幕共享、机器人开放平面与 AI Agent 运维入口。

本目录为 **开发 monorepo**（本地联调）。各组件可独立成仓发布；统一下载见 [OwlSpeak 发布仓](./OwlSpeak/)。

## 架构一览

```text
                    ┌─────────────┐
  用户 / 浏览器 ──► │ Owl-Desktop │  Tauri 壳 + Web / 纯 Web 包
                    └──────┬──────┘
           REST + Gateway  │  媒体 WSS + WebRTC UDP
                           │
         ┌─────────────────┼─────────────────┐
         ▼                                   ▼
  ┌─────────────┐                     ┌─────────────┐
  │ Owl-Server  │ ◄── mTLS gRPC ───── │  Owl-SFU    │
  │  控制面     │                     │  媒体面     │
  │  + 管理 SPA │                     │  选路转发   │
  └──────┬──────┘                     └─────────────┘
         │
         ▼
   PostgreSQL

  Bot 进程 ── OwlBotSdk ──► /bot-api/v1
  运维 / AI ── Owl-Agent ──► OAuth + /gapi/v1（+ MCP）
```

| 信任区 | 组件 | 说明 |
|--------|------|------|
| 不可信客户端 | Desktop / Agent / Bot | 权限最终以 Server 裁决为准 |
| 公网 TLS | Caddy 等 | 终结 HTTPS / WSS |
| 控制面 | Server + PostgreSQL | 账号、RBAC、消息、调度、签发 Media Token |
| 媒体面 | SFU | 音视频不经 Server；只认短时 Media Token |

## 仓库一览

| 目录 | 角色 | 一句话 |
|------|------|--------|
| [**Owl-Server**](./Owl-Server/) | 控制面 | 账号/Guild/RBAC/消息/语音调度/Bot API/OAuth；内嵌管理后台 |
| [**Owl-SFU**](./Owl-SFU/) | 媒体面 | WebRTC 选路转发、级联、热迁移；主动 mTLS 回连 Server |
| [**Owl-Desktop**](./Owl-Desktop/) | 用户端 | 桌面客户端 + 可独立部署的 Web 前端 |
| [**OwlBotSdk**](./OwlBotSdk/) | 机器人 SDK | JS / Go / Python / Rust 官方 Bot 客户端 |
| [**Owl-Agent**](./Owl-Agent/) | CLI · MCP · Skill | 用户 OAuth 委托的运维与 AI 工具入口 |
| [**OwlSpeak**](./OwlSpeak/) | 发布中心 | 聚合各组件 Release 产物与 changelog |
| [**docs/**](./docs/) | 文档 | 部署、SDK、Agent、架构图、设计记录 |
| [**deploy/**](./deploy/) | 编排 | 生产 systemd / Caddy / 安装脚本等 |

## 文档入口

| 文档 | 说明 |
|------|------|
| [docs/README.md](./docs/README.md) | 文档总索引 |
| [docs/deploy/server.md](./docs/deploy/server.md) | Server 部署 |
| [docs/deploy/sfu.md](./docs/deploy/sfu.md) | SFU 部署 |
| [docs/sdk/usage.md](./docs/sdk/usage.md) | Bot SDK 使用 |
| [docs/agent/usage.md](./docs/agent/usage.md) | Agent CLI 使用 |
| [docs/architecture/](./docs/architecture/) | 运行时 / 内部结构图 |

## 本地开发（建议顺序）

1. **Server**：`Owl-Server` 起 PostgreSQL + `make dev`（开发默认内嵌 SFU）  
2. **Desktop**：`Owl-Desktop` → `bun install` → `bun run dev` / `dev:tauri`  
3. **独立 SFU**（可选）：见 `Owl-SFU` 与 [deploy/sfu](./docs/deploy/sfu.md)  
4. **Bot / Agent**（可选）：`OwlBotSdk`、`Owl-Agent`

生产推荐：独立 `owl-server` + `owl-sfu` + Caddy + PostgreSQL（`deploy/prod/install.sh`）。

## 许可证

- **Owl-Server / Owl-SFU / Owl-Desktop**：双重许可（个人非商用免费 + 强制开源义务；商用需授权）——见各仓 `LICENSE*`  
- **OwlBotSdk / Owl-Agent**：以各仓 `LICENSE` 为准（SDK 为 MIT）
