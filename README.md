# NewtSpeak（统一发布仓）

[NewtSpeak](https://github.com/NewtSpeak) 组织的 **版本发布中心**：不承载运行时源码，只汇总各组件 **安装包 / 二进制 / changelog**，方便用户一站下载。

开发与贡献请到各源码仓；本仓用于 **Release 分发**。

## 发布了什么

| 组件 | 源码仓库 | 本仓产物 |
|------|----------|----------|
| 桌面客户端 | [Newt-Desktop](https://github.com/NewtSpeak/Newt-Desktop) | Windows / macOS / Linux 安装包 + **Web 静态 zip** |
| 控制面 | [Newt-Server](https://github.com/NewtSpeak/Newt-Server) | 多平台 `newt-server` 二进制 |
| 媒体面 | [Newt-SFU](https://github.com/NewtSpeak/Newt-SFU) | 多平台 `newt-sfu` 二进制 |
| 机器人 SDK | [NewtBotSdk](https://github.com/NewtSpeak/NewtBotSdk) | 随源码仓发布；文档见 monorepo `docs/sdk` |
| Agent CLI | [Newt-Agent](https://github.com/NewtSpeak/Newt-Agent) | 多平台 `newt` 二进制（若已接入 hub） |
| 移动端 | 待定 | — |

## 如何获取

打开 **[Releases](https://github.com/NewtSpeak/NewtSpeak/releases)**，按版本号（如 `v0.1.0`）下载。

同一 tag 下通常包含：

- 各平台安装包 / 二进制  
- 各组件 changelog（commit + 作者说明自动汇总）  
- 校验和文件  

## 产物命名

| 前缀 | 说明 |
|------|------|
| `newt-desktop-*` | 桌面安装包（dmg / msi / deb / AppImage 等） |
| `newt-desktop-web-*.zip` | 纯前端静态包（Nginx / 任意静态托管） |
| `newt-server-<ver>-<os>-<arch>` | 控制面服务端 |
| `newt-sfu-<ver>-<os>-<arch>` | 媒体面 SFU |
| `owl-*`（Agent 等） | 以当次 Release 说明为准 |
| `SHA256SUMS-*` | 校验和 |

## 版本约定

- 标签：`vMAJOR.MINOR.PATCH`（例 `v0.1.0`）  
- **源码仓**与**本发布仓**使用相同版本 tag  
- 各源码仓仍可自建 Release（便于开发者）  
- 源码仓发版成功后，CI **自动同步**产物与说明到本仓同版本 Release  

## 自托管最小组合

从本仓（或对应源码仓 Release）取：

1. `newt-server` + PostgreSQL + 反代（Caddy）  
2. `newt-sfu`（语音；可与 Server 同机或独立）  
3. 桌面安装包，或 `newt-desktop-web-*.zip`  

部署说明：

- [Server 部署](./docs/deploy-server.md)  
- [SFU 部署](./docs/deploy-sfu.md)  
- [生态总览](./docs/MONOREPO.md) · [文档索引](./docs/INDEX.md)  

## 给维护者

1. 源码仓：`git tag -a vX.Y.Z -m "…"` → `git push origin vX.Y.Z`  
2. 源码仓 Actions 构建 → 同步到本仓 Release  
3. 各源码仓需组织 Secret：

   | Secret | 用途 |
   |--------|------|
   | `RELEASE_HUB_TOKEN` | 对本仓 `NewtSpeak/NewtSpeak` 有 Contents 写权限的 PAT |

4. 可选：源码仓根目录 `RELEASE_NOTES.md` 优先拼进发版说明  

## 生态文档

| 文档 | 说明 |
|------|------|
| [docs/INDEX.md](./docs/INDEX.md) | 文档总索引 |
| [docs/MONOREPO.md](./docs/MONOREPO.md) | 组件总览与架构 |
| [docs/deploy-server.md](./docs/deploy-server.md) | Server 部署 |
| [docs/deploy-sfu.md](./docs/deploy-sfu.md) | SFU 部署 |
| [NewtBotSdk](https://github.com/NewtSpeak/NewtBotSdk) | Bot SDK |
| [Newt-Agent](https://github.com/NewtSpeak/Newt-Agent) | Agent CLI |

## 许可证

商业授权与双许可条款见 **各源码仓** `LICENSE*`，不以本仓替代。
