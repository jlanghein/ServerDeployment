# Starlink Bypass Mode with OPNsense Router Setup

## Overview

This guide covers migrating from:
- **Current**: Comcast -> OPNsense -> LAN
- **Target**: Starlink (Bypass Mode) -> OPNsense -> LAN

### Hardware
- Starlink Gen 3 (Standard) - Rectangular dish with compact router
- OPNsense router

---

## Step 1: Prepare Before Switching

### 1.1 Document Current Configuration
Before making any changes, document your existing OPNsense settings:

1. **Export OPNsense configuration backup**
   - Go to **System > Configuration > Backups**
   - Click **Download configuration**
   - Save the XML file safely

2. **Document your current settings**:
   - WAN interface settings (IP assignment method)
   - Port forwards (especially WireGuard VPN ports)
   - Firewall rules
   - NAT rules
   - Static routes (if any)

### 1.2 Document Comcast Port Forwards (To Be Removed)
The following port forwards exist on the Comcast router that will no longer apply:

| Name          | Protocol | External Port | Internal Port | Internal IP  |
|---------------|----------|---------------|---------------|--------------|
| opensense-dev | TCP/UDP  | 51822         | 51822         | 10.1.10.30   |

> **Note**: These forwards worked because Comcast provided a public IP. With Starlink CGNAT, 
> direct port forwarding will not work without a workaround (see Step 6).

### 1.3 Current OPNsense Firewall Rules

#### LAN Rules
| Protocol | Source  | Port | Destination | Port | Gateway | Description                          |
|----------|---------|------|-------------|------|---------|--------------------------------------|
| IPv4 *   | LAN net | *    | *           | *    | *       | Default allow LAN to any rule        |
| IPv6 *   | LAN net | *    | *           | *    | *       | Default allow LAN IPv6 to any rule   |

#### WAN Rules
| Protocol   | Source | Port | Destination   | Port  | Gateway | Description                    |
|------------|--------|------|---------------|-------|---------|--------------------------------|
| IPv4+6 UDP | *      | *    | This Firewall | 51822 | *       | Allow wireguard from Stately   |

> **Note**: The WAN rule "Allow wireguard from Stately" currently works because Comcast 
> provides a public IP and forwards port 51822. With Starlink CGNAT, inbound connections 
> to this port will not reach your firewall without a workaround.

### 1.4 Current NAT Outbound Configuration
- **Mode**: Automatic outbound NAT rule generation (no manual rules)

> This is the default OPNsense setting and should continue to work with Starlink.

### 1.5 Note Your WireGuard Configuration
Record your current WireGuard setup:
- Listening port: `51822`
- WAN firewall rule: "Allow wireguard from Stately" (UDP 51822)
- Peer configurations
- Allowed IPs

### 1.6 Current OPNsense Interface Overview (Before Migration)
| Interface | Device | Type   | IPv4            | IPv6                                      | Gateway       |
|-----------|--------|--------|-----------------|-------------------------------------------|---------------|
| WAN       | igb0   | DHCP   | 192.168.1.177/24| 2605:59c8:6301:f208:2e0:67ff:fe31:ab8a/64 | 192.168.1.1   |
| LAN       | igb1   | Static | 10.10.20.1/24   | 2605:59c8:6301:f20c:2e0:67ff:fe31:ab8b/62 | -             |
| wg1       | -      | -      | 10.10.19.2      | -                                         | -             |

> WAN was receiving private IP from Comcast router (double NAT situation).

---

## Step 2: Set Starlink Router to Bypass Mode

### 2.1 Connect to Starlink App
1. Open the **Starlink app** on your phone
2. Ensure your phone is connected to the Starlink WiFi network
3. Go to **Settings** (gear icon)

### 2.2 Enable Bypass Mode
1. Navigate to **Network** settings
2. Look for **Bypass Mode** option
3. Toggle **Bypass Mode** to **ON**

> **Important**: When bypass mode is enabled:
> - The Starlink router's WiFi will be disabled
> - The Starlink router's DHCP server will be disabled
> - The Starlink router becomes a simple pass-through device
> - Your OPNsense will receive the public-facing IP directly

### 2.3 What Bypass Mode Does
- Disables NAT on the Starlink router
- Passes the CGNAT or public IP directly to your connected device
- Disables WiFi on the Starlink router
- Only ONE device can connect to the Starlink router Ethernet port

---

## Step 3: Physical Connection

### 3.1 Cable Setup
1. **Disconnect** the Comcast connection from OPNsense WAN port
2. **Connect** an Ethernet cable from the **Starlink router's Ethernet port** to your **OPNsense WAN interface**

### 3.2 Connection Diagram
```
[Starlink Dish] 
      |
      | (Proprietary cable)
      v
[Starlink Router] (Bypass Mode)
      |
      | (Ethernet)
      v
[OPNsense WAN Port]
      |
      | (LAN)
      v
[Your Network/Switch]
```

---

## Step 4: Configure OPNsense WAN Interface

### 4.1 Access OPNsense
1. Connect to OPNsense via LAN (your current local network should still work)
2. Navigate to **Interfaces > WAN**

