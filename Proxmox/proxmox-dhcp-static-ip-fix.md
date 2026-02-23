# Proxmox VE: Fix DHCP Static IP Assignment Issues

## Problem

OPNsense DHCP static mapping not being assigned correctly to a Proxmox VE host. The lease shows as "abandoned" or the wrong IP is assigned despite having a static reservation.

## Solution

Temporarily switch Proxmox to DHCP mode to force a fresh lease, then revert to static configuration.

### Steps

#### 1. SSH to the Proxmox server

```bash
ssh root@<proxmox-ip>
```

#### 2. Edit the network interfaces file

```bash
nano /etc/network/interfaces
```

#### 3. Temporarily change from static to DHCP

Find the bridge interface configuration and change:

```
# Before
iface vmbr0 inet static
    address 10.10.20.11/24
    gateway 10.10.20.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0

# After (temporary)
iface vmbr0 inet dhcp
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
```

Save the file (Ctrl+O, Enter, Ctrl+X).

#### 4. Restart networking

```bash
systemctl restart networking
```

#### 5. Check the assigned IP in OPNsense

1. Go to **Services > ISC DHCPv4 > Leases**
2. Find the Proxmox server by MAC address or hostname
3. Note the dynamically assigned IP
4. If needed, create/update the static mapping for this MAC address with the desired IP

#### 6. Revert to static configuration

Edit the interfaces file again:

```bash
nano /etc/network/interfaces
```

Change back to static:

```
iface vmbr0 inet static
    address 10.10.20.11/24
    gateway 10.10.20.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
```

Save the file.

#### 7. Restart networking again

```bash
systemctl restart networking
```

#### 8. Verify connectivity

```bash
ip addr show vmbr0
ping -c 3 10.10.20.1
```

## Why This Works

This process forces the DHCP server to:
1. Issue a fresh lease and clear any stale/abandoned lease state
2. Register the correct MAC address association
3. Allow you to verify the static mapping is working before reverting

Once the static mapping is confirmed in OPNsense, switching back to static configuration on Proxmox ensures the server always uses the expected IP, even if DHCP is unavailable.

---

*Last updated: February 2026*
