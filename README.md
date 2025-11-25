**English** | [中文](https://p3terx.com/archives/cloudflare-warp-configuration-script.html)

# Cloudflare WARP Installer

A Bash script that automatically installs and configures CloudFlare WARP in Linux, connects to WARP networks with WARP official client or WireGuard.

## Features

- Automatically install CloudFlare WARP Official Linux Client
- Quickly enable WARP Proxy Mode, access WARP network with SOCKS5
- Automatically install WireGuard related components
- Configuration WARP IPv4 Network interface (WireGuard Mode)
- Configuration WARP IPv6 Network interface (WireGuard Mode)
- Configuration WARP Dual Stack Network interface (WireGuard Mode)
- Optional performance toggles (network cache / MTU probe) with safe fallbacks
- Built-in health monitor that can automatically restart WARP/WireGuard
- Systemd timer integration for hands-off auto-heal
- ...

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

## Usage

```bash
bash <(curl -fsSL git.io/warp.sh) [SUBCOMMAND]
# or
wget git.io/warp.sh
bash warp.sh [SUBCOMMAND]
```

### Subcommands

```
install         Install Cloudflare WARP Official Linux Client
uninstall       uninstall Cloudflare WARP Official Linux Client
restart         Restart Cloudflare WARP Official Linux Client
proxy           Enable WARP Client Proxy Mode (default SOCKS5 port: 40000)
unproxy         Disable WARP Client Proxy Mode
wg              Install WireGuard and related components
wg4             Configuration WARP IPv4 Global Network (with WireGuard), all IPv4 outbound data over the WARP network
wg6             Configuration WARP IPv6 Global Network (with WireGuard), all IPv6 outbound data over the WARP network
wgd             Configuration WARP Dual Stack Global Network (with WireGuard), all outbound data over the WARP network
wgx             Configuration WARP Non-Global Network (with WireGuard), set fwmark or interface IP Address to use the WARP network
rwg             Restart WARP WireGuard service
dwg             Disable WARP WireGuard service
autoheal-task   Run a single auto-heal cycle (used by the timer)
autoheal-install    Install the systemd timer that runs auto-heal every few minutes
autoheal-uninstall  Remove the systemd timer
status          Prints status information
version         Prints version information
help            Prints this message or the help of the given subcommand(s)
menu            Chinese special features menu
```

### Health monitoring & auto-heal

- By default the script keeps tracking the latest WireGuard handshake age, MTU, endpoint and route snapshot.  
- When `AUTO_HEAL_ENABLED=1`, every status check (or the optional systemd timer) automatically restarts `wg-quick@wgcf` if:
  - No handshake has been observed for `AUTO_HEAL_HANDSHAKE_TIMEOUT` seconds (default 300s)
  - A simple connectivity probe to `AUTO_HEAL_ROUTE_TARGET` (default `1.1.1.1`) fails
- The timer interval can be controlled via `AUTO_HEAL_TIMER_INTERVAL` (default `15min`).

To keep the tunnel healthy without manual intervention:

```bash
# run from the directory that contains warp.sh
sudo bash warp.sh autoheal-install
# systemd now runs `warp.sh autoheal-task` every AUTO_HEAL_TIMER_INTERVAL

# to remove it later
sudo bash warp.sh autoheal-uninstall
```

### Environment toggles

All toggles can be provided as environment variables before invoking the script or by editing `/etc/default/warp-autoheal` after installing the timer.

| Variable | Default | Purpose |
|----------|---------|---------|
| `ENABLE_NETCACHE` | `0` | When `1`, avoid stopping WireGuard for every connectivity probe (faster but newer behavior). |
| `ENABLE_FAST_MTU` | `0` | When `1`, use binary-search MTU probing; otherwise use the original step-down loop. |
| `ENABLE_FAST_ENDPOINT` | `0` | When `1`, use faster endpoint detection with tighter timeouts. |
| `AUTO_HEAL_ENABLED` | `1` | Toggle the auto-heal logic globally. |
| `AUTO_HEAL_HANDSHAKE_TIMEOUT` | `300` | Seconds before a stale handshake triggers a restart. |
| `AUTO_HEAL_ROUTE_TARGET` | `1.1.1.1` | Target IP used for connectivity checks. |
| `AUTO_HEAL_COOLDOWN` | `60` | Minimum seconds between auto-heal attempts. |
| `AUTO_HEAL_TIMER_INTERVAL` | `15min` | systemd timer frequency (when installed). |
| `LOG_WITH_TIMESTAMP` | `1` | Prefix log lines with timestamps to simplify troubleshooting. |

### Example

- Install and automatically configure the Proxy Mode feature of the WARP client, enable the local loopback port 40000, and use an application that supports SOCKS5 to connect to this port.
    ```
    bash <(curl -fsSL git.io/warp.sh) proxy
    ```

- Install and automatically configure WARP IPv6 Network (with WireGuard)，Giving your Linux server access to IPv6 networks.
    ```
    bash <(curl -fsSL git.io/warp.sh) wg6
    ```

- This Bash script is also a good WireGuard installer.
    ```
    bash <(curl -fsSL git.io/warp.sh) wg
    ```

## Credits

- [Cloudflare WARP](https://1.1.1.1/)
- [WireGuard](https://www.wireguard.com/)
- [ViRb3/wgcf](https://github.com/ViRb3/wgcf)

## License

[MIT](https://github.com/P3TERX/warp.sh/blob/main/LICENSE) © **[P3TERX](https://p3terx.com/)**

## Notice of Non-Affiliation and Disclaimer

We are not affiliated, associated, authorized, endorsed by, or in any way officially connected with Cloudflare, or any of its subsidiaries or its affiliates. The official Cloudflare website can be found at https://www.cloudflare.com/.

The names Cloudflare Warp and Cloudflare as well as related names, marks, emblems and images are registered trademarks of their respective owners.
