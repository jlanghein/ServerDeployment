# WireGuard Installation Guide for Ubuntu Server

This guide walks you through installing and configuring **WireGuard** on an Ubuntu Server (20.04, 22.04, 24.04).

---

## 1. Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Install WireGuard

Ubuntu includes WireGuard in the official repositories.

```bash
sudo apt install wireguard -y
```

Verify installation:

```bash
wg --version
```

---

## 3. Generate Server Keys

Create a secure directory for WireGuard keys:

```bash
sudo mkdir -p /etc/wireguard
sudo chmod 700 /etc/wireguard
cd /etc/wireguard
```

Generate private and public keys:

```bash
wg genkey | sudo tee server_private.key | wg pubkey | sudo tee server_public.key
```

Secure the private key:

```bash
sudo chmod 600 server_private.key
```

---

## 4. Configure the WireGuard Interface

Create the configuration file:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Example configuration:

```ini
[Interface]
Address = 10.220.95.1/32
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
SaveConfig = true

# Enable NAT
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

Replace `<SERVER_PRIVATE_KEY>` with:

```bash
sudo cat /etc/wireguard/server_private.key
```

> Note: Adjust `eth0` if your main network interface has a different name.

---

## 5. Enable IP Forwarding

Edit sysctl configuration:

```bash
sudo nano /etc/sysctl.conf
```

Ensure the following line exists and is uncommented:

```ini
net.ipv4.ip_forward=1
```

Apply changes:

```bash
sudo sysctl -p
```

---

## 6. Configure Firewall (UFW)

Allow WireGuard port:

```bash
sudo ufw allow 51820/udp
```

Allow forwarding:

```bash
sudo nano /etc/default/ufw
```

Set:

```ini
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Reload UFW:

```bash
sudo ufw reload
```

---

## 7. Start and Enable WireGuard

Start the interface:

```bash
sudo wg-quick up wg0
```

Enable on boot:

```bash
sudo systemctl enable wg-quick@wg0
```

Check status:

```bash
sudo wg
```

---

## 8. Add a Client (Example)

Generate client keys:

```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

Add client to server config:

```ini
[Peer]
PublicKey = <CLIENT_PUBLIC_KEY>
AllowedIPs = 10.220.95.2/32
```

Apply changes:

```bash
sudo wg syncconf wg0 <(wg-quick strip wg0)
```

---

## 9. Client Configuration Example

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.220.95.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

---

## 10. Troubleshooting

Useful commands:

```bash
sudo wg
sudo systemctl status wg-quick@wg0
sudo journalctl -u wg-quick@wg0
```

---

## Notes

- UDP port **51820** is the default but can be changed
- Use `/24` subnets if you plan multiple clients
- For site-to-site VPNs, expand `AllowedIPs` accordingly

---

**Done!** Your WireGuard server is now operational.
