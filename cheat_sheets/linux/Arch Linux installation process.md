# Arch Linux installation process

This guide describes the installation steps, packages, and settings that I usually use on my personal laptop.

## Create bootleable pendrive

Download the current ISO from [here](https://archlinux.org/download/) and create a bootable pendrive with the following
command:

```bash
sudo dd bs=4M if=archlinux-<release_date>-x86_64.iso of=/dev/<pendrive> status=progress oflag=sync
```
Replace `release_date` with the proper relase date and `pendrive` with the mapped device.

## Installation from scratch

### Boot the pendrive

On most Dell laptops, you can choose this option by pressing *F12* key at boot time.

### Connect to WiFi on Arch Linux installer

```bash
iwctl
[iwd]# device list
...
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
...
[iwd]# station connect some-wifi-network
[iwd]# station wlan0 show
...
[iwd]# exit
ping www.google.com
```

### Update the system clock

Use `timedatectl` to ensure the system clock is accurate by enabling NTP:

```bash
timedatectl set-ntp true
```

### Partition and format the disk

In my case I use `fdisk` but you can use `parted`.

```bash
fdisk /dev/the_disk_to_be_partitioned
```

Use GTP partition table if your laptop support it. Three partition will be enought to install Arch Linux:

- EFI: Type 1 - `EFI System` formated with FAT32. 512 MiB is enough.
- root: Type 20 - `Linux Filesystem` formated with ext4
- swap: Type 19 - `Linux Swap`. Swap size should be at least the size of RAM to support hibernation.


Once the partitions have been created, format the partitions with the appropriate file system:

```bash
mkfs.fat -F 32 /dev/efi_partition
mkfs.ext4 /dev/root_partition
mkswap /dev/swap_partition
```

### Mount the file systems

Mount the root partition to `/mnt`:

```bash
mount /dev/root_partition /mnt
```

Create the **efi** mount point:

```bash
mkdir /mnt/efi
mount /dev/efi_partition /mnt/efi
```

Enable the swap partition:

```bash
swapon /dev/swap_partition
```

### Essential packages installation

Check the list of mirror servers that the live system has in `/etc/pacman.d/mirrorlist`. By default it comes with a
list generated by [reflector](https://wiki.archlinux.org/title/Reflector). You can edit this list and move the
geographically closest mirrors to the top of the list.

In my case I use:

```bash
reflector -c 'United States' -c Canada --latest 5 --protocol https --save /etc/pacman.d/mirrorlist
```

Use the pacstrap script to install the base package:

```bash
pacstrap /mnt base linux linux-firmware
```

### Configure the system

1) Generate the `fstab` file:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

2) Chroot into the new system:

```bash
arch-chroot /mnt
```

3) Set the timezone:

```bash
ln -sf /usr/share/zoneinfo/America/Argentina/Cordoba /etc/localtime
```

4) Generate the `/etc/adjtime` file:

```bash
hwclock --systohc
```

5) Edit `/etc/locale.gen`, uncomment `en_US.UTF-8 UTF-8` and other needed locales, and execute:

```bash
locale-gen
```

6) Create the `locale.conf` file with:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

7) Set the keyboard layout:

```bash
echo "KEYMAP=us" > /etc/vconsole.conf
```

8) Set the hostname:

```bash
echo myhostname > /etc/hostname
```

9) Add matching entries to the `/etc/hosts` file:

```bash
127.0.0.1	localhost
::1		    localhost
127.0.1.1	myhostname.localdomain	myhostname
```

10) Create a new initramfs:

```
mkinitcpio -P
```

11) Set the root password:

```
passwd root
```

12) Install a boot loader (in my case GRUB) and microcode:

```bash
pacman -S grub efibootmgr os-prober intel-ucode
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=arch_grub
grub-mkconfig -o /boot/grub/grub.cfg
```

If Windows is installed in another partition, uncomment or add `GRUB_DISABLE_OS_PROBER=false` in the
`/etc/default/grub` file and re-run `grub-mkconfig -o /boot/grub/grub.cfg`.

13) Install the following packages before rebooting:

