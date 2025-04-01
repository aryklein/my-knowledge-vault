---
tags:
  - cheatsheet
  - linux
---

# SSH SOCKS5 Proxy

You can set up a SOCKS5 proxy with SSH in Linux using the ssh command's `-D`
option. Here is how you can do it:

```bash
ssh -D 8080 -C -q -N [user]@[your_ssh_server]
```

Here is what each flag does:

`-D` 8080 tells SSH that we want to open up a **SOCKS5** proxy on localhost port
8080.
`-C` requests compression of all data.
`-q `uses quiet mode.
`-N` tells SSH that no command will be sent once the tunnel is up.

You should replace `[user]` with your SSH username and `[your_ssh_server]` with
the IP address or hostname of your SSH server.

Enter your password when prompted.

Leave the terminal open. The SSH connection will stay alive as long as the
terminal is open.

Now, you can configure your browser to use this proxy server by executing with
the `--proxy-server` argument:

```bash
chromium --proxy-server='socks5://localhost:8080'
```
