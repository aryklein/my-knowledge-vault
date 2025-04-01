---
tags: linux, command
---

# lsoft

`lsof` (short for "list open files") is a command-line utility in Unix-like
operating systems that allows you to list information about files that are
currently opened by processes on your system. Here are some useful lsof
commands:

### List open files for a specific process ID (PID)

You can specify a process ID to list all open files associated with that
process.

```sh
lsof -p <PID>
```

### List open files for a specific user

This command lists all open files owned by a specific user.

```sh
lsof -u <username>
```

### List open files for a specific IP address or port

You can use the `-i` option with a port number to filter open network
connections. You can identify which process is using a port:

```sh
sudo lsof -i :<port_number>
```

### List open network connections

This command displays all network connections established by processes on the
system.

```sh
sudo lsof -i
```

### List open files by a specific program or command name

You can use the `-c` option to filter open files by a specific program or
command name.

```sh
lsof -c <program_name>
```
