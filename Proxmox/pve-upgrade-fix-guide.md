# Fixing Proxmox Virtual Environment (PVE) Upgrade Issues

If you encounter the following error while trying to update **Proxmox Virtual Environment (PVE)**:

```
Err:1 https://enterprise.proxmox.com/debian/pve bookworm InRelease
  401  Unauthorized [IP: 66.70.154.82 443]
E: Failed to fetch https://enterprise.proxmox.com/debian/pve/dists/bookworm/InRelease  401  Unauthorized
E: The repository 'https://enterprise.proxmox.com/debian/pve bookworm InRelease' is not signed.
```

This happens because **PVE is using the Enterprise repository**, which requires a **paid subscription**.  
For users without a subscription, the **No-Subscription repository** should be used instead.

---

## Step 1: Disable the Enterprise Repository

The **Enterprise repository** is located in:
```sh
/etc/apt/sources.list.d/pve-enterprise.list
```

### Edit the file and comment out the Enterprise repository:
```sh
nano /etc/apt/sources.list.d/pve-enterprise.list
```

Change:
```
deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```
To:
```
# deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```

**Save and exit**: `CTRL + X`, then `Y`, then `ENTER`.

---

## Step 2: Add the No-Subscription Repository

Create or edit the **No-Subscription repository file**:
```sh
nano /etc/apt/sources.list.d/pve-no-subscription.list
```

Add this line:
```
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```

**Save and exit**: `CTRL + X`, then `Y`, then `ENTER`.

---

## Step 3: Disable Enterprise Ceph Repository (If Using Ceph)

If you're using **Ceph**, Proxmox may also try to fetch updates from the **Enterprise Ceph repository**, causing a `401 Unauthorized` error:

```
Err:4 https://enterprise.proxmox.com/debian/ceph-quincy bookworm InRelease
  401  Unauthorized [IP: 170.130.165.90 443]
```

To fix this, disable the **Enterprise Ceph repository** and enable the **No-Subscription Ceph repository**.

### Edit the Ceph repository file:
```sh
nano /etc/apt/sources.list.d/ceph.list
```

Change:
```
deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
```
To:
```
# deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
```

**Save and exit**: `CTRL + X`, then `Y`, then `ENTER`.

### Add the No-Subscription Ceph Repository
```sh
nano /etc/apt/sources.list.d/pve-ceph.list
```

Add this line:
```
deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription
```

**Save and exit**: `CTRL + X`, then `Y`, then `ENTER`.

---

## Step 4: Update and Upgrade PVE

Now, update and upgrade your system:
```sh
apt update && apt full-upgrade -y
```

If everything is successful, your PVE will now be up to date!

---

## Step 5: Remove Unused Packages (Optional)

After upgrading, clean up unused packages:
```sh
apt autoremove -y && apt clean
```

---

## Summary

| Step | Action |
|------|--------|
| 1 | Disable Enterprise Repo (`pve-enterprise.list`) |
| 2 | Add No-Subscription Repo (`pve-no-subscription.list`) |
| 3 | Disable Enterprise Ceph Repo (`ceph.list`) if using Ceph |
| 4 | Add No-Subscription Ceph Repo (`pve-ceph.list`) if using Ceph |
| 5 | Run `apt update && apt full-upgrade -y` |
| 6 | Clean up with `apt autoremove -y && apt clean` |

Your Proxmox Virtual Environment (PVE) is now updated!
