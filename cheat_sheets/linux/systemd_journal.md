---
tags: linux, systemd
---
# systemd/Journal

The systemd journal is a logging solution that systemd provides, replacing traditional syslog daemons in many environments. It collects not just syslog messages, but also kernel logs, standard output from services, messages logged before the daemon was started, and even, optionally, early boot messages.

To interact with systemd journal, you primarily use the journalctl command. This tool allows you to query the journal using various criteria, such as the time the message was logged, the service that logged the message, the log level, and more.

## journactl command

`journalctl` is a versatile tool for interacting with the `systemd` journal. Here are some useful commands:

### Display all logs

Simply running `journalctl` will show all logs in the system. The list is typically quite long, and older entries are displayed first.

```sh
journalctl
```

### Display logs from the current boot

This shows only the logs generated during the current boot session.

```sh
journalctl -b
```

### Display logs from a specific service

To show only the logs from a particular service, use `-u` followed by the service name. For example, to view logs from the `sshd` service:

```sh
journalctl -u sshd
```

### Display logs for a specific process

If you want to see logs from a specific process, use `_PID=` followed by the process id. For example:

```sh
journalctl _PID=1001
```

### Display logs within a specific time frame

You can specify a time range to display logs within that period. The format is "YYYY-MM-DD HH:MM:SS". Here is an example:

```sh
journalctl --since="2023-06-14 00:00:00" --until="2023-06-14 12:00:00"
```

### Follow new entries

This works similar to the tail -f command and shows new entries in real time.

```sh
journalctl -f
```

### Display logs with a specific priority

Log messages are assigned a priority level from 0 (emerg) to 7 (debug). To see only messages with a specific priority or higher, use `-p`. For example, to display warnings and more severe issues:

```sh
journalctl -p warning
```

### Display kernel messages

This is similar to the `dmesg` command and will show only kernel messages.

```sh
journalctl -k
```

### Display a specific number of lines

Similar to the tail command, this will show only the last n lines.

```sh
journalctl -n 20
```

### Display disk usage by journal files

This shows how much disk space the journal logs are consuming.

```sh
journalctl --disk-usage
```

### Vacuum Journal

This command will delete the oldest logs until the disk usage reaches the specified size.

```sh
sudo journalctl --vacuum-size=500M
```

or by days:

```sh
sudo journalctl --vacuum-time=1d
```