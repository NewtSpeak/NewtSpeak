# Newt-SFU 部署

媒体面节点：WebRTC 选路转发。控制面见 [server.md](./server.md)。

## 架构要点

```
Client ──WSS──► Caddy ──► newt-sfu :8443（信令）
Client ──UDP──► newt-sfu :3478（媒体，直连，不经 Caddy）
newt-sfu ──mTLS gRPC──► Server :9443（主动外连，不反向暴露管理面）
```

- 首次：`enroll token` → 领 mTLS 证书 → 落盘 `DATA_DIR`
- 之后：无 token 即可重启；证书自动续期

## 前置

| 项 | 说明 |
|----|------|
| 系统 | Linux amd64 |
| 二进制 | `newt-sfu` |
| 已就绪 | Newt-Server 在线，控制面 gRPC 可达 |
| DNS | SFU 域名 A 记录 → 本机公网 IP（信令 WSS） |
| 公网 IP | 填入 `NEWTSFU_PUBLIC_IP`（NAT1To1 host candidate） |

## 端口

| 端口 | 方向 | 用途 |
|------|------|------|
| `tcp/8443` | 本机（Caddy 反代） | 信令 `/ws`、`/rtt`、`/healthz`、`/metrics` |
| `udp/3478` | **公网入站** | WebRTC 媒体（UDPMux，必开） |
| `tcp/8843` | 节点间（可选） | 级联 mTLS |
| `tcp/9443` | 出站到 Server | Enroll + 控制通道 |

## 目录约定

```
/opt/newtspeak/
  bin/newt-sfu
  newt-sfu.env
  data/sfu/             # 证书、密钥、CA、控制面地址（enroll 后生成）
```

## 1. 准备二进制

Release 下载 `newt-sfu-<ver>-linux-amd64`，或：

```bash
cd Newt-SFU
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags="-s -w" -o bin/newt-sfu ./cmd/newt-sfu
install -D bin/newt-sfu /opt/newtspeak/bin/newt-sfu
mkdir -p /opt/newtspeak/data/sfu
```

## 2. 在 Server 创建节点占位

1. 打开面板 → **节点管理**
2. 创建占位，记下：
   - `node_id`（如 `sfu-node-1`）
   - **一次性** `enroll_token`
3. 确认 Server 侧：
   - `SFU_CONTROL_PUBLIC_ENDPOINT` 对 SFU 可达
   - `SFU_CONTROL_TLS_SANS` 含 SFU 连接时使用的主机名/IP

## 3. 环境变量

`/opt/newtspeak/newt-sfu.env`（`chmod 600`）。全部可用 `NEWTSFU_` 覆盖 `config.yaml`。

### 与 Server 同机（推荐起步）

```bash
NEWTSFU_NODE_ID=sfu-node-1
NEWTSFU_ENROLL_TOKEN=<面板拿到的一次性 token>
NEWTSFU_SERVER_ENROLL_ENDPOINT=127.0.0.1:9443
NEWTSFU_ENROLL_INSECURE=true          # 仅首次 enroll 跳过校验；领证后控制通道严格 mTLS

NEWTSFU_DATA_DIR=/opt/newtspeak/data/sfu
NEWTSFU_WSS_LISTEN=127.0.0.1:8443
NEWTSFU_NO_TLS=true                   # TLS 由 Caddy 终结
NEWTSFU_MEDIA_UDP_PORT=3478
NEWTSFU_PUBLIC_IP=<本机公网 IP>
NEWTSFU_ADVERTISE_WSS_URL=wss://newt-sfu.example.com/ws
NEWTSFU_CASCADE_LISTEN=127.0.0.1:8843
NEWTSFU_MAX_USERS=1200
```

### 独立 SFU 机（外置节点）

