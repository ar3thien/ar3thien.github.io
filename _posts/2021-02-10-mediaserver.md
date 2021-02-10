---
layout: post
# published: false
title:  "Install Transmission Client & Minidlna on Debian 10"
# date:   2021-02-09 18:43:09 +0100
# categories: jekyll update
---

# __Introduction__
---
<br>
In this tutorial I will show you how to install [transmission][transmission-info] and [minidlna][minidlna-info] on a [Debian 10][debian-info] machine.

My environment is hosted on a [Proxmox][proxmox-info] hypervisor and running on a [LXC][lxc-info] container.

The container will be running with the following resources:

| __CPU__ | 1, limit: 0.5 |
| __RAM__ | 96 MB |
| __Disk__ | 4 GB |
| __VLAN__ | 100 |

# __Objectives__
---
<br>
  1. Download media content __securely__
  2. Stream them to different devices at home

# __Installation__
---
<br>
I'm starting from a fresh Debian installation, so I will update the repo and upgrade the existing packages
```shell
> apt update && apt -y upgrade
```

Install the required dependencies
```shell
> apt install -y transmission-daemon minidlna cifs-utils
```

Mount an external storage (NAS)
```shell
> echo "//nas/public/Downloads  /mnt/downloads  cifs    noperm,guest,vers=1.0" > /etc/fstab
```

Stop tranmission service to alter its settings file, then start it
```shell
> systemctl stop transmission-daemon
> echo "{
      "alt-speed-down": 50,
      "alt-speed-enabled": false,
      "alt-speed-time-begin": 540,
      "alt-speed-time-day": 127,
      "alt-speed-time-enabled": false,
      "alt-speed-time-end": 1020,
      "alt-speed-up": 50,
      "bind-address-ipv4": "0.0.0.0",
      "bind-address-ipv6": "::",
      "blocklist-enabled": true,
      "blocklist-url": "https://github.com/sahsu/transmission-blocklist/releases/latest/download/blocklist.gz",
      "cache-size-mb": 4,
      "dht-enabled": true,
      "download-dir": "/mnt/downloads",
      "download-limit": 100,
      "download-limit-enabled": 0,
      "download-queue-enabled": true,
      "download-queue-size": 5,
      "encryption": 2,
      "idle-seeding-limit": 30,
      "idle-seeding-limit-enabled": false,
      "incomplete-dir": "/var/lib/transmission-daemon/Downloads",
      "incomplete-dir-enabled": false,
      "lpd-enabled": false,
      "max-peers-global": 200,
      "message-level": 1,
      "peer-congestion-algorithm": "",
      "peer-id-ttl-hours": 6,
      "peer-limit-global": 200,
      "peer-limit-per-torrent": 100,
      "peer-port": 51413,
      "peer-port-random-high": 65535,
      "peer-port-random-low": 49152,
      "peer-port-random-on-start": false,
      "peer-socket-tos": "default",
      "pex-enabled": true,
      "port-forwarding-enabled": false,
      "preallocation": 1,
      "prefetch-enabled": true,
      "queue-stalled-enabled": true,
      "queue-stalled-minutes": 30,
      "ratio-limit": 0,
      "ratio-limit-enabled": true,
      "rename-partial-files": true,
      "rpc-authentication-required": false,
      "rpc-bind-address": "0.0.0.0",
      "rpc-enabled": true,
      "rpc-host-whitelist": "",
      "rpc-host-whitelist-enabled": false,
      "rpc-password": "",
      "rpc-port": 9091,
      "rpc-url": "/transmission/",
      "rpc-username": "transmission",
      "rpc-whitelist": "127.0.0.1",
      "rpc-whitelist-enabled": false,
      "scrape-paused-torrents-enabled": true,
      "script-torrent-done-enabled": true,
      "script-torrent-done-filename": "/usr/bin/OpenSubtitlesDownload.py $TR_TORRENT_DIR",
      "seed-queue-enabled": false,
      "seed-queue-size": 10,
      "speed-limit-down": 100,
      "speed-limit-down-enabled": false,
      "speed-limit-up": 0,
      "speed-limit-up-enabled": true,
      "start-added-torrents": true,
      "trash-original-torrent-files": false,
      "umask": 18,
      "upload-limit": 100,
      "upload-limit-enabled": 0,
      "upload-slots-per-torrent": 14,
      "utp-enabled": true
  }" > /> /etc/transmission-daemon/settings.json
> systemctl start transmission-daemon
```

# __Resource Usage__
---  
<br>



[transmission-info]: https://transmissionbt.com/
[debian-info]: https://www.debian.org/
[minidlna-info]: https://wiki.debian.org/minidlna
[lxc-info]: https://linuxcontainers.org/lxc/introduction/
[proxmox-info]: https://www.proxmox.com/en/proxmox-ve