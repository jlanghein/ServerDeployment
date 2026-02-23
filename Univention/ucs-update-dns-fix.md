# Fixing UCS Package Update Issues Due to DNS Resolution Failure

## Issue

Univention Corporate Server (UCS) failed to update packages due to DNS resolution issues, causing errors like:

```
Err:1 https://updates.software-univention.de ... Could not resolve 'updates.software-univention.de'
E: Failed to fetch ...
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

## Solution

### 1. Switch to Root User

Ensure you have root access:
```bash
sudo -i
```

### 2. Manually Set DNS

Since UCS auto-generates `/etc/resolv.conf`, set DNS temporarily:
```bash
nano /etc/resolv.conf
```
Replace its contents with:
```
nameserver 8.8.8.8
nameserver 1.1.1.1
```
Save and exit (`Ctrl + X`, then `Y`, and `Enter`).

Test DNS resolution:
```bash
ping -c 4 updates.software-univention.de
```

### 3. Reinstall Univention Configuration Registry

If the Univention Configuration Registry (`ucr`) is missing or broken:
```bash
apt-get install --reinstall univention-config-registry
```

### 4. Restart UCS Services

```bash
systemctl restart univention-config-registry
systemctl restart univention-directory-listener
```

### 5. Verify `ucr` and Set Permanent DNS

Check if `ucr` is working:
```bash
ucr get nameserver
```
Set permanent DNS using UCS Configuration Registry:
```bash
ucr set nameserver1="8.8.8.8"
ucr set nameserver2="1.1.1.1"
ucr commit /etc/resolv.conf
service networking restart
```

### 6. Update the System

Once DNS is fixed, update UCS:
```bash
apt-get update && apt-get dist-upgrade
```

### 7. Reboot to Apply Changes

```bash
reboot
```

After reboot, verify everything works:
```bash
ucr get nameserver
apt-get update
```
