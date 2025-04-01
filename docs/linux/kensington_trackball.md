---
tags:
  - linux
---
# Remapping Kensington Expert Trackball Buttons with wayland

If you want to swap the order of buttons on your Kensington trackball mouse
using `hwdb`, you'll first need to identify the device and then create an
appropriate rule. Here's a step-by-step guide to help you achieve this

1) Identify the Device:

Plug in your Kensington trackball and use the `evtest` command (you might need
to install it first) to list all input devices:

```bash
sudo evtest
```

From the list, identify your trackball device. It'll likely be named something
related to Kensington.

2) Get the modalias matching string:

The matching rule in the `hwdb` entry is just the USB Vendor + Product. You can
get it with

```bash
lsusb | grep Kensington
```

```txt
Bus 003 Device 014: ID 047d:8018 Kensington Expert Wireless Trackball Mouse
(K72359WW)
```

In this example it is:

```txt
evdev:input:b*v047dp8018*
```

3) Write the `hwdb` Rule:

With the modalias matching string in hand, you can create a custom `hwdb` file.
Let's name it `99-kensington-trackball.hwdb` and place it in
`/etc/udev/hwdb.d/`.

If, for instance, you want to swap button 1 with button 2, and the modalias
matching string you got was `evdev:input:b*v047dp8018*`, your file would look
something like this:

```txt
# Kensington Trackball Button Remap
evdev:input:b*v047dp8018*
 KEYBOARD_KEY_<scan_code_of_button1>=btn_side
 KEYBOARD_KEY_<scan_code_of_button2>=btn_middle
```

Here, you need to replace `<scan_code_of_button1>` and `<scan_code_of_button2>`
with the actual scan codes of the buttons. You can find these scan codes using
`evtest`

```bash
sudo evtest
```

And then clicking on the button you wanna change.

It will look like this:

```txt
Event: time 1693774835.443847, -------------- SYN_REPORT ------------
Event: time 1693774835.519771, type 4 (EV_MSC), code 4 (MSC_SCAN), value 90003
Event: time 1693774835.519771, type 1 (EV_KEY), code 275 (BTN_SIDE), value 0
Event: time 1693774835.519771, -------------- SYN_REPORT ------------
Event: time 1693774835.569792, type 4 (EV_MSC), code 4 (MSC_SCAN), value 90004
Event: time 1693774835.569792, type 1 (EV_KEY), code 274 (BTN_MIDDLE), value 1
Event: time 1693774835.569792, -------------- SYN_REPORT ------------
```

In this example. they are **90003** and **90004**

4) Update hwdb:

After writing the rule, update the hwdb:

```bash
sudo systemd-hwdb update
```

5) Trigger `udev`:

```bash
sudo udevadm trigger
```

Source:
- https://www.reddit.com/r/linux4noobs/comments/fih5aw/how_to_change_or_assign_the_mouse_buttons_in/