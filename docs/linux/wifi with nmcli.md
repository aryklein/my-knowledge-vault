---
tags:
  - cheatsheet
  - linux
  - networking
  - wireless
---
# nmcli quick cheat sheet

`nmcli` (Network Manager Command Line Interface) is a command-line tool used for
controlling NetworkManager and for reporting network status.

Here are some common `nmcli` tasks, including connecting to a Wi-Fi network:

## Listing Available Wi-Fi Networks

Lists all available Wi-Fi networks

```bash
nmcli device wifi rescan
nmcli device wifi list
```

## Connecting to a Wi-Fi Network

```bash
nmcli device wifi connect [SSID] password [password]
```

Replace `[SSID]` with the network's SSID and `[password]` with the network's
password.

## Disconnecting from a Wi-Fi Network

```bash
nmcli device disconnect iface [interface]
```

Replace `[interface]` with the name of the wireless interface (e.g., wlan0).

## Viewing Connection Status

```bash
nmcli connection show
```

Shows the status of all network connections.

## Enabling/Disabling a Network Connection

```bash
nmcli connection up [connection]
nmcli connection down [connection] ```

Replace `[connection]` with the name of the network connection. ## Displaying
All Network Interfaces

```bash
nmcli device status
```

Lists all network interfaces and their status
## Getting Detailed Information About a Connection

```bash
nmcli connection show [connection]
```

Replace `[connection]` with the name of the network connection for detailed
information

## Deleting a Saved Network

```bash
nmcli connection delete id [connection]
```

Replace `[connection]` with the name of the network you want to forget.
