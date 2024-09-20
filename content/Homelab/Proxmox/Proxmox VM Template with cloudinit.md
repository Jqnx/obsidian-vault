---
title: Proxmox VM Template with cloudinit
draft: false
tags:
  - proxmox
  - ubuntu
  - cloudinit
  - homelab
  - tailscale
---

![[cloud-init.png]]

cloud-init is a popular mechanism for initializing cloud server images. It is supported by [most major Linux distributions](https://cloudinit.readthedocs.io/en/latest/reference/distros.html) and is available with [most public cloud providers](https://cloudinit.readthedocs.io/en/latest/reference/datasources.html#datasources-supported). I use it on a self-hosted Proxmox node with an Ubuntu Cloud image. I am also installing the following packages automatically with cloud-init:
- [qemu-guest-agent](https://ubuntu.pkgs.org/24.04/ubuntu-universe-amd64/qemu-guest-agent_8.2.2+ds-0ubuntu1_amd64.deb.html)
- [git](https://ubuntu.pkgs.org/24.04/ubuntu-main-amd64/git_2.43.0-1ubuntu7_amd64.deb.html)
- [nfs-common](https://ubuntu.pkgs.org/24.04/ubuntu-main-amd64/nfs-common_2.6.4-3ubuntu5_amd64.deb.html)
- [tailscale](https://tailscale.com/) (also joins my tailnet automatically)

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

Import the downloaded Ubuntu img to `local-lvm` storage (this is for Proxmox 8.2 and below).
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


runcmd:
	# Start qemu-guest-agent
	- systemctl start qemu-guest-agent
	# One-command install, from https://tailscale.com/download/
	- ['sh', '-c', 'curl -fsSL https://tailscale.com/install.sh | sh']
	# Set sysctl settings for IP forwarding (useful when configuring an exit node)
	- ['sh', '-c', "echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf && echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf && sudo sysctl -p /etc/sysctl.d/99-tailscale.conf" ]
	# Generate an auth key from your Admin console
	# https://login.tailscale.com/admin/settings/keys
	# and replace the placeholder below
	- ['tailscale', 'up', '--authkey=tskey-REPLACE_ME']
	- # (Optional) Include this line to make this node available over Tailscale SSH
	- ['tailscale', 'set', '--ssh']

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
[Install Tailscale with cloud-init · Tailscale Docs](https://tailscale.com/kb/1293/cloud-init)