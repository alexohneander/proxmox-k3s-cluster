# Proxmox Kubernetes Cluster Tutorial/Installation

## Setup Cloud Init Template

This step is describe how to create cloud init template for provide provisioning virtual machine template on proxmox. If you already have VM on proxmox server you can skip.

First, we need to remote on proxmox server using SSH or direct access into proxmox server. And we need to download operating system, I will use Ubuntu server 22.04 as cloud init template. Download the Ubuntu cloud init.

```bash
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
```

Then, we need customize the iso images to enable qemu agent. Install libguestfs-tools if you don’t have it.

```bash
apt install libguestfs-tools
virt-customize -a jammy-server-cloudimg-amd64.img --install qemu-guest-agent,net-tools --truncate /etc/machine-id
```

Create VM follow this command.

```bash
qm create 8000 --name ubuntu-cloud-init --core 2 --memory 2048 --net0 virtio,bridge=vmbr0
```

It will create VM template with VM ID is `8000` and set default processor core is `2`, memory `2GB` and use VM bridge `vmbr0` as network interface.

Then, import disk into cloud init VM. This step is like we have storage but the SATA cable is not connected.

```bash
qm disk import 8000 jammy-server-cloudimg-amd64.img local-lvm
```

So, we need to attach disk into VM and setup boot order

```bash
qm set 8000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-8000-disk-0
qm set 8000 --boot c --bootdisk scsi0
```

Then activate qemu agent also set the serial socket vga for console and hotplug.

```bash
qm set 8000 --agent 1
qm set 8000 --serial0 socket
qm set 8000 --vga serial0
qm set 8000 --hotplug network,usb,disk
```

Convert cloud init VM into template

```bash
qm template 8000
```

Create API Token that will be used for `Terraform`. Go to `Data Center` -> `Permissions` -> `API Tokens` then add new API token. **Note** : Uncheck the `Privilege Separation` and don’t forget to take a note the Token ID and Secret.


