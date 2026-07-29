# OwlSpeak 文档索引

配套 monorepo 总览：[仓库根 README](../README.md)。

## 按角色

| 你想… | 去读 |
|--------|------|
| 部署控制面 | [deploy/server.md](./deploy/server.md) |
| 部署媒体节点 | [deploy/sfu.md](./deploy/sfu.md) |
| 写机器人 | [sdk/usage.md](./sdk/usage.md) → [sdk/api.md](./sdk/api.md) |
| 用 CLI / 接 AI | [agent/usage.md](./agent/usage.md) → [agent/api.md](./agent/api.md) |
| 看系统怎么连 | [architecture/](./architecture/) |
| 查设计定稿 | [design/](./design/) 与 `Owl-Server/docs/设计讨论/` |

## 目录

| 分类 | 文档 | 说明 |
|------|------|------|
| **部署** | [deploy/server.md](./deploy/server.md) | Owl-Server 生产部署 |
| | [deploy/sfu.md](./deploy/sfu.md) | Owl-SFU 媒体节点部署 |
| **Bot SDK** | [sdk/usage.md](./sdk/usage.md) | 机器人 SDK 使用 |
| | [sdk/api.md](./sdk/api.md) | `/bot-api/v1` 接口调用 |
| **Agent CLI** | [agent/usage.md](./agent/usage.md) | `owl` CLI / MCP 使用 |
| | [agent/api.md](./agent/api.md) | 命令、Tools、底层 API |
| **架构** | [architecture/](./architecture/) | 运行时 / 内部结构 / 时序图 |
| **设计** | [design/](./design/) | monorepo 级功能设计记录 |

## 各仓库 README

| 仓库 | 说明 |
|------|------|
| [Owl-Server](../Owl-Server/README.md) | 控制面 |
| [Owl-SFU](../Owl-SFU/README.md) | 媒体面 |
| [Owl-Desktop](../Owl-Desktop/README.md) | 用户端 |
| [OwlBotSdk](../OwlBotSdk/README.md) | 官方 Bot SDK |
| [Owl-Agent](../Owl-Agent/README.md) | CLI · MCP · Skill |
| [OwlSpeak](../OwlSpeak/README.md) | 统一发布仓 |

## 组件内深度文档

| 路径 | 内容 |
|------|------|
| `Owl-Server/docs/设计讨论/` | 架构与产品定稿（编号越大越权威） |
| `Owl-Server/docs/协议/` | Media Token、级联、热迁移 |
| `Owl-Server/backend/README.md` | 后端开发与环境变量 |
| `Owl-Desktop/docs/` | 客户端功能地图 00–23 |
| `OwlBotSdk/docs/API.md` | Bot 协议完整版 |
| `Owl-Agent/docs/DEEP-LINK.md` | 深链 |
| `deploy/prod/` | 生产脚本与 unit 文件 |
