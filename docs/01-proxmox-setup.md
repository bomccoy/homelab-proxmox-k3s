# Switch Proxmox to No-Subscription Repo (via UI)
1. Log into Proxmox Web UI
* Go to https://<your-proxmox-ip>:8006
* Log in as root.

2. Select your node
* In the left tree, click on your Proxmox node (e.g., andromeda).

3. Go to Repositories
* Top menu: Updates → Repositories

4. Remove Enterprise Repo
* Select the line that contains:
```
deb https://enterprise.proxmox.com/debian/pve ...
```
* Click Disable (or Remove if you prefer).

5. Add the No-Subscription Repo
* Click Add → No-Subscription
* A new repo line appears:
```
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```

6. Verify
* Make sure you now only see the no-subscription repo and (optionally) security updates repo.

7. Update package list
* Still under Updates, click Refresh.
* Or run from the Proxmox Shell:
```
apt update
```

8. (Optional) Remove Ceph enterprise repos
* If you’re not running Ceph, you can safely disable those repos as well.

✅ That’s it — you’re now on the community (no-subscription) repo, and you’ll be able to update without subscription errors.
