---
title: VM Template
draft: false
tags:
  - proxmox
  - linux
  - ubuntu
  - cloudinit
  - homelab
---

## Instructions

Choose your [Ubuntu Cloud Image](https://cloud-images.ubuntu.com/)

Download Ubuntu Cloud Image.
```shell
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
```

Create a new VM. Change specs as needed.
```shell
qm create 9000 --memory 2048 --core 2 --name ubuntu-noble --net virtio,bridge=vmbr0
```

Import the downloaded Ubuntu img to local-lvm storage (this is for Proxmox 8.2 and below).
```shell
qm importdisk 9000 noble-server-cloudimg-amd64.img local-lvm
```

For Proxmox 8.2 and above use this instead.
```shell
qm disk import 9000 noble-server-cloudimg-amd64.img local-lvm
```

Attach the new disk to the VM as a scsi controller.
```shell
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0
```

Add the cloud init drive.
```shell
qm set 9000 --ide2 local-lvm:cloudinit
```

Make the cloud init drive bootable and restrict BIOS to boot from disk only.
```shell
qm set 9000 --boot c --bootdisk scsi0
```

Add serial console.
```shell
qm set 9000 --serial0 socket --vga serial0
```

Enable qemu guest agent.
```shell
qm set 9000 --agent enabled=1,freeze-fs-on-backup=1,fstrim_cloned_disks=1
```

Enable snippets storage in Proxmox GUI.

![[snippets-storage.png]]

Create cloud init vendor config.
```yaml title="var/lib/vz/snippets/9000-civendor.yaml"
#cloud-config

# Updates
package_update: true
package_upgrade: true

# Install packages
packages:
- qemu-guest-agent
- git
- nfs-common

# Start qemu-guest-agent
runcmd:
- systemctl start qemu-guest-agent

# Set timezone
timezone: "Europe/Brussels"
```

Add this vendor config to your VM.
```shell
qm set 9000 --cicustom vendor=local:9000-civendor.yaml
```

> [!warning] DO NOT START THE VM

Change specs and/or cloud init config if needed.

Then create the template.
```shell
qm template 9000
```

> [!question] Troubleshooting
>If you need to reset your machine-id.
> ```shell
> sudo rm -f /etc/machine-id
> sudo rm -f /var/lib/dbus/machine-id
> ```
>  Then shut it down and do not boot it up. A new id should be generated the next time it boots.
>> [!tip]- If a new machine-id doesn't get generated run this
>> ```shell
>> sudo systemd-machine-id-setup
>> ```

## Source
[Perfect Proxmox Template with Cloud Image and Cloud Init | Techno Tim](https://technotim.live/posts/cloud-init-cloud-image/)