# Nextcloud LDAP Outage - Incident Summary & Fix

## Overview

Nextcloud became unavailable with **Internal Server Error** messages.  
Logs showed repeated errors:

```
OC\ServerNotAvailableException: Lost connection to LDAP server
```

This affected Web UI and WebDAV clients.

---

## Root Cause

The issue was **not Nextcloud itself**, but a combination of **Docker networking + DNS resolution**:

1. **LDAP backend**
   - LDAP server: **Univention**
   - Internal IP: `10.230.8.5`
   - LDAP port: `7389`
   - LDAP hostname in Nextcloud: `master.atlanhomeinc.com`

2. **DNS resolution problem**
   - `master.atlanhomeinc.com` is publicly hosted in **Hetzner DNS**
   - Public DNS resolved it to: `98.62.164.77`
   - Inside the Docker container, DNS resolution followed public DNS
   - LDAP on the public IP/port was unreachable → connection refused

3. **Docker networking**
   - Nextcloud runs inside Docker (`172.17.0.0/16`)
   - Docker containers initially had no explicit routing/NAT to the internal LDAP subnet
   - Even after routing was fixed, DNS still pointed to the wrong (public) IP

---

## Immediate Mitigation

To restore service quickly:

```bash
occ app:disable user_ldap
```

This confirmed LDAP connectivity as the blocking issue and restored access for local users.

---

## Networking Fix (Docker → LDAP)

Enabled proper forwarding and NAT so containers can reach the LDAP server:

```bash
# Allow Docker → LDAP
iptables -I FORWARD -s 172.17.0.0/16 -d 10.230.8.5 -p tcp --dport 7389 -j ACCEPT
iptables -I FORWARD -d 172.17.0.0/16 -s 10.230.8.5 -p tcp --sport 7389 -j ACCEPT

# NAT for return traffic
iptables -t nat -A POSTROUTING -s 172.17.0.0/16 -d 10.230.8.5 -p tcp --dport 7389 -j MASQUERADE
```

Verified from inside the container:

```bash
nc -vz 10.230.8.5 7389
```

---

## DNS Fix (Targeted, Non-Invasive)

To avoid changing office-wide DNS during business hours, DNS was fixed **locally**:

- Ensured `master.atlanhomeinc.com` resolved to `10.230.8.5`
- Avoided reliance on public DNS for internal LDAP resolution
- This stabilized LDAP connectivity without touching OPNsense VLAN DNS

Once DNS resolved correctly, LDAP was re-enabled:

```bash
occ app:enable user_ldap
occ ldap:test-config s01
```

---

## Why Backups Did Not Help

Restoring a Proxmox backup did **not** fix the issue because:
- Backups restore files and containers
- They do **not** restore firewall, routing, or DNS behavior
- The DNS and routing issues reappeared immediately after restore

---

## Current Stability Assessment

**Status:** Stable, but configuration-dependent

### Stable as long as:
- LDAP IP remains `10.230.8.5`
- Current DNS resolution remains unchanged
- Docker routing rules persist

### Potential future risks:
- DNS changes in OPNsense
- LDAP IP changes
- VM rebuilds without documenting DNS assumptions

---

## Recommended Long-Term Fix (Deferred)

Implement **split-horizon DNS in OPNsense**:

- Use Unbound DNS
- Add a **Domain Override** for `atlanhomeinc.com → 10.230.8.5`
- Let OPNsense forward all other queries to public DNS (1.1.1.1 / 8.8.8.8)

This removes all IP-based assumptions and makes the setup future-proof.

---

## Key Takeaways

- Nextcloud was healthy; LDAP dependency was the failure point
- Public DNS + private LDAP is a classic split-DNS problem
- Quick mitigation restored service
- Proper fix requires DNS authority separation

---

*Incident Date: 2026-01-13*
