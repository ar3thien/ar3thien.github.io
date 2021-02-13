---
layout: post
title:  "AWS - Monitoring Daily Usage In a Multi-Account Environment"
published: false
# date:   2021-02-09 18:43:09 +0100
# categories: jekyll update
---

# __Objectives__
---
<br>
  1. Monitor daily AWS spending to the closest minimum interval
  2. Display it on a dashboard
  3. Generate alerts and get notified

# __Introduction__
---
<br>
I had a need to closely monitor my AWS account for daily usage, knowing that anyone can easily exceed his planned budget, by provisionning an oversized instance, or simply by forgetting terminating unused ones. Usually those instance are dev and test instances. And if you're in an enviroment where dev-ops teams have a lot to do, monitoring daily spending becomes a __MUST__.

After some research, I found out that the best way to monitor my daily AWS account usage is using CloudWatch EstimatedCharges metric.
By googling a little while writing this post, I saw that AWS has now an [official KB][aws-doc] on how to implement this, although when I implemented the solution there were none.

After comparing their KB to my solution, although they look the same, I found few differences. I will explain them, and I they have a bug in their solution based on my need.

# __What's the issue?__
---
<br>
I completely agree with all the content in AWS KB, modulo the expression used to calculate the daily usage:
```
AWS expression: RATE(m1) * 86400
My expression:  IF(RATE(m1) < 0, m1, RATE(m1)*PERIOD(m1))
```

Based on [AWS CloudWatch documentation][cw-mathmetric], the RATE expression returns the rate of change of the metric per second. This is calculated as the difference between the latest data point value and the previous data point value, divided by the time difference in seconds between the two values.

The bug that will happen by using AWS expression is that at 00:00:00 of the first day of every month the EstimatedCharges metric will be 0. That means the expression will result in a negative value, which will break my dashboard.

The graph below shows when the problem will occur before applying the CloudWatch expression:

[ ![](/assets/aws-estimatedcharges-problem.png) ](/assets/aws-estimatedcharges-problem.png)

Now applying AWS expression will give the following graph, emphisizing more the bug:

[ ![](/assets/aws-estimatedcharges-rate-problem.png) ](/assets/aws-estimatedcharges-rate-problem.png)

Finally applying my expression will generate the following graph:

[ ![](/assets/aws-estimatedcharges-rate-fixed.png) ](/assets/aws-estimatedcharges-rate-fixed.png)

# __Final Thoughts__
---
<br>
If you have multiple AWS accounts and you'd like to automate monitoring their daily charges, check this post that explains how to do it.

[omada-download]: https://www.tp-link.com/en/support/download/omada-software-controller/#Controller_Software
[omada-info]: https://www.tp-link.com/en/omada-sdn/
[ubuntu-18]: https://releases.ubuntu.com/18.04/
[lxc-info]: https://linuxcontainers.org/lxc/introduction/
[proxmox-info]: https://www.proxmox.com/en/proxmox-ve
[aws-doc]: https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/
[cw-mathmetric]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html