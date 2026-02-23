# Server Deployment Documentation

A collection of guides and documentation for server infrastructure, applications, and networking.

## Directory Structure

| Directory | Description |
|-----------|-------------|
| **Applications/** | Self-hosted application deployments (Jitsi, Zammad, Tactical RMM) |
| **Infrastructure/** | Hardware and power management (UPS, NUT) |
| **Nextcloud/** | Nextcloud configuration and incident reports |
| **Proxmox/** | Proxmox VE and PBS guides, upgrades, and troubleshooting |
| **Starlink/** | Starlink setup with OPNsense, CGNAT workarounds |
| **Ubuntu/** | Ubuntu server administration (LVM, SSH, Zsh) |
| **Univention/** | Univention Corporate Server (UCS) guides |
| **VoIP/** | Voice over IP systems (FusionPBX) |
| **WireGuard/** | WireGuard VPN setup and configuration |

## Quick Links

### Proxmox
- [PVE Upgrade Fix Guide](Proxmox/pve-upgrade-fix-guide.md) - Switch from Enterprise to No-Subscription repo
- [PBS Upgrade Fix Guide](Proxmox/pbs-upgrade-fix-guide.md) - Proxmox Backup Server repo fix
- [DHCP Static IP Fix](Proxmox/proxmox-dhcp-static-ip-fix.md) - Fix DHCP/static IP assignment issues

### Networking
- [Starlink Bypass Mode Setup](Starlink/starlink-bypass-opnsense-setup.md) - OPNsense with Starlink CGNAT
- [WireGuard Ubuntu Server](WireGuard/wireguard-ubuntu-server-setup.md) - Full WireGuard setup guide

### Applications
- [Jitsi Docker Setup](Applications/jitsi-docker-setup.md) - Self-hosted video conferencing
- [Zammad Docker Setup](Applications/zammad-docker-setup.md) - Helpdesk ticketing system
- [Tactical RMM Agent](Applications/tactical-rmm-agent-ubuntu.md) - Remote monitoring agent

### Ubuntu Administration
- [LVM Root Resize](Ubuntu/ubuntu-lvm-root-resize.md) - Expand root filesystem after disk resize
- [SSH Key Setup](Ubuntu/ssh-copy-id-guide.md) - Passwordless SSH authentication
- [Zsh + Oh My Zsh](Ubuntu/zsh-oh-my-zsh-setup.md) - Server-safe shell setup

### Infrastructure
- [UPS NUT Eaton Setup](Infrastructure/ups-nut-eaton-setup.md) - Auto shutdown on power loss

---

*Last updated: February 2026*
