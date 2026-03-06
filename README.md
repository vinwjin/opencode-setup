# opencode-setup

> OpenClaw + WSL2 环境配置教程

本仓库包含 OpenClaw AI 助手在 WSL2 环境下的安装配置指南，以及解决网络代理问题的方案。

## 目录

1. **[OpenClaw 安装配置指南](./01-OpenClaw安装配置指南.md)** - 完整的安装和配置流程
2. **[OpenClaw 快速设置指南](./02-OpenClaw快速设置指南.md)** - 快速参考和问题排查
3. **[WSL2 Clash 网络配置](./03-WSL2-Clash网络配置.md)** - 解决代理导致 WSL 断网问题

## 快速开始

### 安装 OpenClaw

```bash
npm install -g openclaw
```

### 配置开机自启

```bash
sudo loginctl enable-linger $(whoami)
openclaw gateway install
systemctl --user enable --now openclaw-gateway.service
```

### 访问

获取局域网 IP：
```bash
ip addr show | grep "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"
```

访问 `http://<IP>:18789/`

## WSL2 网络问题

如果在使用 Clash 等代理软件时 WSL2 断网，请参考 [03-WSL2-Clash网络配置.md](./03-WSL2-Clash网络配置.md)。

## 注意事项

- 配置中的 Token、密码、API Key 等敏感信息请替换为实际值
- 切勿将包含敏感信息的配置提交到公开仓库
- 建议仅在受信任的网络环境中使用

## 相关链接

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)

---

**本项目仅供学习参考，配置前请确保理解各项设置的含义。**
