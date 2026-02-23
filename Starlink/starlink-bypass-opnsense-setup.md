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

### 1.2 Note Your WireGuard Configuration
Record your current WireGuard setup:
- Listening port (commonly `51820`)
- Peer configurations
- Allowed IPs
- Any custom firewall rules for WireGuard

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

#### Option A: Request Public IP from Starlink (If Available)
- Some business plans or portability plans may offer public IPs
- Contact Starlink support to inquire

#### Option B: Use a VPS as a Jump Server (Recommended)
Set up a VPS with a public IP and create a reverse tunnel:
1. Get a cheap VPS (e.g., from Vultr, DigitalOcean, Linode)
2. Set up WireGuard on the VPS
3. Create a tunnel from your OPNsense to the VPS
4. Route traffic through the VPS to your home network

#### Option C: Use Cloudflare Tunnel
- Free option for HTTP/HTTPS services
- Does not work for UDP-based services like WireGuard directly

#### Option D: Use a Dynamic DNS + Tailscale/ZeroTier
- Use mesh VPN services that work with CGNAT
- Tailscale or ZeroTier can establish connections despite CGNAT

### 6.4 If You Have a Public IP (Lucky!)
If you're fortunate to have a public IP, configure port forwards:

1. Go to **Firewall > NAT > Port Forward**
2. Add a new rule:
   - **Interface**: WAN
   - **Protocol**: UDP
   - **Destination port range**: Your WireGuard port (e.g., 51820)
   - **Redirect target IP**: Your WireGuard server IP (OPNsense or internal server)
   - **Redirect target port**: 51820
3. Save and apply

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
- [ ] OPNsense WAN has IP from Starlink
- [ ] LAN devices can access internet
- [ ] DNS resolution works
- [ ] WireGuard VPN connects (with CGNAT workaround if needed)
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
- Confirm you're behind CGNAT (likely)
- Implement one of the CGNAT workaround solutions
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
5. ✅ Address CGNAT limitations for inbound connections
6. ✅ Test all services

---

*Last updated: February 2026*
