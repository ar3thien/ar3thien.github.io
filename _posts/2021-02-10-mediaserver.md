---
layout: post
published: true
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
  1. Download movies __securely__
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
> apt install -y transmission-daemon minidlna __cifs-utils__
```

Install MongoDB v3.6
```shell
> wget -qO - https://www.mongodb.org/static/pgp/server-3.6.asc | sudo apt-key add -
> echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/3.6 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list
> sudo apt-get update && apt-get install -y mongodb-org
> sudo systemctl start mongod
> sudo systemctl enable mongod
```

Obtain Omada Controller v4 binary from [TP-Link official website][omada-download] and install it
```shell
> cd /tmp
> wget https://static.tp-link.com/2020/202012/20201211/omada_v4.2.8_linux_x64.deb
> dpkg -i omada_v4.2.8_linux_x64.deb
```

# __Resource Usage__
---  
<br>



[transmission-info]: https://transmissionbt.com/
[debian-info]: https://www.debian.org/
[minidlna-info]: https://wiki.debian.org/minidlna
[lxc-info]: https://linuxcontainers.org/lxc/introduction/
[proxmox-info]: https://www.proxmox.com/en/proxmox-ve