### 4.2 Configure WAN for DHCP
1. Set **IPv4 Configuration Type**: `DHCP`
2. Set **IPv6 Configuration Type**: `DHCPv6` (Starlink supports IPv6)
3. Under **DHCP Client Configuration**:
   - Hostname: leave blank or set your preference
   - Reject leases from: leave blank

### 4.3 Save and Apply
1. Click **Save**
2. Click **Apply Changes**
3. OPNsense should obtain an IP from Starlink

---

## Step 5: Verify Connectivity

### 5.1 Check WAN Status
1. Go to **Interfaces > Overview**
2. Verify WAN interface shows an IP address
   - Starlink typically assigns CGNAT IPs (100.x.x.x range)
   - Some residential plans may get public IPs

### 5.1.1 Actual Post-Migration Interface Overview
| Interface | Device | Type   | IPv4              | IPv6                                      | Gateway       |
|-----------|--------|--------|-------------------|-------------------------------------------|---------------|
| WAN       | igb0   | DHCP   | 100.124.179.91/10 | 2605:59c8:6100:fd85:2e0:67ff:fe31:ab8a/64 | 100.64.0.1    |
| LAN       | igb1   | Static | 10.10.20.1/24     | fe80::2e0:67ff:fe31:ab8b/64               | -             |
| wg1       | -      | -      | 10.10.19.2        | -                                         | -             |

**Confirmed:**
- **IPv4**: CGNAT address (`100.124.179.91`) - inbound IPv4 connections blocked by Starlink
- **IPv6**: Public address (`2605:59c8:6100:fd85:...`) - can accept inbound connections
- **Gateway**: `100.64.0.1` (Starlink CGNAT gateway)

### 5.2 Test Internet Connectivity
1. Go to **Interfaces > Diagnostics > Ping**
2. Ping `8.8.8.8` or `1.1.1.1`
3. Verify successful responses

### 5.3 Check DNS Resolution
1. Ping `google.com`
2. Ensure DNS is resolving correctly

---

## Step 6: Reconfigure Port Forwards (WireGuard VPN)

### 6.1 Important CGNAT Consideration

> **Warning**: Starlink uses **Carrier-Grade NAT (CGNAT)** for most residential customers. This means:
> - You do NOT have a public IP address
> - **Inbound port forwarding will NOT work** directly
> - You cannot host services accessible from the internet without workarounds

### 6.2 Check If You Have CGNAT
1. Go to **Interfaces > Overview** in OPNsense
2. Check your WAN IP address
3. If it starts with `100.x.x.x` (specifically 100.64.0.0/10), you have CGNAT

### 6.3 Options for WireGuard with CGNAT

#### Option A: Use IPv6 (Recommended)
- Starlink provides a **public IPv6 address** that can accept inbound connections
- If your WireGuard peer also has IPv6 connectivity, simply update the endpoint
- No VPS or additional services required
- See Section 6.4 for detailed setup instructions

#### Option B: Use a VPS as a Jump Server (Fallback)
If IPv6 is not available on both ends, set up a VPS with a public IP and create a reverse tunnel:
1. Get a cheap VPS (e.g., from Vultr, DigitalOcean, Linode)
2. Set up WireGuard on the VPS
3. Create a tunnel from your OPNsense to the VPS
4. Route traffic through the VPS to your home network

### 6.4 WireGuard over IPv6 Setup (Option A)

Since Starlink provides a public IPv6 address (`2605:59c8:6100:fd85:2e0:67ff:fe31:ab8a/64` in this setup), you can use IPv6 for WireGuard connections, bypassing CGNAT entirely.

#### Prerequisites
- Both OPNsense routers must have IPv6 connectivity
- Remote peer must be able to reach IPv6 addresses

#### 6.4.1 Configure Home OPNsense (Behind Starlink)

1. **Verify WireGuard is listening on IPv6**
   - Go to **VPN > WireGuard > Instances**
   - WireGuard binds to all interfaces by default (IPv4 + IPv6)
   - Confirm Listen Port is `51822`

2. **Verify WAN firewall rule allows IPv6**
   - Go to **Firewall > Rules > WAN**
   - Check the "Allow wireguard from Stately" rule
   - Ensure **TCP/IP Version** is set to `IPv4+IPv6` (not IPv4 only)
   - If IPv4 only, edit the rule:
     - Change TCP/IP Version to `IPv4+IPv6`
     - Save and Apply

#### 6.4.2 Configure Remote Peer (Stately OPNsense)

1. **Update the peer endpoint to use IPv6**
   - Go to **VPN > WireGuard > Peers**
   - Edit the peer configuration for the home network
   - Update **Endpoint Address**:
     ```
     Old: <previous Comcast IPv4 or hostname>
     New: 2605:59c8:6100:fd85:2e0:67ff:fe31:ab8a
     ```
   - Keep **Endpoint Port**: `51822`
   - Save and Apply

2. **Restart WireGuard service** (optional but recommended)
   - Go to **VPN > WireGuard > General**
   - Toggle the service off and on, or click restart

#### 6.4.3 Verify Connection

