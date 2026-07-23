# OwlSpeak 统一发布仓库

本仓库是 [OwlSpeak](https://github.com/OwlSpeak) 组织的**版本发布中心**，汇总：

| 组件 | 源码仓库 | 产物 |
|------|----------|------|
| 桌面客户端 | [Owl-Desktop](https://github.com/OwlSpeak/Owl-Desktop) | Windows / macOS / Linux 安装包 + **可独立部署的 Web 前端包** |
| 控制面服务端 | [Owl-Server](https://github.com/OwlSpeak/Owl-Server) | 多平台 `owl-server` 二进制 |
| 媒体面 SFU | [Owl-SFU](https://github.com/OwlSpeak/Owl-SFU) | 多平台 `owl-sfu` 二进制 |
| 未来 App | 待定 | 移动端安装包等 |

## 如何获取更新

打开右侧 **[Releases](https://github.com/OwlSpeak/OwlSpeak/releases)**，按版本号（如 `v0.1.0`）下载对应平台文件。

同一版本号下会聚合各组件的：

- 安装包 / 二进制产物
- 各组件 changelog（根据 commit 与作者补充说明自动汇总）

## 版本约定

- 标签格式：`vMAJOR.MINOR.PATCH`（例如 `v0.1.0`）
- **源码仓**与**本发布仓**使用相同版本标签
- 各源码仓仍会在自己仓库创建 Release（便于开发者查阅）
- 源码仓发版成功后，会通过 CI **自动同步**产物与说明到本仓同版本 Release

## 产物命名（摘要）

| 前缀 | 说明 |
|------|------|
| `owl-desktop-*` | 桌面安装包（dmg / msi / deb / AppImage 等） |
| `owl-desktop-web-*.zip` | 纯前端静态包（可放到任意静态托管 / Nginx） |
| `owl-server-<ver>-<os>-<arch>` | 服务端二进制 |
| `owl-sfu-<ver>-<os>-<arch>` | SFU 二进制 |
| `SHA256SUMS-*` | 校验和 |

## 给维护者

1. 在各源码仓打标签：`git tag -a vX.Y.Z -m "本版本重点说明..."` 并 `git push origin vX.Y.Z`
2. 源码仓 Actions 构建 → 本仓 Release + 本仓 Release
3. 组织级 Secret（各源码仓需要）：

   | Secret | 用途 |
   |--------|------|
   | `RELEASE_HUB_TOKEN` | 具有 `OwlSpeak/OwlSpeak` 写权限的 PAT（或 fine-grained token：Contents 读写） |

4. 可选：在源码仓根目录维护 `RELEASE_NOTES.md`，会优先拼进发版说明

---

商业授权与双许可说明见各源码仓 `LICENSE*`。
