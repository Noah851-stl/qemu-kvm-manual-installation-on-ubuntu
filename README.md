# QEMU/KVM Virtualization Step-by-Step Manual Guide

A beginner-friendly guide to manually installing, configuring, and managing QEMU/KVM on Ubuntu.

![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04%2B-orange)
![QEMU](https://img.shields.io/badge/QEMU-KVM-blue)
![License](https://img.shields.io/badge/License-MIT-green)

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
11. [Uninstall](#11-uninstall)
12. [References](#12-references)
13. [License](#13-license)

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
