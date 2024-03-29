---
tags: linux, networking, cheatsheet
---
# Network Troubleshooting on Linux

Listed below are essential Linux commands I'm compiling for network
troubleshooting on Linux. The goal is to expedite the process and eliminate
repeated searches when seeking useful arguments.
## List opened ports in a Linux system

```bash
sudo lsof -i -n -P | grep LISTEN
```

`netstat` is a command used to display network connections on Linux machines.
However, it is now considered a deprecated command because it has been replaced
by `ss` command, which offers more advanced features and better control over
network connections

```bash
ss -tuln
```

## Display only network interfaces with IPv4 address assigned

```bash
ip -4 -brief ad ls
```

## Display bandwidth usage on an interface by host

```bash
sudo iftop [-i interface]
```