```bash
pacman -S openssh \
          polkit \
	  networkmanager
systemctl enable NetworkManager
```

14) Exit the chroot environment, umount mounted partitions and reboot.

15) Setup the network interface to have access to the Internet. You can use `nmtui`.

16) Install `doas` package, create a user and add it to the `wheel` group:

```bash
pacman -S doas
cd /usr/bin
ln -s doas sudo
cd ~
useradd -m -c "Ary Kleinerman" -s /bin/bash ary
passwd ary
gpasswd -a ary wheel
```

17) Create the `/etc/doas.conf` file and fix the permissions:

```bash
echo "permit nopass :wheel" > /etc/doas.conf
chown -c root:root /etc/doas.conf
chmod -c 0400 /etc/doas.conf
```

18) Enable NTP service:

```bash
doas timedatectl set-ntp true
```

### Install basic packages

(**note**: `intel-ucode` for intel `amd-ucode` for AMD)

Install the following basic packages:

```bash
pacman -Syu
pacman -S intel-ucode \
    linux-firmware \
    bash-completion \
    neovim \
    gopls \
    man-db \
    man-pages \
    base-devel \
    git \
    htop \
    bind-tools \
    tcpdump \
    reflector \
    tmux \
    zsh \
    zsh-completions \
    zsh-autosuggestions \
    zsh-syntax-highlighting \
    colordiff \
    python-virtualenv \
    python-pip \
    ipython \
    docker \
    docker-compose \
    cups \
    ntfs-3g \
    fzf \
    fd \
    ripgrep \
    stow \
    bat \
    exa \
    usbutils \
    mtpfs \
    bluez \
    bluez-utils \
    pipewire \
    pipewire-{alsa,jack,pulse} \
    wireplumber \
    kitty \
    gst-libav \
    dmidecode \
    ansible \
    ansible-lint \
    sysstat \
    nmap \
    freerdp
```

### Mirror List

Generate a good mirror list file (`/etc/pacman.d/mirrorlist`) by using the online
[generator](https://www.archlinux.org/mirrorlist/) or using `reflector`:

```bash
doas reflector -c 'United States' -c Canada --latest 5 --protocol https --save /etc/pacman.d/mirrorlist
doas pacman -Syu
```

### Colored pacman

Uncomment the `Color` line in `/etc/pacman.conf`.

### Docker and docker-compose

Add your user to the docker group:

```bash
doas systemct start docker
doas usermod -aG docker <user>
```

### Enable bluetooth service

```bash
doas systemctl enable bluetooth
```

**Note**: if you don't use GDM I notice that by default the Bluetooth adapter does not
power on after boot. If you have issues with this, set `AutoEnable=true` in
`/etc/bluetooth/main.conf` in the `[Policy]` section.

### Colorized files

To have colorized files according to the extension, generate `/etc/DIR_COLORS` with:

```bash
dircolors -p | doas tee /etc/DIR_COLORS
```
### Paru as AUR helper

Paru will be used to install AUR packages:

```
# set yo makepkg to use doas
echo PACMAN_AUTH=(doas) | doas tee -a /etc/makepkg.conf
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -s
doas pacman -U paru-xxx
cd ..
rm -rf paru
```

### NetworkManager setup with systemd-resolved**

1) Remove `/etc/resolv.conf`.

2) Enable `systemd-resolved.service` to work with NetworkManager:

```bash
systemctl enable systemd-resolved.service
```

3) Create the file `/etc/NetworkManager/conf.d/dns.conf` with:

```bash
[main]
dns=systemd-resolved
```

4) Make `/etc/resolv.conf` to be a symlink to `/run/systemd/resolve/stub-resolv.conf`:

```bash
doas ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

5) Reboot.

### Swaywm

Install the following non-AUR packages:

```bash
doas pacman -S sway \
    fuzzel \
    waybar \
    blueberry \
    xdg-desktop-portal \
    xdg-desktop-portal-wlr \
    khal \
    kanshi \
    pavucontrol \
    foot \
    swaylock \
    light \
    alacritty \
    grim \
    slurp \
    lxappearance \
    pcmanfm-gtk3 \
    qt5ct \
    qt5-wayland \
    qt6-wayland \
    lm_sensors \
    swayidle \
    swaybg \
    mako \
    flameshot \
    pulsemixer \
    qalculate-gtk \
    gnome-keyring \
    ttf-dejavu \
    ttf-hack \
    ttf-liberation \
    ttf-opensans \
    ttf-roboto \
    ttf-roboto-mono \
    ttf-hack-nerd \
    ttf-noto-nerd \
    ttf-dejavu-nerd
    noto-fonts \
    noto-fonts-emoji \
    libreoffice-still \
    hunspell-es_ar \
    hunspell-en_US \
    firefox \
    kubectl \
    gimp \
    obs-studio \
    youtube-dl \
    github-cli \
    mpv \
    android-udev \
    papirus-icon-theme \
    cups \
    system-config-printer \
    nautilus \
    gvfs-mtp \
    eog \
    libappindicator-gtk3 \
    gtk4 \
    go \
    pyright \
    xorg-xwayland \
    gnome-themes-extra \
    aws-cli-v2 \
    zsa-wally \
    swappy \
    kubectl \
    helmfile \
    transmission-gtk
```

Add your user to the video group to manage screen brightness:

```bash
doas gpasswd -a <user> video
```

Install the following AUR packages:

You need to execute:

```bash
# for Dropbox
gpg --recv-keys 1C61A2656FB57B7E4DE0F4C1FC918B335044912E
# for spotify
curl -sS https://download.spotify.com/debian/pubkey_5E3C45D7B312C643.gpg | gpg --import -
# for wob
gpg --keyserver keys.openpgp.org --receive-keys 5C6DA024DDE27178073EA103F4B432D5D67990E3
```

```bash
paru -S visual-studio-code-bin \
        brave-bin \
        zsh-theme-powerlevel10k-git \
        dropbox \
        nautilus-dropbox \
        teamviewer \
	zoom \
	slack-electron \
	spotify \
	tfswitch-bin \
	wob \
	helm-diff \
	appimagelauncher
```

## Hardware video acceleration

Hardware video acceleration makes it possible for the video card to decode/encode video, thus offloading the CPU and
saving power.

- HD Graphics series starting from Broadwell (2014) and newer are supported by `intel-media-driver`.
- GMA 4500 (2008) and newer GPUs, including HD Graphics up to Coffee Lake (2017) are supported by `libva-intel-driver`.

Install the following packages:

```bash
pacman -S libva-utils [intel-media-driver|libva-intel-driver]
```

More info [here](https://wiki.archlinux.org/title/Hardware_video_acceleration)

## Pipewire

Pipewire is a new low-level multimedia framework. It aims to offer capture and playback for both audio and video with
minimal latency and support for PulseAudio, JACK, ALSA and GStreamer-based applications. It replaces PlulseAudio. You can check if your system is using it with:

```bash
$ pactl info
```

If it says `Server Name: PulseAudio (on PipeWire 0.3.32)` it means it's already installed.

Execute `systemctl status --user pipewire-pulse.service` to see the result.

## Respecting the regulatory domain

To configure the regdomain, install `crda` package, edit `/etc/conf.d/wireless-regdom` and uncommenting the appropriate
domain, then reboot and check with:

```bash
doas pacman -S crda
doas sed -i 's/^#\(.*\)\("US"\)/\1\2/' /etc/conf.d/wireless-regdom
```

Reboot and check:

```bash
iw reg get
```

## GNOME/Keyring

When using console-based login, edit `/etc/pam.d/login` and add `auth optional pam_gnome_keyring.so` at the end of the
`auth` section and `session optional pam_gnome_keyring.so auto_start` at the end of the `session` section.

```sh
auth       required     pam_securetty.so
auth       requisite    pam_nologin.so
auth       include      system-local-login
auth       optional     pam_gnome_keyring.so
account    include      system-local-login
session    include      system-local-login
session    optional     pam_gnome_keyring.so auto_start
```