1. **Check handshake status**
   - Go to **VPN > WireGuard > Status** on either OPNsense
   - Verify "Latest Handshake" shows a recent timestamp

2. **Test connectivity**
   - From Stately, ping the home WireGuard tunnel IP: `10.10.19.2`
   - From home, ping the Stately WireGuard tunnel IP

#### 6.4.3.1 Verified Connection Status

**Office OPNsense (Home behind Starlink):**
| Instance | Type | Name | Port | Peer Endpoint | Handshake | TX | RX |
|----------|------|------|------|---------------|-----------|----|----|
| wg1 | interface | backupoffice-dev | 51822 | - | - | - | - |
| wg1 | peer | backup-office-peer | - | [2600:1700:2ed8:2e0:2e0:67ff:fe2f:6628]:20903 | 46s | 5.03 GB | 163.31 GB |

**Stately OPNsense (Remote peer):**
| Instance | Type | Name | Port | Peer Endpoint | Handshake | TX | RX |
|----------|------|------|------|---------------|-----------|----|----|
| wg2 | interface | backup-stately | 20903 | - | - | - | - |
| wg2 | peer | Office-Peer-to-Peer | - | [2605:59c8:6100:fd85:2e0:67ff:fe31:ab8a]:51822 | 90s | 163.38 GB | 4.75 GB |

> **Confirmed**: Both peers show recent handshakes and active data transfer over IPv6.

#### 6.4.4 Handle Dynamic IPv6 Address (Optional)

Starlink's DHCPv6 may change your IPv6 address over time. To handle this:

**Option 1: Dynamic DNS with IPv6 support**
- Use a DDNS provider that supports IPv6 (Cloudflare, Hurricane Electric, dynv6.com)
- Configure OPNsense DDNS client at **Services > Dynamic DNS**
- Update the peer endpoint to use the DDNS hostname instead of the IP

**Option 2: Monitor and update manually**
- Check your IPv6 address periodically at **Interfaces > Overview**
- Update the remote peer endpoint if it changes

> **Note**: IPv6 addresses from Starlink tend to be relatively stable, but monitoring is recommended.

---

## Step 7: Update Firewall Rules

### 7.1 Review WAN Rules
1. Go to **Firewall > Rules > WAN**
2. Ensure rules for WireGuard are present (auto-created with NAT or manual)

### 7.2 Verify LAN Rules
1. Go to **Firewall > Rules > LAN**
2. Ensure your outbound traffic rules are correct

---

## Step 8: DNS Configuration

### 8.1 Update DNS Servers (Optional)
Starlink's DNS servers work fine, but you can use alternatives:

1. Go to **System > Settings > General**
2. Set DNS servers:
   - `1.1.1.1` (Cloudflare)
   - `8.8.8.8` (Google)
   - Or use your preferred DNS

---

## Step 9: Test Everything

### 9.1 Checklist
- [x] OPNsense WAN has IP from Starlink
- [x] LAN devices can access internet
- [x] DNS resolution works
- [x] WireGuard VPN connects via IPv6
- [x] WireGuard handshake successful (check VPN > WireGuard > Status)
- [ ] All necessary services are accessible

### 9.2 Speed Test
1. Run a speed test from a LAN device
2. Compare with Starlink's expected speeds for your area

---

## Troubleshooting

### No IP on WAN Interface
1. Verify bypass mode is enabled in Starlink app
2. Reboot Starlink router
3. Reboot OPNsense
4. Check Ethernet cable connection

### Intermittent Connectivity
- Starlink may have brief outages during satellite handoffs
- Check dish placement for obstructions
- Use Starlink app to check for obstructions

### Slow Speeds
- Check dish alignment
- Verify no obstructions (trees, buildings)
- Check OPNsense interface statistics for errors

### WireGuard Not Working
- Confirm you're behind CGNAT (likely if IPv4 is 100.x.x.x)
- **Recommended**: Use IPv6 for WireGuard endpoint (see Section 6.4)
- Verify WAN firewall rule allows IPv4+IPv6 on port 51822
- Check WireGuard status page for handshake errors
- Use VPS jump server as fallback if IPv6 unavailable (see Option B in Section 6.3)
- Check firewall logs for blocked traffic

---

## Additional Notes

### Starlink Characteristics
- **Latency**: 20-60ms typical (better than old satellite internet)
- **Speed**: Varies by location and congestion (50-250 Mbps typical)
- **Reliability**: Brief outages during satellite transitions are normal
- **IP Changes**: DHCP lease may change; consider DDNS if you get a public IP

### Accessing Starlink Statistics
Even in bypass mode, you can access Starlink dish statistics:
- Navigate to `http://192.168.100.1` from your network
- This IP is always available for dish diagnostics

---

## Summary

1. ✅ Backup OPNsense configuration
2. ✅ Enable Bypass Mode in Starlink app
3. ✅ Connect Starlink router to OPNsense WAN
4. ✅ Configure OPNsense WAN for DHCP
5. ✅ Address CGNAT limitations for inbound connections (use IPv6 for WireGuard)
6. ✅ Test all services

---

*Last updated: February 2026*
