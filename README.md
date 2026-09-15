# QEMU/KVM Virtualization Step-by-Step Manual Guide

A beginner-friendly guide to manually installing, configuring, and managing QEMU/KVM on Ubuntu.

## 1.To Check Hardware Virtualization
      egrep -c '(vmx|svm)' /proc/cpuinfo
        
if the output is > 0 the virtualization is good to support.

## 2.Manually install packages Install the required QEMU, KVM, and Management Tools.
      sudo apt update
      sudo apt install -y qemu-system-x86 qemu-utils libvirt-daemon-system libvirt-clients bridge-utils virt-manager
      
## 3. User Permission and Service Check To run VMs without sudo
     sudo usermod -aG libvirt $USER
     sudo usermod -aG kvm $USER

## 4.Check that the Libvirt Service is running:
     sudo systemctl status libvirtd
Note: Log out of Terminal and log back in (or restart the machine) for the permission to work.

## 5. Basic Management Commands (CLI) 
To list VMs: 
              
    virsh list --all 
To start a VM:
              
    virsh start <vm-name> 
To shut down a VM:
         
    virsh shutdown <vm-name> 
To force stop a VM:

    virsh destroy <vm-name> 
To delete a VM:

    virsh undefine <vm-name> --remove-all-storage
