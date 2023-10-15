---
tags: linux, archlinux, virtualization
---
# Manage virtual machines with virt-manager

Virt-manager is a desktop user interface designed to manage virtual machines using libvirt, with a primary focus on KVM VMs. This guide will provide step-by-step instructions for installing and configuring virt-manager on Arch Linux.

## Install virt-manager on Arch Linux

To install `virt-manager` on Arch Linux, execute the following command:

```bash
sudo pacman -S virt-manager dnsmasq qemu-img qemu-system-x86
```

To ensure the functioning of `virt-manager`, you will need to start the `libvirtd.service` systemd unit.

**_Note_**: While `virt-manager` utilizes dnsmasq in NAT network mode, it is not required to start the `dnsmasq` service.

You may choose to keep the dnsmasq service disabled during boot time and only start it when necessary.

```bash
sudo systemctl start libvirtd.service
```

To grant your user access to the **libvirt** daemon, add the user as a member of the `libvirt` user group. By default, members of the **libvirt** group have passwordless access to the RW daemon socket.

```bash
sudo usermod -a -G libvirt <user>
```

## Running virtual machines

When utilizing NAT network mode, it is necessary to define and start a network in libvirt. By default, libvirt has a network defined in `/etc/libvirt/qemu/networks/default.xml`. To make use of this network, execute the following command::

```bash
sudo virsh
```

Inside the **virsh** cli, initiate the network by entering the following command:

```
virsh # net-start default
```

If you want the network to be enabled by default every time you start the **libvirtd** service,
run the following commands:

```
virsh # net-start default
virsh # net-autostart default
```