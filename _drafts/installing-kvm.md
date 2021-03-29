---
title: "安装kvm"
date: 2021-03-29T14:48:52+08:00
draft: true
---

### 在centos上安装kvm

```bash
yum -y install qemu-kvm qemu-img virt-manager libvirt libvirt-python libvirt-client virt-install virt-viewer bridge-utils
```

具体的解释放在ubuntu一节

### 在ubuntu上安装kvm

```bash
# https://linuxize.com/post/how-to-install-kvm-on-ubuntu-20-04/

# This guide provides instructions on how to install and configure KVM on Ubuntu 20.04

# check if your CPU supports hardware virtualization
egrep -c '(vmx|svm)' /proc/cpuinfo 
# or grep -Eoc '(vmx|svm)' /proc/cpuinfo

sudo apt install cpu-checker
sudo kvm-ok
# INFO: /dev/kvm exists
# KVM acceleration can be used

sudo apt install -y qemu qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
# qemu package (quick emulator) is an application that allows you to perform hardware virtualization.
# qemu-kvm - software that provides hardware emulation for the KVM hypervisor.
# libvirt-daemon-system - configuration files to run the libvirt daemon as a system service.
# libvirt-clients - software for managing virtualization platforms.
# bridge-utils - a set of command-line tools for configuring ethernet bridges.
# virtinst - a set of command-line tools for creating virtual machines.
# virt-manager - an easy-to-use GUI interface and supporting command-line utilities for managing virtual machines through libvirt.

# Once the packages are installed, the libvirt daemon will start automatically. You can verify it by typing:
sudo systemctl is-active libvirtd
# active

# To be able to create and manage virtual machines, you’ll need to add your user to the “libvirt” and “kvm” groups. To do that, enter:
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
# Log out and log back in so that the group membership is refreshed.
virsh list --all

# Network Setup
# A bridge named “virbr0” is created during the installation process. This device uses NAT to connect the guests' machines to the outside world.
brctl show
# bridge name     bridge id               STP enabled     interfaces
# virbr0          8000.5254001c2cd3       yes             virbr0-nic

# The “virbr0” bridge does not have any physical interfaces added. “virbr0-nic” is a virtual device with no traffic routed through it. The sole purpose of this device is to avoid changing the MAC address of the “virbr0” bridge.
```

### kvm上几种网卡的区别

理论上virtio支持万兆
```
rtl8139 10/100Mb/s
e1000 1Gb/s
virtio 10Gb/s
```
