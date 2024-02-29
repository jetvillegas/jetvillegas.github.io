---
author: jet
categories:
- Homelab
- Linux
- Networking
- Server
- Tech
date: "2021-07-04T06:28:57Z"
guid: https://jetvillegas.com/?p=19
id: 19
title: Cloning Virtual Machines in KVM
url: /2021/07/04/cloning-virtual-machines-in-kvm/
---

I run many virtual machines on KVM and VirtualBox. I typically clone an existing VM that is already set up for the project or service I’m working on. These are the steps I take on KVM on Linux (Ubuntu) hosts to clone and deploy a guest VM.

### On the host

Suspend the VM you’re copying

`sudo virsh shutdown source_VM`

Clone the source VM with a new name

`sudo` `virt-clone --original source_VM --name destination_VM --file /var/lib/libvirt/images/destination_VM.qcow2`

Start the source VM back up

`sudo` `virsh start source_VM`

Prepare the new VM for service

`sudo virt-sysprep -d destination_VM --hostname destination_VM --enable customize,user-account,ssh-hostkeys,net-hostname,net-hwaddr,machine-id --keep-user-accounts your_username --keep-user-accounts root`

Start the new VM

`sudo virsh start destination_VM`

### On the guest

Fix up SSH

`sudo dpkg-reconfigure openssh-server`

Restart the guest

`sudo reboot`