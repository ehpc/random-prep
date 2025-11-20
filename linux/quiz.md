# How to setup a daemon in linux

A Linux daemon is a background process not tied to a terminal. 
There are multiple ways to set one up, but today the standard, recommended 
solution is systemd. Other methods exist, especially for older systems or 
containers.

1. systemd service (modern, recommended)

This is the primary method on almost all modern Linux distros 
(Ubuntu, Debian, Fedora, Arch, RHEL).

Create a unit file:

/etc/systemd/system/myapp.service:

```sh
[Unit]
Description=My App Service
After=network.target

[Service]
ExecStart=/usr/local/bin/myapp
Restart=on-failure
User=myuser
WorkingDirectory=/opt/myapp

[Install]
WantedBy=multi-user.target
```

```sh
sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp
```

2. init.d script (SysV init) — legacy

Older method (pre-systemd) using shell scripts in /etc/init.d.

Example:

```sh
#!/bin/sh
case "$1" in
  start) /usr/bin/myapp & ;;
  stop) killall myapp ;;
  restart) ... ;;
esac
```

```sh
sudo update-rc.d myapp defaults
```

