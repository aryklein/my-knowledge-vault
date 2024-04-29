---
tags:
  - cheatsheet
  - android
---
## Change Network mobile parameters

In the dialer app, ingress:
```
*#*#4636#*#
```

## Sideload a full OTA update

This has the same effect as flashing the corresponding factory image, but
without the need to wipe the device or unlock the bootloader.

1) Download  the `adb` tool. In Arch Linux:
```bash
sudo pacman -S android-tools android-udev
```
2) Make sure the user is in `plugdev` group
3)  Download the OTA update from
[here](https://developers.google.com/android/ota)
4) Enable USB debugging mode in Android, plug the USB cable and execute:
```bash
adb devices
```
This command should be executed twice: one will request permission in your Pixel
and the second time should list your pixel. If that doesn't work, don't continue
5) Reboot in recovery mode:
```bash
adb reboot recovery
```
6) To access the recovery menu, hold the **Power** button and press **Volume
Up** once. The recovery text menu will appear.
7) To enter sideload mode, select the option `Apply update from ADB`.
8) Run the following command to check that the device is in sideload mode:
```bash
adb devices
```
9) Sideload the zip file:
```bash
adb sideload ota_file.zip
```
10) Once the update finishes, reboot the phone by choosing Reboot system now
11) Disable  USB debugging mode
