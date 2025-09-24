# Ubuntu Server VM with Cloud-Init in Proxmox template

## Step 1: Download the Ubuntu Cloud Image
Proxmox works best with the official cloud images (prebuilt for cloud-init).
1. SSH into your Proxmox node.
2. Go to your ISO/container template storage path (usually /var/lib/vz/template/iso or /var/lib/vz/template/qemu depending on setup).
3. Download the Ubuntu cloud image, for example Ubuntu 22.04 LTS:
```
cd /var/lib/vz/template/iso
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
```

## Step 2: Create the VM
Use the downloaded image to create a base VM.
```
qm create 9000 --name "ubuntu-2204-cloudinit" --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0
```
* 9000 → VM ID (pick a high number for templates).
* Adjust memory, cores, and bridge as needed.

## Step 3: Import the Disk
Attach the downloaded cloud image to the VM as a boot disk.
```
qm importdisk 9000 jammy-server-cloudimg-amd64.img local-lvm
```
Then attach it:
```
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0
```

## Step 4: Add Cloud-Init Drive
Add the Cloud-Init device for configuration:
```
qm set 9000 --ide2 local-lvm:cloudinit
```

## Step 5: Configure Boot & Serial Console
Set boot order and enable serial console (needed for cloud images):
```
qm set 9000 --boot c --bootdisk scsi0
qm set 9000 --serial0 socket --vga serial0
```

## Step 6: Cloud-Init Settings
(Optional) Set defaults for the template:
```
qm set 9000 --ciuser ubuntu --sshkeys ~/.ssh/id_rsa.pub --ipconfig0 ip=dhcp
```
* Replace ~/.ssh/id_rsa.pub with your actual public key file.
* You can later override these per-VM when deploying from template.

## Step 7: Convert VM to Template
Shut down the VM (if running) and convert to a template:
```
qm template 9000
```

## Step 8: Deploy New VMs from Template
To create a new VM from the template:
```
qm clone 9000 100 --name ubuntu-test --full true
```
Then set networking and Cloud-Init per VM:
```
qm set 100 --ciuser bo --sshkeys ~/.ssh/id_rsa.pub --ipconfig0 ip=192.168.1.50/24,gw=192.168.1.1
```
Finally, start the VM:
```
qm start 100
```

✅ At this point, you have a clean, reusable Ubuntu Cloud-Init template.
Any clones can be configured with unique IPs, usernames, and SSH keys via Cloud-Init.
