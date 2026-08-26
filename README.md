# VPS工具箱 - 服务器运维瑞士军刀

> 实用的 VPS 测速、优化、部署脚本集合，一键部署即用

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/CG-spring/vps-guide-8q3khk.svg?style=flat-square)](https://github.com/CG-spring/vps-guide-8q3khk/stargazers)
[![Tools](https://img.shields.io/badge/tools-5_major_scripts-green)]()
[![Auto Update](https://img.shields.io/badge/auto_update-daily-orange)]()

**[English](README_EN.md)** | 中文

---

## 📌 什么是 VPS 工具箱？

VPS 工具箱是一组精选的服务器运维脚本，专为需要快速搭建和优化 VPS 环境的中国用户设计。无论你是首次接触 Linux 服务器，还是需要批量管理多台机器，这套工具都能帮你节省大量时间。

每个脚本都经过实际生产环境验证，支持一键执行、自动配置开机启动，并兼容主流的境外 VPS 提供商（搬瓦工、CloudCone、 RackNerd 等）。

---

## 🧰 工具列表

| 脚本 | 功能简介 | 一键安装命令 |
|------|---------|------------|
| `bench.sh` | 服务器基准测试 | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/bench.sh \| bash` |
| `bbr.sh` | BBR/LotServer 加速 | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/bbr.sh \| bash` |
| `clash.sh` | 一键安装 Clash | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/clash.sh \| bash` |
| `docker.sh` | Docker + Compose | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/docker.sh \| bash` |
| `wireguard.sh` | WireGuard 隧道 | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/wireguard.sh \| bash` |

---

## 📊 bench.sh — 服务器基准测试

### 核心功能

- **系统信息采集**：CPU 型号/核心数、内存总量/已用/可用、SWAP 信息、硬盘型号与容量、系统运行时间
- **磁盘 I/O 测试**：使用 `dd` 命令测试 1GB 文件的顺序读写速度（两次测试取平均）
- **带宽测速**：自动选择就近节点测试上行/下行带宽，覆盖东京（亚洲）、法兰克福（欧洲）、迈阿密（北美）三大核心节点，并额外测试中国方向链路的延迟与可用性
- **路由追踪**：使用 `traceroute` 展示到各测速节点的完整路由路径，帮助判断网络质量
- **流媒体解锁检测**：智能检测 Netflix（各区）、Disney+、YouTube Premium、HBO Max、Hulu 等主流流媒体平台解锁状态

### 输表示例

```
=== CPU Info ===
Model: AMD EPYC 7282 16-Core Processor
Cores: 2 vCPU
=== Memory Info ===
Total: 1932 MB  Used: 287 MB  Available: 1578 MB
=== Disk I/O ===
WRITE: 520 MB/s
READ:  610 MB/s
=== Speedtest ===
Tokyo:   298.5 Mbps ↓ /  89.2 Mbps ↑
Frankfurt: 45.2 Mbps ↓ / 12.1 Mbps ↑
=== Netflix Unlock ===
Netflix: ✅ US/JP Full Access
Disney+: ✅ Available
YouTube: ✅ Premium OK
```

### 使用场景

- 购买 VPS 后快速评估机器性能
- 横向对比多家 VPS 提供商的线路质量
- 定期巡检服务器磁盘与网络状态

---

## 🚀 bbr.sh — TCP 拥塞控制加速

### 支持的加速算法

| 算法 | 适用场景 | 内核要求 | 推荐指数 |
|------|---------|---------|---------|
| **Google BBR** | 跨境高延迟线路 | Linux 4.9+ | ⭐⭐⭐⭐ |
| **BBRplus** | 丢包率高线路 | Linux 4.14+ | ⭐⭐⭐⭐⭐ |
| **LotServer** | 极端丢包环境 | CentOS/Debian | ⭐⭐⭐ |
| **锐速（ServerSpeeder）** | 老牌付费方案 | CentOS 6-7 | ⭐⭐ |

### 技术原理简介

BBR（Bottleneck Bandwidth and RTT）是 Google 于 2016 年发布的 TCP 拥塞控制算法。与传统的 Reno/Cubic 等基于丢包的算法不同，BBR 通过实时估算链路带宽与往返延迟的「瓶颈」，动态调整发送窗口，在高延迟和高丢包率环境下效果显著。

脚本会自动：
1. 检测当前系统内核版本与已启用的拥塞控制模块
2. 备份原有 sysctl 配置
3. 按选定算法写入 sysctl 参数
4. 验证是否生效（`sysctl net.ipv4.tcp_congestion_control`）

### 注意事项

- 部分服务商（如 AWS、Azure）禁止修改内核参数
- 切换算法后建议重启相关网络服务
- BBR 和 BBRplus 不能同时启用

---

## 🌐 clash.sh — Clash Meta 一键部署

### 支持的内核

- **mihomo（推荐）**：Clash.Meta 的活跃分支，原名 Clash.Meta，推荐使用。支持完整的 GEOIP、Script、Snell 等高级特性
- **Clash.Meta（兼容）**：功能全面的主流内核，社区生态丰富

### 安装特性

- 自动识别系统架构（amd64/arm64）并下载对应二进制
- 自动生成基础配置文件（包含常见节点/规则）
- 配置 Systemd 服务文件，支持 `systemctl start\|stop\|restart\|status clash`
- 自动配置开机启动（`systemctl enable clash`）
- 提供 `update.sh` 脚本用于热更新节点订阅

### 基础使用

```bash
# 安装完成后管理服务
systemctl status clash     # 查看状态
systemctl restart clash    # 重启（如更新订阅后）
journalctl -u clash -f     # 实时查看日志

# 编辑配置文件
vim /etc/clash/config.yaml

# 更新订阅
bash /opt/clash/update.sh
```

### Clash Dashboard（可选）

安装完成后可通过浏览器访问 `http://<VPS-IP>:9090/ui` 打开管理面板（默认关闭外部访问，建议配合防火墙规则或内网使用）。

---

## 🐳 docker.sh — Docker + Compose 部署

### 安装内容

- Docker Engine（最新稳定版）
- Docker Compose v2
- containerd.io 运行时
- 常用工具：`docker-compose`、`docker-credential-helper`

### 适用场景

- 快速在 VPS 上搭建个人服务（Nextcloud、AdGuard Home、rclone 等）
- 隔离运行多个服务，避免端口冲突
- 配合脚本批量管理容器

### 快速启动示例（AdGuard Home）

```bash
# 创建配置目录
mkdir -p /opt/adguard && cd /opt/adguard

# 创建 docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: "3"
services:
  adguard:
    image: adguard/adguardhome:latest
    container_name: adguard
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:80/tcp"
      - "8443:443/tcp"
    volumes:
      - ./work:/opt/adguard/work
      - ./conf:/opt/adguard/conf
    restart: unless-stopped
EOF

# 启动
docker-compose up -d

# 访问管理界面
# http://<VPS-IP>:3000
```

---

## 🔐 wireguard.sh — WireGuard VPN 搭建

### WireGuard 优势

- 现代加密协议：Curve25519 ChaCha20-Poly1305，性能远超 OpenVPN/IPSec
- 内核级集成：Linux 5.6+ 原生支持，延迟极低
- 配置简洁：公钥/私钥配对，无需复杂 PKI
- 移动端友好：支持热更新隧道配置不断线

### 脚本功能

- 自动生成服务器端密钥对
- 自动生成客户端密钥对（可指定数量 N，自动生成 N 份配置）
- 自动分配客户端 IP 段（默认 `10.0.0.0/24`，服务器 `10.0.0.1`）
- 写入服务端配置并启用 WireGuard 接口
- 生成可直接导入客户端的 `.conf` 文件（打印到终端，可复制）
- 配置 NAT 转发与防火墙规则

### 快速使用

```bash
# 默认安装（生成 1 个客户端配置）
bash wireguard.sh

# 生成 3 个客户端配置
bash wireguard.sh 3

# 卸载
bash wireguard.sh uninstall
```

---

## 💻 适用系统

| 操作系统 | 版本要求 | 架构支持 |
|---------|---------|---------|
| **Ubuntu** | 18.04+ | amd64, arm64 |
| **Debian** | 10+ | amd64, arm64 |
| **CentOS** | 7+ | amd64 |
| **Fedora** | 35+ | amd64 |

> ⚠️ **注意**：部分脚本（如 bbr.sh、wireguard.sh）需要 root 权限执行，请使用 `sudo -i` 切换到 root 用户后再运行。

---

## 🔧 常见问题

### Q: 运行脚本后没有反应怎么办？

```bash
# 查看详细输出
bash -x bench.sh

# 检查网络连通性
curl -I https://raw.githubusercontent.com
```

### Q: bench.sh 测速结果比预期低很多

- 可能是测速节点限速或 VPS 自身带宽被限制（部分商家的带宽为"突发带宽"）
- 建议多节点测试后取综合评估
- 中国方向测试受国际出口带宽影响较大属正常

### Q: clash.sh 安装后无法连接

- 检查防火墙是否放行 7890 端口（或脚本配置的端口）
- 确认节点订阅链接有效且未过期
- 查看日志：`journalctl -u clash -n 50`

### Q: Docker 安装失败

```bash
# 先卸载旧版本
apt remove docker docker-engine docker.io containerd runc
# 再重新运行
bash docker.sh
```

---

## 📈 推荐 VPS

本工具箱针对主流境外 VPS 优化，以下服务商经实测兼容性良好：

| 提供商 | 特点 | 推荐场景 |
|-------|------|---------|
| **搬瓦工（BandwagonHost）** | 稳定、CN2 GIA 线路 | 建站、高端需求 |
| **RackNerd** | 价格低、性价比高 | 入门、V2Ray/Clash 落地 |
| **CloudCone** | 按小时计费、灵活 | 测试、临时用途 |
| **DMIT** | 高端线路、CMI 优化 | 高质量需求 |
| **V.ps** | 日本/香港高端线路 | 低延迟需求 |

> 💡 **提示**：购买前可先用本工具箱中的 `bench.sh` 测试样机性能，确认线路质量再大规模部署。

---

## 🔗 相关资源

- [Clash.Meta 官方文档](https://wiki.metacubex.one/)
- [WireGuard 官方站](https://www.wireguard.com/)
- [Docker 官方文档](https://docs.docker.com/)
- [BBR 原理详解](https://github.com/google/bbr)

---

## 📄 许可证

本项目采用 [MIT License](LICENSE) 开源，欢迎 Fork 和贡献。

---

> 🛠️ 由 **ClashHub** 维护 · 脚本每日自动更新
>
> 📮 问题反馈：[GitHub Issues](https://github.com/CG-spring/vps-guide-8q3khk/issues)
>
> 🔄 最新版本：[Releases](https://github.com/CG-spring/vps-guide-8q3khk/releases)

