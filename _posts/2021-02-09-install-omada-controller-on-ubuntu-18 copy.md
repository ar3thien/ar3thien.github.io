---
layout: post
# title:  "Install Omada Controller on Ubuntu 18.04"
# date:   2021-02-09 18:43:09 +0100
# categories: jekyll update
---


# __Introduction__
---
<br>
In this tutorial I will show you how to install [TP-Link Omada Controller][omada-info] on a [Ubuntu 18.04][ubuntu-18] server.

My environment is hosted on a [Proxmox][proxmox-info] hypervisor and running on a [LXC][lxc-info] container.

The container will be running with the following resources:
  * 1 vCPU
  * 512 MB RAM
  * 4 GB disk


# __Installation__
---
<br>
First step is to update the repo and upgrade to get the latest packages
```shell
> sudo apt-get update && apt-get -y upgrade
```

Install the required dependencies
```shell
> sudo apt-get -y install openjdk-8-jre-headless jsvc curl gnupg
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



[omada-download]: https://www.tp-link.com/en/support/download/omada-software-controller/#Controller_Software
[omada-info]: https://www.tp-link.com/en/omada-sdn/
[ubuntu-18]: https://releases.ubuntu.com/18.04/
[lxc-info]: https://linuxcontainers.org/lxc/introduction/
[proxmox-info]: https://www.proxmox.com/en/proxmox-ve