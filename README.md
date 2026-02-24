**English** | [中文](https://p3terx.com/archives/cloudflare-warp-configuration-script.html) | [فارسی](README_FA.md)

# Cloudflare WARP Installer (Optimized Fork)

A Bash script that automatically installs and configures Cloudflare WARP in Linux, connects to WARP networks with WARP official client or WireGuard. This fork is optimized for **low-resource servers** (1 CPU / 1GB RAM) with built-in system optimization, Cloudflare failure protection, and safe defaults.

## What's New in This Fork

### System Optimizer (`bash warp.sh optimize`)
A built-in "Advanced SystemCare for Linux" that prepares low-resource servers for WARP:
- **RAM Cleanup** -- drops filesystem caches, kills zombie processes, trims systemd journal logs
- **Swap Management** -- auto-creates a 512MB swap file if none exists (prevents OOM kills)
- **Kernel Tuning** -- applies `vm.vfs_cache_pressure=200`, `vm.min_free_kbytes=16384`, and other sysctl params optimized for low-memory systems
- **Bloat Service Removal** -- disables `snapd`, `ModemManager`, `accounts-daemon`, `unattended-upgrades`

### Auto Quick-Optimize
A lightweight RAM cleanup (cache flush + zombie kill) runs **automatically** before every WARP install, restart, and WireGuard start/restart. No manual action needed.

### Cloudflare Failure Protection
- **Retry limits** -- registration loops have a max retry count (default 5) with exponential backoff instead of spinning forever
- **Profile backup & restore** -- WGCF profiles are automatically backed up; if Cloudflare blocks new registrations, the script restores from backup
- **Dynamic endpoint resolution** -- resolves `engage.cloudflareclient.com` via DNS before using hardcoded IPs; auto-detects if Cloudflare changes their endpoint IPs
- **DNS validation** -- `dig`/`nslookup` output is validated with regex to prevent error messages from being used as IP addresses

### Optimized Defaults
All performance flags are **enabled by default** in this fork (upstream has them disabled):

| Setting | Upstream | This Fork | Impact |
|---------|----------|-----------|--------|
| `ENABLE_NETCACHE` | `0` | `1` | Stops WireGuard teardown on every status check |
| `ENABLE_FAST_MTU` | `0` | `1` | Binary search (~8 pings) instead of linear scan (~50 pings) |
| `ENABLE_FAST_ENDPOINT` | `0` | `1` | Faster endpoint detection |
| `Network_Status_Cache_TTL` | `5s` | `15s` | Cache doesn't expire before script finishes using it |
| `AUTO_HEAL_TIMER_INTERVAL` | `15min` | `30min` | Halves health check frequency |
| `AUTO_HEAL_HANDSHAKE_TIMEOUT` | `300s` | `600s` | Less aggressive restart trigger |
| `AUTO_HEAL_COOLDOWN` | `60s` | `120s` | Prevents restart storms on slow systems |
| curl timeouts | `2s` | `1s` | Faster failure detection |
| systemd auto-heal service | `Nice=10` | `Nice=10, MemoryMax=64M, CPUQuota=50%` | Prevents health checks from starving WARP |

## Features

- Automatically install Cloudflare WARP Official Linux Client
- Quickly enable WARP Proxy Mode, access WARP network with SOCKS5
- Automatically install WireGuard related components
- Configuration WARP IPv4 / IPv6 / Dual Stack / Non-Global Network (WireGuard Mode)
- Built-in system optimizer for low-resource servers
- Auto RAM cleanup before every WARP operation
- Health monitor with auto-heal via systemd timer
- Cloudflare registration failure protection with retry limits and profile backup
- Dynamic endpoint IP resolution with DNS fallback

## Requirements

### WARP Official Linux Client

Official WARP client support is currently limited to x86_64 platforms, see OS Support for details: https://pkg.cloudflareclient.com

### WARP WireGuard Network Mode

Supported distributions:

- Debian >= 10
- Ubuntu >= 16.04
- Fedora
- CentOS
- Oracle Linux
- Arch Linux
- Other similar distributions

Supported platform architecture:

- x86(i386)
- x86_64(amd64)
- ARMv8(aarch64)
- ARMv7(armhf)

## Quick Start (Low-Resource Server)

```bash
# 1. Download
git clone https://github.com/roshanayixyz/warp.sh.git
cd warp.sh
chmod +x warp.sh

# 2. Optimize your system first (frees RAM, creates swap, tunes kernel)
sudo bash warp.sh optimize

# 3. Setup WARP (use wg4 for safety -- keeps SSH accessible)
sudo bash warp.sh wg4

# 4. Install auto-heal (optional, keeps tunnel alive)
sudo bash warp.sh autoheal-install

# 5. Check status
sudo bash warp.sh status
```

## Usage

```bash
bash warp.sh [SUBCOMMAND]
```

### Subcommands

```
install             Install Cloudflare WARP Official Linux Client
uninstall           Uninstall Cloudflare WARP Official Linux Client
restart             Restart Cloudflare WARP Official Linux Client
proxy               Enable WARP Client Proxy Mode (default SOCKS5 port: 40000)
unproxy             Disable WARP Client Proxy Mode
wg                  Install WireGuard and related components
wg4                 WARP IPv4 Global Network (WireGuard) -- RECOMMENDED for VPS
wg6                 WARP IPv6 Global Network (WireGuard)
wgd                 WARP Dual Stack Global Network (WireGuard) -- WARNING: routes ALL traffic
wgx                 WARP Non-Global Network (WireGuard)
rwg                 Restart WARP WireGuard service
dwg                 Disable WARP WireGuard service
optimize            Optimize system for low-resource servers (RAM, swap, kernel, services)
autoheal-task       Run a single auto-heal cycle (used by the timer)
autoheal-install    Install the systemd auto-heal timer
autoheal-uninstall  Remove the systemd auto-heal timer
status              Print status information
version             Print version information
help                Print help message
menu                Chinese special features menu
```

## WARP Modes Explained

| Mode | Command | What It Does | SSH Safe? |
|------|---------|-------------|-----------|
| **IPv4 Only** | `wg4` | Routes IPv4 through WARP, IPv6 stays direct | Yes |
| **IPv6 Only** | `wg6` | Routes IPv6 through WARP, IPv4 stays direct | Yes |
| **Dual Stack** | `wgd` | Routes ALL traffic through WARP | **No** -- if tunnel breaks, SSH dies |
| **Non-Global** | `wgx` | Selective routing via fwmark | Yes |
| **Proxy** | `proxy` | SOCKS5 proxy on port 40000 | Yes |

> **Warning:** Do NOT use `wgd` on a remote VPS unless you have VNC/console access. If the tunnel breaks, you lose SSH access. Use `wg4` instead.

## System Optimizer Details

Running `sudo bash warp.sh optimize` performs:

### RAM Cleanup
- Flushes filesystem page cache, dentries, and inodes (`echo 3 > /proc/sys/vm/drop_caches`)
- Trims systemd journal logs to 16MB max
- Kills zombie processes
- Clears `/tmp` files older than 7 days

### Swap Management
- Checks if swap exists
- If no swap: creates a 512MB swap file at `/swapfile`
- Makes it persistent in `/etc/fstab`

### Kernel Tuning
Writes to `/etc/sysctl.d/99-warp-optimize.conf`:
```
vm.swappiness = 60
vm.vfs_cache_pressure = 200
vm.min_free_kbytes = 16384
vm.overcommit_memory = 1
net.core.somaxconn = 512
```

### Bloat Service Removal
Disables these services if present (safe to disable on headless VPS):
- `snapd` -- can consume 100-200MB RAM on Ubuntu
- `unattended-upgrades` -- causes CPU spikes during background apt updates
- `ModemManager` -- not needed on VPS
- `accounts-daemon` -- not needed on headless server

## Health Monitoring & Auto-Heal

The script tracks WireGuard handshake age, MTU, endpoint, and route snapshots. When `AUTO_HEAL_ENABLED=1`, the auto-heal system:

1. Checks handshake age (kernel query -- zero overhead)
2. Only pings if handshake is stale (avoids unnecessary network calls)
3. Restarts WireGuard if tunnel is unhealthy
4. Respects cooldown period to prevent restart storms

```bash
# Install auto-heal timer (runs every 30 minutes)
sudo bash warp.sh autoheal-install

# Remove it later
sudo bash warp.sh autoheal-uninstall
```

## Cloudflare Failure Protection

This fork protects against Cloudflare blocking or changing things:

| Risk | Protection |
|------|-----------|
| `wgcf register` blocked/rate-limited | Max 5 retries with exponential backoff, clear error message |
| `warp-cli register` fails | Max 5 retries with backoff, exits cleanly instead of hanging |
| Profile generation fails | Auto-restores from backup at `/etc/warp/wgcf-profile.conf.bak` |
| Endpoint IPs change | DNS resolution of `engage.cloudflareclient.com` before using hardcoded IPs |
| DNS resolver broken | Validates dig/nslookup output with regex; falls back to hardcoded IPs |

## Environment Variables

All variables can be set before running the script or in `/etc/default/warp-autoheal`.

| Variable | Default | Purpose |
|----------|---------|---------|
| `ENABLE_NETCACHE` | `1` | Cache network status to avoid WireGuard teardown on checks |
| `ENABLE_FAST_MTU` | `1` | Binary-search MTU probing (faster setup) |
| `ENABLE_FAST_ENDPOINT` | `1` | Faster endpoint detection with tighter timeouts |
| `AUTO_HEAL_ENABLED` | `1` | Toggle auto-heal logic globally |
| `AUTO_HEAL_HANDSHAKE_TIMEOUT` | `600` | Seconds before stale handshake triggers restart |
| `AUTO_HEAL_ROUTE_TARGET` | `1.1.1.1` | Target IP for connectivity checks |
| `AUTO_HEAL_COOLDOWN` | `120` | Minimum seconds between auto-heal attempts |
| `AUTO_HEAL_TIMER_INTERVAL` | `30min` | Systemd timer frequency |
| `REGISTER_MAX_RETRIES` | `5` | Max registration attempts before giving up |
| `REGISTER_RETRY_DELAY` | `3` | Base delay between retries (multiplied by attempt number) |
| `LOG_WITH_TIMESTAMP` | `1` | Prefix log lines with timestamps |

## Troubleshooting

### Server frozen after `wgd`
`wgd` routes ALL traffic through WARP. If the tunnel fails, SSH dies.
1. Access your VPS via **VNC console** from your provider's web panel
2. Run: `wg-quick down wgcf && systemctl disable wg-quick@wgcf`
3. Fix DNS: `echo "nameserver 8.8.8.8" > /etc/resolv.conf`
4. Use `wg4` instead of `wgd`

### DNS resolution failing
If you see `communications error to 127.0.0.53#53: timed out`:
```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
systemctl restart systemd-resolved
```

### Registration keeps failing
Cloudflare may be rate-limiting `wgcf`. The script will retry 5 times then stop. Options:
- Wait a few hours and try again
- Use an existing profile: place it at `/etc/warp/wgcf-profile.conf`

### High memory usage
```bash
sudo bash warp.sh optimize   # run anytime to free RAM
```

## Examples

```bash
# Optimize system + setup WARP IPv4 (recommended for VPS)
sudo bash warp.sh optimize
sudo bash warp.sh wg4

# SOCKS5 proxy mode on port 40000
sudo bash warp.sh proxy

# IPv6 access for your server
sudo bash warp.sh wg6

# Just install WireGuard (no WARP config)
sudo bash warp.sh wg

# Check everything
sudo bash warp.sh status
```

## Credits

- [Cloudflare WARP](https://1.1.1.1/)
- [WireGuard](https://www.wireguard.com/)
- [ViRb3/wgcf](https://github.com/ViRb3/wgcf)
- [P3TERX/warp.sh](https://github.com/P3TERX/warp.sh) -- original project

## License

[MIT](https://github.com/P3TERX/warp.sh/blob/main/LICENSE) (c) [P3TERX](https://p3terx.com/)

## Notice of Non-Affiliation and Disclaimer

We are not affiliated, associated, authorized, endorsed by, or in any way officially connected with Cloudflare, or any of its subsidiaries or its affiliates. The official Cloudflare website can be found at https://www.cloudflare.com/.

The names Cloudflare Warp and Cloudflare as well as related names, marks, emblems and images are registered trademarks of their respective owners.
