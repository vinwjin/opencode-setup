# OpenClaw 快速设置指南

> 快速参考手册，适用于问题排查和配置检查

---

## 优先级问题列表

| 优先级 | 问题 | 影响 | 解决方案 |
|--------|------|------|----------|
| **P0** | Gateway 未运行 | 无法访问 Web UI | 启动 Gateway 服务 |
| **P0** | bind 配置为 loopback | 仅本机能访问 | 修改为 `lan` |
| **P1** | 缺少 controlUi 配置 | 浏览器显示断开 | 添加 controlUi 配置项 |
| **P1** | openclaw 不在 PATH | 命令无法执行 | 使用完整路径 |
| **P2** | 防火墙阻止端口 | 局域网无法访问 | 开放 18789 端口 |

---

## 快速检查脚本

```bash
#!/bin/bash
echo "=== OpenClaw 状态检查 ==="
echo ""
echo "1. 进程状态:"
ps aux | grep openclaw | grep -v grep
echo ""
echo "2. 服务状态:"
systemctl --user status openclaw-gateway.service --no-pager
echo ""
echo "3. 网关可达性:"
openclaw gateway probe
echo ""
echo "4. 局域网 IP:"
ip addr show | grep -E "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"
echo ""
echo "5. 防火墙状态:"
sudo ufw status | grep 18789
```

---

## Gateway 配置示例

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
      "dangerouslyDisableDeviceAuth": true
    }
  }
}
```

---

## 常用命令速查

| 命令 | 说明 |
|------|------|
| `openclaw status` | 查看完整状态 |
| `openclaw gateway probe` | 检查网关可达性 |
| `openclaw gateway restart` | 重启 Gateway |
| `openclaw logs --follow` | 实时查看日志 |
| `openclaw tui` | 打开终端界面 |
| `systemctl --user restart openclaw-gateway.service` | 重启服务 |
| `journalctl --user -u openclaw-gateway.service -f` | 查看服务日志 |

---

## 访问地址

```
http://<局域网IP>:18789/
```

获取 Token：
```bash
grep '"token"' ~/.openclaw/openclaw.json
```

---

## 安全建议

| 风险 | 建议 |
|------|------|
| 同一网络内任何人可访问 | 仅在受信任网络使用 |
| Token 可能被截获 | 定期轮换 Token |
| 公共 WiFi 下数据泄露 | 禁止在公共 WiFi 使用 |
| 未授权访问 | 生产环境启用设备认证 |

**生成新 Token：**
```bash
openssl rand -hex 32
```

---

## 配置文件位置

| 文件 | 路径 |
|------|------|
| 主配置 | `~/.openclaw/openclaw.json` |
| 日志文件 | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` |
| systemd 服务 | `~/.config/systemd/user/openclaw-gateway.service` |
| Agent 会话 | `~/.openclaw/agents/main/sessions/` |

---

**注意**：配置中的 token 等敏感信息请替换为你的实际值，切勿泄露。
