# Manage virtual machines with virt-manager

Virt-manager is a desktop user interface designed for managing virtual machines through libvirt,
primarily targeting KVM VMs. This guide will walk you through the installation and configuration
of virt-manager on Arch Linux.

## Install virt-manager on Arch Linux

To install virt-manager on Arch Linux, execute the following command:

```bash
sudo pacman -S virt-manager dnsmasq qemu-img qemu-system-x86
```

Since `virt-manager` relies on `libvirt`, you will need to start the `libvirtd.service` systemd
unit:

**Note**: Although `virt-manager` uses dnsmasq in NAT network mode, it is not necessary to start
the `dnsmasq` service.

You may prefer to keep the service disabled at boot time and only start it when needed:

```bash
sudo systemctl start libvirtd.service
```

To ensure your user has access to the libvirt daemon, add the user as a member of the `libvirt`
user group. By default, members of the libvirt group have passwordless access to the RW daemon
socket.

```bash
sudo usermod -a -G libvirt <user>
```

## Running virtual machines

When using NAT network mode, you will need to define and start a network in libvirt. By default,
libvirt has a network defined in `/etc/libvirt/qemu/networks/default.xml`. To use this network,
run:

```bash
sudo virsh
```

Within the virsh CLI, start the network by entering:

```
virsh # net-start default
```

If you want the network to be enabled by default every time you start the **libvirtd** service,
run the following commands:

```
virsh # net-start default
virsh # net-autostart default
```

#linux #vm #virtualization #archlinux
