# VPS Toolkit — Your Server Operations Swiss Army Knife

> A curated collection of one-click VPS testing, optimization, and deployment scripts for Linux servers

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/CG-spring/vps-guide-8q3khk.svg?style=flat-square)](https://github.com/CG-spring/vps-guide-8q3khk/stargazers)
[![Tools](https://img.shields.io/badge/tools-5_major_scripts-green)]()
[![Auto Update](https://img.shields.io/badge/auto_update-daily-orange)]()

**English** | [中文](README.md)

---

## What Is the VPS Toolkit?

The VPS Toolkit is a carefully curated suite of Linux server management scripts designed specifically for users who need to quickly set up and optimize VPS environments. Whether you're managing your first Linux server or deploying dozens of machines, this toolkit helps you get things done with a single command.

All scripts have been validated in real production environments, support one-click execution, auto-configure systemd startup, and are fully compatible with popular overseas VPS providers (BandwagonHost, CloudCone, RackNerd, DMIT, etc.).

---

## Tool Overview

| Script | Description | One-Line Install |
|--------|-------------|----------------|
| `bench.sh` | Server benchmark & speed test | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/bench.sh \| bash` |
| `bbr.sh` | BBR/LotServer acceleration | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/bbr.sh \| bash` |
| `clash.sh` | One-click Clash Meta install | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/clash.sh \| bash` |
| `docker.sh` | Docker + Compose setup | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/docker.sh \| bash` |
| `wireguard.sh` | WireGuard VPN setup | `wget -qO- https://raw.githubusercontent.com/CG-spring/vps-guide-8q3khk/main/wireguard.sh \| bash` |

---

## bench.sh — Server Benchmark & Network Speed Test

### What It Measures

- **System Info**: CPU model/core count, total/used/available RAM, SWAP details, disk model & capacity, system uptime
- **Disk I/O**: Sequential read/write performance via `dd` on a 1GB file (averaged over two runs)
- **Bandwidth**: Download/upload speeds via nearest servers — Tokyo (Asia), Frankfurt (Europe), Miami (North America) — plus China-routed latency check
- **Traceroute**: Route path to each speedtest node, helping you diagnose routing quality
- **Streaming Unlock Detection**: Smart detection for Netflix (multi-region), Disney+, YouTube Premium, HBO Max, Hulu, and other popular streaming services

### Sample Output

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
Tokyo:       298.5 Mbps ↓ /  89.2 Mbps ↑
Frankfurt:    45.2 Mbps ↓ /  12.1 Mbps ↑
Miami:        52.1 Mbps ↓ /  11.8 Mbps ↑
=== Netflix Unlock ===
Netflix:  ✅ US/JP Full Access
Disney+: ✅ Available
YouTube: ✅ Premium OK
```

### When to Use It

- Immediately after purchasing a new VPS to evaluate hardware performance
- Comparing routing quality across multiple providers before committing
- Periodic server health checks for disk and network status

---

## bbr.sh — TCP Congestion Control Acceleration

### Supported Algorithms

| Algorithm | Best For | Kernel Req | Rating |
|-----------|----------|-----------|--------|
| **Google BBR** | High-latency cross-border links | Linux 4.9+ | ⭐⭐⭐⭐ |
| **BBRplus** | High packet-loss links | Linux 4.14+ | ⭐⭐⭐⭐⭐ |
| **LotServer** | Extreme packet loss | CentOS/Debian | ⭐⭐⭐ |
| **ServerSpeeder** | Legacy paid solution | CentOS 6-7 | ⭐⭐ |

### How It Works

BBR (Bottleneck Bandwidth and RTT) is a TCP congestion control algorithm released by Google in 2016. Unlike traditional丢包-based algorithms (Reno/Cubic), BBR dynamically estimates the link's bandwidth-delay product and continuously adjusts the sending window. The result is dramatically better throughput on high-latency, lossy links — the exact conditions typical of cross-border VPS connections.

The script automatically:
1. Detects your current kernel version and active congestion control modules
2. Backs up your existing sysctl configuration
3. Writes the appropriate parameters for your chosen algorithm
4. Verifies activation (`sysctl net.ipv4.tcp_congestion_control`)

### Caveats

- Some providers (AWS, Azure) prohibit modifying kernel parameters — check your TOS
- Restart relevant network services after switching algorithms
- BBR and BBRplus cannot coexist; only one can be active at a time

---

## clash.sh — Clash Meta One-Click Installer

### Supported Kernels

- **mihomo (Recommended)**: The active fork of Clash.Meta (formerly known as Clash.Meta). Supports full GEOIP, scripting, Snell, and other advanced features.
- **Clash.Meta (Legacy)**: Feature-rich mainstream kernel with an extensive community ecosystem.

### What Gets Installed

- Binary compiled for your architecture (amd64/arm64 auto-detected)
- A functional starter `config.yaml` with common proxies and rules pre-loaded
- Systemd service file for `systemctl start|stop|restart|status clash`
- Auto-start on boot (`systemctl enable clash`)
- An `update.sh` helper for hot-reloading subscription feeds

### Basic Usage

```bash
# Post-install management
systemctl status clash     # Check service status
systemctl restart clash    # Restart after updating subscription
journalctl -u clash -f     # Tail live logs

# Edit config
vim /etc/clash/config.yaml

# Refresh subscription
bash /opt/clash/update.sh
```

### Web Dashboard (Optional)

After installation, access the management UI at `http://<VPS-IP>:9090/ui` (external access is disabled by default; use firewall rules or restrict to internal networks for security).

---

## docker.sh — Docker + Compose Setup

### What Gets Installed

- Docker Engine (latest stable release)
- Docker Compose v2
- containerd.io runtime
- Utilities: `docker-compose`, `docker-credential-helper`

### Common Use Cases

- Deploying personal services on a VPS: Nextcloud, AdGuard Home, rclone, etc.
- Isolating multiple services to avoid port conflicts
- Batch management of containers via scripts

### Quick Start: AdGuard Home

```bash
# Create config directory
mkdir -p /opt/adguard && cd /opt/adguard

# Create docker-compose.yml
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

# Launch
docker-compose up -d

# Access admin panel at http://<VPS-IP>:3000
```

---

## wireguard.sh — WireGuard VPN Setup

### Why WireGuard?

- **Modern cryptography**: Curve25519 + ChaCha20-Poly1305 — dramatically faster than OpenVPN or IPSec
- **Kernel-native**: Linux 5.6+ includes WireGuard as a built-in module, yielding ultra-low latency
- **Minimal config**: Public/private key pairs, no complex PKI required
- **Mobile-friendly**: Seamless tunnel updates without dropping connections

### What the Script Does

- Generates server-side key pair automatically
- Generates client key pair(s) (supply N to generate N client configs)
- Assigns client IP range (default: `10.0.0.0/24`, server at `10.0.0.1`)
- Writes server config and brings up the WireGuard interface
- Outputs ready-to-import `.conf` files for clients (printed to terminal for easy copying)
- Configures NAT forwarding and firewall rules

### Quick Usage

```bash
# Default: install with 1 client config
bash wireguard.sh

# Generate 3 client configs
bash wireguard.sh 3

# Uninstall
bash wireguard.sh uninstall
```

---

## Supported Operating Systems

| OS | Minimum Version | Architectures |
|----|----------------|--------------|
| **Ubuntu** | 18.04+ | amd64, arm64 |
| **Debian** | 10+ | amd64, arm64 |
| **CentOS** | 7+ | amd64 |
| **Fedora** | 35+ | amd64 |

> ⚠️ **Important**: Some scripts (bbr.sh, wireguard.sh) require root privileges. Run `sudo -i` to switch to root before executing.

---

## Frequently Asked Questions

### Q: Script runs but produces no output?

```bash
# Run with debug output
bash -x bench.sh

# Check network connectivity
curl -I https://raw.githubusercontent.com
```

### Q: Speedtest results are much lower than expected?

- The speedtest node may be rate-limited, or your VPS plan may cap bandwidth (many "burstable" plans throttle after initial high-speed bursts)
- Test multiple nodes and compare the averages
- China-routed tests are affected by international exit bandwidth — lower results here are normal

### Q: Can't connect after clash.sh installation?

- Verify firewall allows traffic on port 7890 (or the configured proxy port)
- Confirm your subscription URL is valid and not expired
- Check logs: `journalctl -u clash -n 50`

### Q: Docker installation fails?

```bash
# Remove old installations first
apt remove docker docker-engine docker.io containerd runc
# Then retry
bash docker.sh
```

---

## Recommended VPS Providers

The toolkit is optimized for popular overseas VPS providers. The following have been tested and work reliably:

| Provider | Highlights | Best For |
|----------|-----------|---------|
| **BandwagonHost** | Stable, CN2 GIA routes | Production websites, premium use |
| **RackNerd** | Budget-friendly, great value | Beginners, V2Ray/Clash endpoints |
| **CloudCone** | Hourly billing, flexible | Testing, temporary workloads |
| **DMIT** | Premium routes, CMI-optimized | High-quality connectivity |
| **V.ps** | Japan/HK premium lines | Low-latency requirements |

> 💡 **Tip**: Before committing to a provider, use `bench.sh` to test a trial instance's routing and performance.

---

## Related Resources

- [Clash.Meta Documentation](https://wiki.metacubex.one/)
- [WireGuard Official Site](https://www.wireguard.com/)
- [Docker Documentation](https://docs.docker.com/)
- [BBR Algorithm Explained](https://github.com/google/bbr)

---

## License

This project is open source under the [MIT License](LICENSE). Fork and contributions are welcome!

---

> 🛠️ Maintained by **ClashHub** · Scripts auto-update daily
>
> 📮 Issues: [GitHub Issues](https://github.com/CG-spring/vps-guide-8q3khk/issues)
>
> 🔄 Releases: [Releases](https://github.com/CG-spring/vps-guide-8q3khk/releases)

