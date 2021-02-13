---
layout: post
title:  "AWS - Monitor Daily EstimatedCharges"
# date:   2021-02-09 18:43:09 +0100
# categories: jekyll update
---

# __Introduction__
---
<br>
I had a need to closely monitor my AWS account for daily usage, knowing that anyone can easily exceed his planned budget, by provisionning an oversized instance, or simply forgetting terminating unused instances. Usually those instance are dev and test instances. And if you happen you're in an enviroment where dev-ops have a lot to manage and maintain, monitoring daily spending becomes a __MUST__.
After some research, I found out that the best way to monitor my daily AWS account usage is using CloudWatch EstimatedCharges metric.
By googling a little while writing this post, I saw that AWS has now an [official KB][aws-doc] on how to implement this.
After comparing their KB to my implementation, although the base solution is the same, however there are some differences. I will explain them, and I believe they have few bugs that will occur in their implementations.

I completely agree with all the content in AWS KB modulo the expression used to calculate the daily usage:
```
AWS expression: RATE(m1) * 86400
My expression: IF(RATE(m1) < 0, m1, RATE(m1)*PERIOD(m1))
```

Based on [AWS CloudWatch documentation][cw-mathmetric], the RATE expression returns the rate of change of the metric per second. This is calculated as the difference between the latest data point value and the previous data point value, divided by the time difference in seconds between the two values.

The bug that will happen by using AWS expression is that at the first day of every month the EstimatedCharges metric will be lower than the last metric of the last of the previous month. That means the expression will give a negative value, which will __NEVER EVER__ trigger the daily alarm at the first day of the month. Meaning that you can consume 1000$ just on your first day of the month, if you have consumed 1001$ on the previous whole month.

As stated in my objective, I want to be able to monitor the spending to the closest minimum interval. CloudWatch EstimatedCharges metric is published at approximately six-hour. So instead of multiplying by 86400 which is equivalent to 1 day, why not multiplying by the metric period itself, which is 6 hours, and that way I can get more accurate results on my spending, hence better governance.

In this tutorial I will show you how to install [TP-Link Omada Controller][omada-info] on a [Ubuntu 18.04][ubuntu-18] machine.

My environment is hosted on a [Proxmox][proxmox-info] hypervisor and running on a [LXC][lxc-info] container.

The container will be running with the following resources:

| __CPU__ | 1 |
| __RAM__ | 512 MB |
| __Disk__ | 4 GB |
| __VLAN__ (Optional) | Default |

# __Objectives__
---
<br>
  1. Monitor daily AWS spending to the closest minimum interval
  2. Generate alerts and get notified

# __Installation__
---
<br>
I'm starting from a fresh Ubuntu installation, so I will update the repo and upgrade the existing packages
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

Finally set the server timezone
```shell
> timedatectl set-timezone <timezone>
```

# __Final Thoughts__
---
<br>

[omada-download]: https://www.tp-link.com/en/support/download/omada-software-controller/#Controller_Software
[omada-info]: https://www.tp-link.com/en/omada-sdn/
[ubuntu-18]: https://releases.ubuntu.com/18.04/
[lxc-info]: https://linuxcontainers.org/lxc/introduction/
[proxmox-info]: https://www.proxmox.com/en/proxmox-ve
[aws-doc]: https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/
[cw-mathmetric]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html