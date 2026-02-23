# Fixing Proxmox Backup Server (PBS) Upgrade Issues

If you encounter the following error while trying to update **Proxmox Backup Server (PBS)**:

```
Err:1 https://enterprise.proxmox.com/debian/pbs bookworm InRelease
  401  Unauthorized [IP: 66.70.154.82 443]
E: Failed to fetch https://enterprise.proxmox.com/debian/pbs/dists/bookworm/InRelease  401  Unauthorized
E: The repository 'https://enterprise.proxmox.com/debian/pbs bookworm InRelease' is not signed.
```

This happens because **PBS is using the Enterprise repository**, which requires a **paid subscription**.  
For users without a subscription, the **No-Subscription repository** should be used instead.

---

## Step 1: Disable the Enterprise Repository

The **Enterprise repository** is located in:
```sh
/etc/apt/sources.list.d/pbs-enterprise.list
```

### Edit the file and comment out the Enterprise repository:
```sh
nano /etc/apt/sources.list.d/pbs-enterprise.list
```

Change:
```
deb https://enterprise.proxmox.com/debian/pbs bookworm pbs-enterprise
```
To:
```
# deb https://enterprise.proxmox.com/debian/pbs bookworm pbs-enterprise
```

**Save and exit**: `CTRL + X`, then `Y`, then `ENTER`.

---

## Step 2: Add the No-Subscription Repository

Create or edit the **No-Subscription repository file**:
```sh
nano /etc/apt/sources.list.d/pbs-no-subscription.list
```

Add this line:
```
deb http://download.proxmox.com/debian/pbs bookworm pbs-no-subscription
```

**Save and exit**: `CTRL + X`, then `Y`, then `ENTER`.

---

## Step 3: Update and Upgrade PBS

Now, update and upgrade your system:
```sh
apt update && apt full-upgrade -y
```

If everything is successful, your PBS will now be up to date!

---

## Step 4: Remove Unused Packages (Optional)

After upgrading, clean up unused packages:
```sh
apt autoremove -y && apt clean
```

---

## Summary

| Step | Action |
|------|--------|
| 1 | Disable Enterprise Repo (`pbs-enterprise.list`) |
| 2 | Add No-Subscription Repo (`pbs-no-subscription.list`) |
| 3 | Run `apt update && apt full-upgrade -y` |
| 4 | Clean up with `apt autoremove -y && apt clean` |

Your Proxmox Backup Server (PBS) is now updated!
