# Newt-Server 部署

控制面：HTTP API、管理 SPA、Gateway WS、SFU 控制 gRPC。媒体不经 Server。

## 架构要点

```
Client ──HTTPS/WSS──► Caddy ──► owl-server :8080
                                    │
                                    ├── PostgreSQL :5432
                                    └── gRPC mTLS :9443 ◄── SFU 主动外连
```

## 前置

| 项 | 说明 |
|----|------|
| 系统 | Linux amd64（推荐 Debian/Ubuntu） |
| 二进制 | `owl-server`（Release 或自行交叉编译） |
| 依赖 | Docker（跑 PostgreSQL）、Caddy（TLS 反代） |
| DNS | 面板域名 A 记录指向本机公网 IP |
| 端口 | `80/443` 公网；`8080`、`5432`、`9443` 建议仅本机 |

## 目录约定

```
/opt/newtspeak/
  bin/newt-server
  newt-server.env
  data/server/          # CA、Media Token 密钥、附件等
  docker-compose.yml    # 仅 PostgreSQL
```

## 1. 准备二进制

从 [NewtSpeak Releases](https://github.com/NewtSpeak/NewtSpeak/releases) 下载 `owl-server-<ver>-linux-amd64`，或本机构建：

```bash
# monorepo：先构建前端再打进二进制
cd Newt-Server/frontend && bun run build
# 将 frontend/build/client 拷入 backend/internal/web/dist
cd ../backend
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags="-s -w" -o bin/newt-server ./cmd/server
```

```bash
install -D bin/newt-server /opt/newtspeak/bin/newt-server
mkdir -p /opt/newtspeak/data/server
```

## 2. PostgreSQL

```bash
# /opt/newtspeak/docker-compose.yml
services:
  postgres:
    image: postgres:18-alpine
    container_name: owl-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: owl
      POSTGRES_USER: owl
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}   # 强随机
    ports:
      - "127.0.0.1:5432:5432"
    volumes:
      - owl-postgres-data:/var/lib/postgresql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U owl -d owl"]
      interval: 5s
      timeout: 3s
      retries: 20

volumes:
  owl-postgres-data:
```

```bash
echo "POSTGRES_PASSWORD=<强随机>" > /opt/newtspeak/.env
chmod 600 /opt/newtspeak/.env
cd /opt/newtspeak && docker compose up -d
```

## 3. 环境变量

`/opt/newtspeak/newt-server.env`（`chmod 600`）：

```bash
APP_ENV=production
APP_ADDRESS=127.0.0.1:8080
DATABASE_URL=postgres://owl:<PG密码>@127.0.0.1:5432/owl?sslmode=disable
JWT_SECRET=<≥32 字符随机串>
ACCESS_TOKEN_TTL=15m
REFRESH_TOKEN_TTL=720h
DATA_DIR=/opt/newtspeak/data/server
PUBLIC_BASE_URL=https://newt-panel.example.com
AUDIT_INGEST_TOKEN=<强随机>          # 生产开启审计上传必填

# SFU 控制面（本机 SFU 用 127.0.0.1；外置节点改可达地址）
SFU_GRPC_ADDRESS=0.0.0.0:9443
SFU_CONTROL_PUBLIC_ENDPOINT=127.0.0.1:9443
SFU_CONTROL_TLS_SANS=localhost,127.0.0.1,newt-panel.example.com

# 生产用独立 SFU，不要开内嵌
EMBEDDED_SFU=false
```

| 必填 | 说明 |
|------|------|
| `DATABASE_URL` | 仅 PostgreSQL |
| `JWT_SECRET` | ≥32 字符 |
| `PUBLIC_BASE_URL` | 对外根 URL（邀请链、审计上传） |
| `APP_ENV=production` | 才提供内嵌 SPA |

可选：`OTEL_EXPORTER_OTLP_ENDPOINT`（SigNoz）、`METRICS_ADDRESS=127.0.0.1:9091`。

## 4. systemd

`/etc/systemd/system/newt-server.service`：

```ini
[Unit]
Description=NewtSpeak Control Server
After=network-online.target docker.service
Wants=network-online.target
Requires=docker.service

[Service]
Type=simple
WorkingDirectory=/opt/newtspeak
EnvironmentFile=/opt/newtspeak/newt-server.env
ExecStart=/opt/newtspeak/bin/newt-server
Restart=on-failure
RestartSec=3
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now owl-server
systemctl status owl-server
journalctl -u owl-server -f
```

## 5. Caddy（面板 TLS）

```caddyfile
newt-panel.example.com {
	encode gzip zstd
	reverse_proxy 127.0.0.1:8080 {
		header_up Host {host}
		header_up X-Real-IP {remote_host}
		header_up X-Forwarded-For {remote_host}
		header_up X-Forwarded-Proto {scheme}
	}
}
```

```bash
systemctl reload caddy
```

## 6. 验收

```bash
curl -fsS -o /dev/null -w "%{http_code}\n" http://127.0.0.1:8080/
# 公网：https://newt-panel.example.com
```

- 空库首次注册账号 → 自动成为系统管理员；之后 `/signup` 关闭。
- 面板「节点管理」创建 SFU 占位并拿 enroll token → 交给 SFU 节点（见 [sfu.md](./sfu.md)）。

## 升级

```bash
systemctl stop owl-server
install -D newt-server-new /opt/newtspeak/bin/newt-server
systemctl start owl-server
```

保留 `newt-server.env` 与 `data/server/`（含集群 CA / Media Token 密钥，勿丢）。

## 一键脚本（同机 Server+SFU）

仓库 `deploy/prod/install.sh` 可一键装 Docker/Caddy/systemd 并拉起 PG + server + sfu。单独只部署 Server 时按上文分步即可。

## 防火墙（最小）

```
22/tcp  80/tcp  443/tcp
# 外置 SFU 需要连控制面时再开：
9443/tcp   # 建议仅对 SFU 源 IP
```
