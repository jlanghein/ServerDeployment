# Eaton UPS Auto Shutdown Setup on Proxmox Backup Server

## Step 1: Install NUT

Run the following command to install Network UPS Tools (NUT):

```bash
apt update
apt install nut nut-client nut-server -y
```

---

## Step 2: Configure the UPS

### 2.1 Edit `/etc/nut/ups.conf`
```bash
nano /etc/nut/ups.conf
```
Add the following (modify `eaton` as needed):

```ini
[eaton]
    driver = usbhid-ups
    port = auto
    desc = "Eaton UPS"
    vendorid = 0463
```

Save and exit.

---

## Step 3: Configure User Authentication

### 3.1 Edit `/etc/nut/upsd.users`
```bash
nano /etc/nut/upsd.users
```
Change the username (`myuser`) and password (`MyNewSecurePassword`):

```ini
[myuser]
    password = MyNewSecurePassword
    actions = SET
    instcmds = ALL
    upsmon master
```

Save and exit.

### 3.2 Edit `/etc/nut/upsmon.conf`
```bash
nano /etc/nut/upsmon.conf
```
Find and update:

```ini
MONITOR eaton@localhost 1 myuser MyNewSecurePassword master
```

Save and exit.

---

## Step 4: Restart NUT Services

```bash
systemctl restart nut-server nut-monitor
systemctl enable nut-server nut-monitor
```

Check UPS status:

```bash
upsc eaton@localhost
```

---

## Step 5: Configure Auto Shutdown

### 5.1 Edit `/etc/nut/upssched.conf`
```bash
nano /etc/nut/upssched.conf
```
Add:

```ini
CMDSCRIPT /etc/nut/upssched-cmd
PIPEFN /var/run/nut/upssched.pipe
LOCKFN /var/run/nut/upssched.lock
AT ONBATT * START-TIMER onbatt 30
AT LOWBATT * EXECUTE shutdown
```

Save and exit.

### 5.2 Create `/etc/nut/upssched-cmd`
```bash
nano /etc/nut/upssched-cmd
```
Add:

```bash
#!/bin/bash

case $1 in
    onbatt)
        logger "UPS is on battery. System will shut down if battery gets low."
        ;;
    shutdown)
        logger "Battery is low. Initiating shutdown!"
        /sbin/shutdown -h now
        ;;
    *)
        logger "Unknown command: $1"
        ;;
esac
```

Make it executable:

```bash
chmod +x /etc/nut/upssched-cmd
```

---

## Step 6: Test the Setup

1. **Unplug the UPS from the wall** (keep Proxmox running on battery).
2. Run:

   ```bash
   journalctl -f -u nut-server
   ```

3. Monitor the logs and verify **shutdown happens when the battery reaches 20%**.

---

Now your **Proxmox Backup Server** will shut down safely when the **UPS battery is low**!
