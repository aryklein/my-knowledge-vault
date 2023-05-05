# How to set up a Yubikey in Arch Linux

Yubikey is a hardware authentication device that can be used for two-factor authentication,
passwordless login, and other security features. This document shows how to set up a Yubikey
in Arch Linux.

## Installation
To use Yubikey Authenticator app in Linux, you need to install the `ccid` package and start the
`pcscd` service. The `ccid` package provides a generic USB interface driver for smart card readers.

```bash
sudo pacman -S ccid
sudo systemctl start pcscd.service
sudo systemctl enable pcscd.service
```

## Configuration
Once you have installed the necessary packages and started the service, you can configure the
Yubikey for use in Arch Linux.

### Install the Yubikey Management Tools
To manage your Yubikey, you need to install the Yubikey management tools. Use the following
command to install the tools:

```bash
sudo pacman -S yubikey-manager yubikey-manager-qt
```

#linux #archlinux 