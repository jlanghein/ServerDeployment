# Installing Tactical RMM Agent on Ubuntu Desktop

Follow these steps to install the Tactical RMM agent on an Ubuntu Desktop.

## Steps

1. Move the `.sh` installation file to the **workstation folder** on your Ubuntu system.

2. Open a terminal and navigate to the workstation folder:

```bash
cd /path/to/workstation/folder
unset DISPLAY
chmod +x rmm.sh
sudo ./rmm.sh
```

## Notes

- The `unset DISPLAY` command is required to prevent GUI-related issues during installation
- Make sure to run the script with `sudo` for proper permissions
- Replace `/path/to/workstation/folder` with the actual path where you saved the installation script
