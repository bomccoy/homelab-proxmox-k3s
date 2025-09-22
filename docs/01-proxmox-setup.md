# Proxmox Initial Setup

After installing Proxmox:

## 1. Switch to No-Subscription Repo
```bash
rm /etc/apt/sources.list.d/pve-enterprise.list
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" \
  > /etc/apt/sources.list.d/pve-no-subscription.list
```

## 2. Update & Reboot
```bash
apt update && apt full-upgrade -y
reboot
```

## 3. Verify Web UI
- Log in at `https://<your-ip>:8006`
- Confirm node shows “up to date”
