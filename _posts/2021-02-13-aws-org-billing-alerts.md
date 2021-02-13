---
layout: post
title:  "AWS - Monitoring Daily Usage In a Multi-Account Environment"
# published: false
# date:   2021-02-09 18:43:09 +0100
# categories: jekyll update
---

# __Objectives__
---
<br>
  1. Monitor daily AWS spending in a multi-account environment to the closest minimum interval
  2. Generate alerts and get notified by a nice looking email

# __Introduction__
---
<br>
One of the needs was to closely monitor AWS accounts daily spending in a multi-account environment. Multiple AWS accounts are usually part of AWS organizations and found in many enterprises that would like to split their prod, pre-prod, and test environments. Accounts can grow exponentially if the strategy is to split the accounts per activity as well. For someone governing the whole spending envelope, you need to ensure that a certain account does not exceed a certain amount. Also amounts can be different from account A to account B.

# __The Solution__
---
<br>

## __Member Account__
---
<br>
The below architecture describes the final outcome of this solution:

[ ![](/assets/billing-architecture.png) ](/assets/billing-architecture.png)

The solution consists of creating a `CloudWatch alarm` based on the EstimatedCharges metric. Let's name it "Billing-Alarm". For details on how to create this alarm, please refer to [AWS official KB][aws-doc].

Then create an `EventBridge rule` that will forward the generated alarm from a member account to the `EventBridge bus` in the billing account, based on the following pattern:

```json
{
  "detail-type": ["CloudWatch Alarm State Change"],
  "source": ["aws.cloudwatch"],
  "detail": {
    "alarmName": ["Billing-Alarm"],
    "state": {
      "value": ["ALARM"]
    }
  }
}
```

As stated previously, the `EventBridge rule` will forward the alarm to the `EventBridge bus` under the billing account. In order to so, it needs an IAM role that you're going to create. Let's call it "EventBridge-InvokeEventBus-Role".
It will have the following trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "events.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

And the following permissions policy, which you can create as inline policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "events:PutEvents"
            ],
            "Resource": [
                "arn:aws:events:us-east-1:<billing-account-id>:event-bus/default"
            ],
            "Effect": "Allow"
        }
    ]
}
```

# __Where to start?__
---
<br>
Since you're in an AWS Organizations structure, you can benefit from [CloudFormation stacksets][cf-stacksets] to deploy a template that will create the following resources:
1. A CloudWatch alarm based on a certain threshold
2. An EventBridge IAM role
3. An Event Bridge rule

Based on [AWS CloudWatch documentation][cw-mathmetric], the RATE expression returns the rate of change of the metric per second. This is calculated as the difference between the latest data point value and the previous data point value, divided by the time difference in seconds between the two values.

The bug that will happen by using AWS expression is that at 00:00:00 of the first day of every month the EstimatedCharges metric will be 0. That means the expression will result in a negative value, which will break my dashboard.

The graph below shows when the problem will occur before applying the CloudWatch expression:

[ ![](/assets/aws-estimatedcharges-problem.png) ](/assets/aws-estimatedcharges-problem.png)

Now applying AWS expression will give the following graph, emphisizing more the bug:

[ ![](/assets/aws-estimatedcharges-rate-problem.png) ](/assets/aws-estimatedcharges-rate-problem.png)

Finally applying my expression will generate the following graph:

[ ![](/assets/aws-estimatedcharges-rate-fixed.png) ](/assets/aws-estimatedcharges-rate-fixed.png)

On the other hand, I'm using the PERIOD function instead of a fixed number, which will allow me to inherit the value of the PERIOD of the graph on which the alert is applied.
In my CloudWatch alarms, I use a period of 6 hours, because of my requirement to have a monitoring to the closest minimum interval.

# __Final Thoughts__
---
<br>
If you have multiple AWS accounts and you'd like to automate monitoring their daily charges, check this post that explains how to do it.

[cf-stacksets]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html
[aws-doc]: https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/