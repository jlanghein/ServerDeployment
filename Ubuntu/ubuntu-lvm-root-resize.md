# Expanding an LVM Root Filesystem on Ubuntu (After Disk Resize)

This guide documents how to extend an Ubuntu VM root filesystem when the virtual disk
has already been expanded at the hypervisor level (e.g. Proxmox, VMware, Hyper-V).

Tested on Ubuntu with LVM and ext4.

---

## Prerequisites

- VM disk size already increased in the hypervisor
- Root filesystem is on LVM
- You have sudo access
- No reboot required

---

## 1. Verify Disk Size Inside the VM

```bash
lsblk
```

Expected result (example):

```
sda        50G
├─sda2      2G  /boot
└─sda3     48G
  └─ubuntu-vg-ubuntu-lv  24G  /
```

If `sda` does **not** show the increased size, stop here and fix the disk resize in the hypervisor first.

---

## 2. Resize the LVM Physical Volume

Tell LVM that the underlying partition is larger now:

```bash
sudo pvresize /dev/sda3
```

Verify:

```bash
sudo pvs
```

---

## 3. Check Free Space in the Volume Group

```bash
sudo vgs
```

Look at the `VFree` column — this is the space that can be assigned to logical volumes.

---

## 4. Extend the Root Logical Volume

Extend the root LV to use **all remaining free space**:

```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

---

## 5. Resize the Filesystem

For ext4 (default on Ubuntu):

```bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```

For XFS (if applicable):

```bash
sudo xfs_growfs /
```

---

## 6. Verify the Result

```bash
df -h /
```

Expected result (example):

```
/dev/mapper/ubuntu--vg-ubuntu--lv   48G   15G   31G  33% /
```

---

## Summary (Copy/Paste)

```bash
lsblk
sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
df -h /
```

---

## Notes & Best Practices

- Consider leaving free VG space for snapshots on production systems
- Heavy-write paths like `/var/lib/docker` or `/var/lib/postgresql` can be split into separate LVs
- This process is safe and does not affect existing data when done correctly

---

*Last updated: February 2026*
