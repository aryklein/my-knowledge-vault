---
tags:
  - virtualization
  - vagrant
  - linux
---
# Vagrant with QEMU/KVM

QEMU/KVM  is the default full virtualization system used in GNU/Linux systems.
KVM primarily provides the kernel module `kvm.ko`, which has been integrated
into the Linux kernel for years. It utilizes user-space tools from the **QEMU**
project, hence the correct name for the complete virtualization system is
QEMU/KVM.

It is quite common not to use KVM directly but instead to do so through
**libvirt**, which is a virtualization API and an independent project that
allows us to interact with various virtualization systems, including KVM.

Vagrant is a free application developed in Ruby that enables us to create and
customize lightweight, reproducible, and portable development environments.
Vagrant allows us to automate the creation and management of virtual machines.
The virtual machines created by Vagrant can be run on various virtual machine
managers, including VirtualBox, VMWare, and Hyper-V.

The main goal of Vagrant is to bridge the gap between development and production
environments. This way, developers have a straightforward way to deploy an
infrastructure similar to what will be used in production environments. It also
makes it easier for system administrators to create test and development
infrastructures.

In this document, I will introduce the use of Vagrant with the
`vagrant-libvirt`plugin, which allows to set up scenarios in Vagrant using
virtualization provided by KVM. Specifically, Vagrant, using the libvirt API,
will manage resources in the KVM virtualization system.

## Vagrant and libvirt

Not officially supported, we can create scenarios in Vagrant using libvirt +
QEMU/KVM. To do this, you can follow the documentation for the [Vagrant Libvirt
Provider plugin](https://github.com/vagrant-libvirt/vagrant-libvirt)

## Vagrant

Once you have [[[[virt-manager]]], you can install Vagrant. In Arch Linux:

```bash
sudo pacman -S vagrant
```

And to install the libvirt provider:

```bash
vagrant plugin install vagrant-libvirt
```

You can check/update installed plugins with:

```bash
vagrant plugin list
vagrant plugin update
```

### Installing a box

Boxes are pre-configured virtual machine images used by Vagrant. You can obtain
them from the official [Vagrant Cloud](https://app.vagrantup.com/boxes/search)
repository. For example `generic/ubuntu2304`:

```bash
vagrant box add generic/ubuntu2304
```

It's important to note that I'm performing these actions with non-privileged
users. Each user will have their own set of boxes.

To check the boxes installed that my user has installed:

```bash
vagrant box list
```

## Creating a Vagrant VM using the previous box

Create a directory, and inside it, create the `Vagrantfile`. You can create an
empty one using the following command:

```bash
mkdir test_vm
cd test_vm
vagrant init
```

Change the `Vagrantfile` to look like this:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2304"
  config.vm.hostname="test-vm"
  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = "kvm"
    libvirt.connect_via_ssh = false
    libvirt.memory = 1024
    libvirt.cpus = 2
  end
end
```

Start the VMs with:

```bash
vagrant up
```

To check the  resources created:

```bash
virsh -c qemu:///system list
virsh -c qemu:///system net-list
ip -4 --brief ad ls
```
Or you can run:

```bash
vagrant status
```

To create more than one host, you need to define another one in the Vagrantfile:

```ruby
Vagrant.configure("2") do |config|
  config.vagrant.plugins = "vagrant-libvirt"
#
  config.vm.define :test_vm_1 do |test_vm_1|
    test_vm_1.vm.box = "generic/ubuntu2304"
    test_vm_1.vm.hostname = "test-vm-1"
    test_vm_1.vm.provider :libvirt do |libvirt|
      libvirt.connect_via_ssh = false
      libvirt.driver = "kvm"
      libvirt.cpus = 4
      libvirt.memory = 4096
    end
  end
#
  config.vm.define :test_vm_2 do |test_vm_2|
    test_vm_2.vm.box = "generic/ubuntu2304"
    test_vm_2.vm.hostname = "test-vm-2"
    test_vm_2.vm.provider :libvirt do |libvirt|
      libvirt.connect_via_ssh = false
      libvirt.driver = "kvm"
      libvirt.cpus = 4
      libvirt.memory = 4096
    end
  end
end
```

For more provider options check [this
document](https://github.com/vagrant-libvirt/vagrant-libvirt/blob/main/docs/configuration.markdownj)

## Ansible and Vagrant

You can ran an Ansible playbook (called `playbook.yaml` in this example)
byadding the following lines to the VMs definition:

```ruby
Vagrant.configure("2") do |config|
  config.vagrant.plugins = "vagrant-libvirt"
  config.vm.define :test_vm_1 do |test_vm_1|
    test_vm_1.vm.box = "generic/ubuntu2304"
    test_vm_1.vm.hostname = "test-vm-1"
    test_vm_1.vm.provider :libvirt do |libvirt|
      libvirt.connect_via_ssh = false
      libvirt.driver = "kvm"
      libvirt.cpus = 4
      libvirt.memory = 4096
    end
    test_vm_1.vm.provision :ansible do |ansible|
      ansible.playbook = "playbook.yaml"
    end
  end

  config.vm.define :test_vm_2 do |test_vm_2|
    test_vm_2.vm.box = "generic/ubuntu2304"
    test_vm_2.vm.hostname = "test-vm-2"
    test_vm_2.vm.provider :libvirt do |libvirt|
      libvirt.connect_via_ssh = false
      libvirt.driver = "kvm"
      libvirt.cpus = 4
      libvirt.memory = 4096
    end
    test_vm_2.vm.provision :ansible do |ansible|
      ansible.playbook = "playbook.yaml"
    end
  end
```

Vagrant will generate an inventory file encompassing all of the virtual machines
it manages, and use it for provisioning machines.

Note that the generated inventory file is stored as part of your local Vagrant
environment in
`.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory`