```bash
NEWTSFU_NODE_ID=sfu-node-2
NEWTSFU_ENROLL_TOKEN=<token>
NEWTSFU_SERVER_ENROLL_ENDPOINT=<server公网或内网>:9443
# 若 Server 证书 SAN 已含该地址，可关 insecure：
NEWTSFU_ENROLL_INSECURE=false

NEWTSFU_DATA_DIR=/opt/newtspeak/data/sfu
NEWTSFU_WSS_LISTEN=127.0.0.1:8443
NEWTSFU_NO_TLS=true
NEWTSFU_MEDIA_UDP_PORT=3478
NEWTSFU_PUBLIC_IP=<本 SFU 公网 IP>
NEWTSFU_ADVERTISE_WSS_URL=wss://sfu2.example.com/ws
NEWTSFU_CASCADE_LISTEN=0.0.0.0:8843
# 多节点级联时上报对端可达地址：
# NEWTSFU_ADVERTISE_CASCADE_ENDPOINT=<公网IP>:8843
NEWTSFU_MAX_USERS=1200
```

| 关键项 | 说明 |
|--------|------|
| `NEWTSFU_PUBLIC_IP` | 客户端 ICE 用的公网 IP；NAT 后必填 |
| `NEWTSFU_ADVERTISE_WSS_URL` | 上报给 Server、下发给客户端的信令地址 |
| `NEWTSFU_ENROLL_TOKEN` | **仅首次**；成功后可清空，证书在 `DATA_DIR` |
| `NEWTSFU_NO_TLS=true` + 本机 listen | 生产由 Caddy 终结 WSS；勿把 8443 直接暴露公网明文 |

也可用 `config.yaml`（见 `Newt-SFU/config.example.yaml`），env 优先覆盖。

## 4. systemd

`/etc/systemd/system/newt-sfu.service`：

```ini
[Unit]
Description=NewtSpeak SFU Media Node
After=network-online.target
# 同机部署时可依赖 server：
# After=network-online.target newt-server.service
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/opt/newtspeak
EnvironmentFile=/opt/newtspeak/newt-sfu.env
ExecStart=/opt/newtspeak/bin/newt-sfu
Restart=on-failure
RestartSec=3
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now newt-sfu
journalctl -u newt-sfu -f
# 期望：enroll 成功 → Register → 心跳
```

Enroll 成功后可从 env 去掉 `NEWTSFU_ENROLL_TOKEN`（证书已落盘）。

## 5. Caddy（信令 TLS）

```caddyfile
newt-sfu.example.com {
	encode gzip zstd
	reverse_proxy 127.0.0.1:8443 {
		header_up Host {host}
		header_up X-Real-IP {remote_host}
		header_up X-Forwarded-For {remote_host}
		header_up X-Forwarded-Proto {scheme}
		transport http {
			versions 1.1
		}
	}
}
```

**媒体 UDP 不走 Caddy**，防火墙必须放行 `3478/udp` 到本机。

## 6. 防火墙

```
80/tcp  443/tcp     # Caddy / ACME
3478/udp            # WebRTC 媒体（必须公网可达）
# 多节点级联：
8843/tcp            # 仅对其他 SFU
# 出站：到 Server 9443/tcp
```

## 7. 验收

```bash
curl -fsS http://127.0.0.1:8443/healthz
# 面板节点列表：状态在线、心跳正常
# 客户端进语音频道，确认 ICE 使用 PUBLIC_IP，媒体通
```

| 现象 | 排查 |
|------|------|
| enroll 失败 | token 是否一次性已用；`SERVER_ENROLL_ENDPOINT` 可达；SAN 是否匹配 |
| healthz OK 但语音不通 | `3478/udp` 是否开放；`PUBLIC_IP` 是否正确 |
| 客户端连不上 WSS | Caddy / `ADVERTISE_WSS_URL` / DNS |
| 反复重连控制面 | Server `SFU_CONTROL_PUBLIC_ENDPOINT` 与证书 SAN |

## 升级

```bash
systemctl stop newt-sfu          # 默认 drain ~60s 等会话迁空
install -D newt-sfu-new /opt/newtspeak/bin/newt-sfu
systemctl start newt-sfu
```

保留 `data/sfu/`（证书与密钥）。也可在管理后台走远程升级（Server 配置 `SFU_RELEASE_DIR`）。

## 同机一键

`deploy/prod/install.sh` 同时装 Server + SFU。仅加 SFU 节点时：拷二进制 → 写 env → enroll → systemd → Caddy → 开 `3478/udp`。
