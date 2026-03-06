# OpenClaw 安装配置指南

> 适用于 WSL2 Ubuntu 环境，配置局域网访问 + 飞书机器人

---

## 1. 环境检查

```bash
# 检查 Node.js 版本（需要 22+）
node --version

# 检查 npm
npm --version

# 安装 Redis（可选）
sudo apt install redis-server -y
```

---

## 2. 安装 OpenClaw

```bash
# 全局安装
npm install -g openclaw

# 验证安装
openclaw --version
```

---

## 3. 初始化配置

```bash
# 运行配置向导
openclaw onboard

# 或手动创建配置目录
mkdir -p ~/.openclaw
```

---

## 4. 配置局域网访问

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "auth": {
      "mode": "token",
      "token": "your_token_here"
    },
    "controlUi": {
      "dangerouslyAllowHostHeaderOriginFallback": true,
      "allowInsecureAuth": true,
      "dangerouslyDisableDeviceAuth": true,
      "allowedOrigins": [
        "http://localhost:18789",
        "http://127.0.0.1:18789"
      ]
    },
    "tailscale": {
      "mode": "off"
    }
  }
}
```

**关键配置说明：**
- `bind`: 设置为 `"lan"` 允许局域网访问
- `controlUi`: 添加此配置解决浏览器断开问题
- `token`: 设置你的访问令牌

---

## 5. 配置飞书机器人

### 5.1 飞书开放平台设置

1. 访问 https://open.feishu.cn/app
2. 创建企业自建应用
3. 获取 **App ID** 和 **App Secret**
4. 添加权限：消息、群组机器人、事件订阅
5. 订阅事件：`im.message.receive_v1`

### 5.2 OpenClaw 配置

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "accounts": {
        "default": {
          "appId": "你的App_ID",
          "appSecret": "你的App_Secret",
          "mode": "websocket"
        }
      }
    }
  }
}
```

---

## 6. 启动 Gateway

```bash
# 后台启动
openclaw gateway run &

# 或使用 systemd（推荐）
openclaw gateway install
systemctl --user enable --now openclaw-gateway.service
```

---

## 7. 配置开机自启

```bash
# 启用用户 linger（开机自启）
sudo loginctl enable-linger $(whoami)

# 验证
loginctl show-user $(whoami) | grep Linger
```

---

## 8. 验证安装

```bash
# 查看状态
openclaw status

# 获取局域网 IP
ip addr show | grep "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"

# 访问 Web UI
# http://<局域网IP>:18789/
```

---

## 9. 常见问题排查

### Q1: Gateway 无法启动
```bash
# 查看日志
openclaw logs --follow

# 检查进程
ps aux | grep openclaw

# 检查端口占用
sudo lsof -i :18789
```

### Q2: 浏览器显示 disconnected
- 确保配置中包含 `controlUi` 部分
- 重启 Gateway：`systemctl --user restart openclaw-gateway.service`

### Q3: 局域网无法访问
- 确认 `bind` 设置为 `"lan"`
- 检查防火墙是否开放 18789 端口

---

## 10. 常用命令

| 命令 | 说明 |
|------|------|
| `openclaw status` | 查看状态 |
| `openclaw gateway restart` | 重启 Gateway |
| `openclaw logs --follow` | 查看日志 |
| `systemctl --user restart openclaw-gateway.service` | 重启服务 |

---

**注意**：配置中的 `token`、`appId`、`appSecret` 等敏感信息请自行替换为真实值，切勿提交到公开仓库。
