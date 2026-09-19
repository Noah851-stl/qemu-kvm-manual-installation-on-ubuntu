# QEMU/KVM Virtualization Step-by-Step Manual Guide

A beginner-friendly guide to manually installing, configuring, and managing QEMU/KVM on Ubuntu.

---

## 📑 Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Check Hardware Virtualization](#2-check-hardware-virtualization)
3. [Install Required Packages](#3-install-required-packages)
4. [User Permissions & Services](#4-user-permissions--services)
5. [Verify Installation](#5-verify-installation)
6. [Create Your First VM](#6-create-your-first-vm)
7. [Basic Management Commands (CLI)](#7-basic-management-commands-cli)
8. [Networking Basics](#8-networking-basics)
9. [Storage Pool Setup](#9-storage-pool-setup)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Prerequisites

Before you begin, make sure you have:

| Requirement | Details |
|-------------|---------|
| **OS** | Ubuntu 20.04 LTS or newer (22.04 / 24.04 recommended) |
| **Access** | `sudo` or root privileges |
| **CPU** | Intel VT-x or AMD-V virtualization support |
| **RAM** | Minimum 4 GB (8 GB+ recommended) |
| **Disk** | At least 20 GB free space |

> 💡 **Tip:** If you plan to run multiple VMs, allocate more RAM and disk space.

---

## 2. Check Hardware Virtualization

Run the following command to check if your CPU supports virtualization:

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

---

## 3. Install Required Packages
Update your package list and install QEMU, KVM, and management tools:

```bash
sudo apt update
sudo apt install -y \
  qemu-system-x86 \
  qemu-utils \
  libvirt-daemon-system \
  libvirt-clients \
  bridge-utils \
  ```

---

## 4. User Permissions & Services
Add your user to the required groups
To run VMs without sudo, add your user to the libvirt and kvm groups:

```bash
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```
Enable and start the libvirt service
```bash
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
```

📝 Note: On Ubuntu 22.04+, libvirt uses socket activation. If libvirtd service is inactive, check:

```bash
sudo systemctl status libvirtd.socket
```

---

## 5. Verify Installation
Confirm that everything is working:
```bash
virsh version
virt-host-validate qemu
```
If no critical errors appear, your system is ready to run VMs.

---

### 6. Create Your First VM
Option A: Using virt-install (CLI)
```bash
sudo virt-install \
  --name ubuntu-vm \
  --ram 2048 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/ubuntu-vm.qcow2,size=20 \
  --os-variant ubuntu22.04 \
  --network network=default \
  --graphics spice \
  --cdrom /path/to/ubuntu.iso
```

Option B: Using virt-manager (GUI)
```bash
virt-manager
```
Then click Create a new virtual machine and follow the wizard.

---

## 7. Basic Management Commands (CLI)
List all VMs
```bash
virsh list --all
```
Start a VM
```bash
virsh start <vm-name>
```
Graceful shutdown (sends ACPI signal)
```bash
virsh shutdown <vm-name>
```
Force stop (hard power off)
```bash
virsh destroy <vm-name>
```
Reboot a VM
```bash
virsh reboot <vm-name>
```
View VM information
```bash
virsh dominfo <vm-name>
```
Delete a VM
```bash
virsh undefine <vm-name> --remove-all-storage
```
Delete a UEFI VM (with NVRAM)
```bash
virsh undefine <vm-name> --remove-all-storage --nvram
```
⚠️ Warning: --remove-all-storage permanently deletes all disk images. Back up your data first!

Autostart a VM on boot
```bash
virsh autostart <vm-name>
```

---

## 8. Networking Basics
Libvirt provides a default NAT network (virbr0). Check it:
```bash
virsh net-list --all
virsh net-info default
```
Start the default network if it's inactive
```bash
virsh net-start default
virsh net-autostart default
```

Bridged networking (for LAN access)
Install bridge utilities (already installed above) and edit Netplan:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
Example bridge config:
```bash
yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
  bridges:
    br0:
      interfaces: [enp3s0]
      dhcp4: yes
```
Apply:

```bash
sudo netplan apply
```
Then use --network bridge=br0 when creating VMs.

---

## 9. Storage Pool Setup
Default storage pool location: /var/lib/libvirt/images/

List storage pools
```bash
virsh pool-list --all
```
Create a custom storage pool
```bash
sudo mkdir -p /mnt/vm-storage
sudo virsh pool-define-as vmstore dir --target /mnt/vm-storage
sudo virsh pool-build vmstore
sudo virsh pool-start vmstore
sudo virsh pool-autostart vmstore
```
Verify
```bash
virsh pool-list --all
virsh vol-list vmstore
```
---

## 10. Troubleshooting
|  Problem  |  Cause |  Solution  |
|-----------|--------|------------|
|KVM acceleration not available|VT-x/AMD-V disabled	|Enable in BIOS/UEFI
Permission denied on virsh|	User not in libvirt group	|Re-login after usermod
libvirtd service not found|	Ubuntu 22.04+ uses socket|Check libvirtd.socket
Cannot connect to hypervisor|Daemon not running|sudo systemctl restart libvirtd
VM has no internet|Default network inactive|virsh net-start default
Disk image locked|	VM still running|virsh destroy <vm> first

Check kernel modules
```bash
lsmod | grep kvm
```
You should see kvm_intel or kvm_amd.

Load modules manually if missing
```bash
sudo modprobe kvm
sudo modprobe kvm_intel   # or kvm_amd
```
---

## 11. Uninstall
To completely remove QEMU/KVM and related packages:

```bash
sudo systemctl stop libvirtd
sudo apt purge -y \
  qemu-system-x86 qemu-utils \
  libvirt-daemon-system libvirt-clients \
  bridge-utils virt-manager virtinst
sudo apt autoremove -y
```
Remove leftover data (⚠️ destructive):
```bash
sudo rm -rf /var/lib/libvirt/
sudo rm -rf /etc/libvirt/
```
---

## 🎓 What I Learned

This project was more than just following a tutorial — it was a hands-on journey into Linux virtualization. Here's a summary of the key lessons I picked up while building this guide:

### 🔧 Technical Skills

- **Hardware Virtualization Detection** — Learned how to check CPU virtualization support using `egrep` and `kvm-ok`, and understood the difference between Intel VT-x (`vmx`) and AMD-V (`svm`).
- **QEMU vs KVM vs libvirt** — Clarified the relationship: QEMU is the emulator, KVM is the kernel acceleration module, and libvirt is the management layer that ties them together.
- **Package Management** — Understood which packages are essential (`qemu-system-x86`, `libvirt-daemon-system`) versus optional (`virt-manager`, `virtinst`).
- **User & Group Permissions** — Learned why `libvirt` and `kvm` group membership matters, and why a re-login is required after `usermod`.
- **Systemd Socket Activation** — Discovered that Ubuntu 22.04+ changed how `libvirtd` starts, moving from a traditional service to socket-based activation.
- **VM Lifecycle Management** — Practiced `virsh` commands for starting, shutting down, destroying, and undefining VMs.
- **UEFI/NVRAM Handling** — Learned that UEFI-based VMs require the `--nvram` flag when undefining, which is a common pitfall.
- **Networking & Storage Pools** — Explored how libvirt handles virtual networks (`virbr0`) and storage pools.

### 🧠 Soft Skills & Mindset

- **Documentation Matters** — Writing a clear README forced me to organize my thoughts and verify every step.
- **Reading Error Messages** — Many issues (permission denied, daemon not found) were solved by carefully reading the actual error text.
- **Version Awareness** — What works on Ubuntu 20.04 may behave differently on 22.04+. Always check your version.
- **Safety First** — Commands like `virsh undefine --remove-all-storage` are destructive. Learning to warn users (and myself) is essential.

> *"The best way to learn virtualization is to break it, fix it, and document it."*




