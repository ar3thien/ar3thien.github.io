---
layout: post
title:  "AWS - Fixing EstimatedCharges Daily Monitoring"
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

After comparing their KB to my solution, although they look the same, I found few differences. I will explain them, and I believe the doc havs bug in regards to my need.

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

Now applying AWS expression will give the following graph, emphasizing more the bug:

[ ![](/assets/aws-estimatedcharges-rate-problem.png) ](/assets/aws-estimatedcharges-rate-problem.png)

Finally applying my expression will generate the following graph:

[ ![](/assets/aws-estimatedcharges-rate-fixed.png) ](/assets/aws-estimatedcharges-rate-fixed.png)

<br>
On the other hand, I'm using the PERIOD function instead of a fixed period number, which will allow me to inherit the value of the PERIOD of the graph on which the alert is applied.
In my CloudWatch alarms, I use a period of 6 hours, because of my requirement to have a monitoring to the closest minimum interval.
A 6 hours interval will allow me to be more reactive to any unexpected spending.

# __Final Thoughts__
---
<br>
If you have multiple AWS accounts and you'd like to automate monitoring their daily charges, check [this post][org-link] that explains how to do it.

[aws-doc]: https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/
[cw-mathmetric]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html
[org-link]: https://www.mangito.ga/2021/02/13/aws-org-billing-alerts.html