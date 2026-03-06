# WSL2 网络问题：Clash 导致断网解决方案

> 解决 WSL2 在使用 Clash Verge 时无法访问外网的问题

---

## 问题描述

在 Windows 上开启 Clash Verge 的 **TUN 模式**（虚拟网卡模式）时，WSL2 会断网，无法访问外网。

---

## 原因分析

| 模式 | 原理 | WSL2 兼容 |
|------|------|-----------|
| **TUN 模式** | 创建虚拟网卡接管所有系统流量 | ❌ 不兼容 |
| **系统代理模式** | 只影响本机应用流量 | ✅ 兼容 |

- **TUN 模式**：Clash 创建虚拟网卡接管所有系统流量，但 WSL2 有独立的网络 namespace，不走 Windows 的虚拟网卡
- **系统代理模式**：只影响 Windows 本机的流量，WSL 可以通过配置代理环境变量连上网

---

## 解决方案

### 步骤 1：Clash 配置允许 LAN 访问

编辑配置文件：
```
C:\Users\<用户名>\AppData\Roaming\io.github.clash-verge-rev.clash-verge-rev\clash-verge.yaml
```

修改：
```yaml
allow-lan: true
```

### 步骤 2：WSL 设置代理环境变量

在 WSL 终端执行：
```bash
# 添加到 ~/.bashrc（持久化）
echo 'export http_proxy=http://<Windows-IP>:7897
export https_proxy=http://<Windows-IP>:7897' >> ~/.bashrc

# 立即生效
export http_proxy=http://<Windows-IP>:7897
export https_proxy=http://<Windows-IP>:7897
```

其中 `<Windows-IP>` 是 Windows 在局域网的 IP（例如 `10.15.164.76`）。

**获取 Windows IP：**
```powershell
# 在 Windows PowerShell 中执行
ipconfig | Select-String "IPv4"
```

### 步骤 3：Windows 防火墙放行

在 Clash Verge 设置中开启"允许局域网连接"（Allow LAN）。

或以管理员身份运行 PowerShell：
```powershell
New-NetFirewallRule -DisplayName "Clash-Verge-LAN" -Direction Inbound -Protocol TCP -LocalPort 7897 -Action Allow
```

### 步骤 4：使用系统代理模式

- ❌ **不要开启 TUN 模式**（会导致 WSL 断网）
- ✅ **使用系统代理模式**（配合环境变量，WSL 可正常上网）

---

## 验证

在 WSL 中测试：
```bash
# 测试代理连接
curl -x http://10.15.164.76:7897 https://www.google.com

# 或直接测试外网（已配置环境变量）
curl https://www.google.com
```

返回 HTTP 200 即表示成功。

---

## 关键配置总结

| 设置项 | 推荐值 |
|--------|--------|
| Clash 工作模式 | 系统代理（System Proxy） |
| allow-lan | true |
| 代理端口 | 7897（mixed-port） |
| WSL 代理变量 | http_proxy / https_proxy |

---

## 常见问题

### Q1: 代理能连但无法访问外网
- 检查 Clash 是否正常运行
- 检查代理端口是否正确（默认 7897）
- 确认 Clash 已经选择代理节点

### Q2: 开启 TUN 后 WSL 断网
- 这是已知问题，WSL2 不支持 TUN 模式的虚拟网卡
- 建议保持系统代理模式

### Q3: 代理超时
- 检查防火墙是否放行 7897 端口
- 检查 Windows 防火墙设置

---

## 原理补充

当 WSL2 访问网络时：
1. 不经过 Windows 的物理网卡
2. 有自己独立的网络栈（eth0 通过虚拟交换机连接）
3. TUN 模式的虚拟网卡只拦截 Windows 本机的流量
4. 所以 WSL2 的流量不会走 Clash 代理，需要手动配置环境变量

---

**注意**：本文中的 IP 地址（10.15.164.76）和端口（7897）仅为示例，请替换为你实际的值。
