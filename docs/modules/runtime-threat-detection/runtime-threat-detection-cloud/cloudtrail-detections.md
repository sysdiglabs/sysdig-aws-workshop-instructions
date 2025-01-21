---
title: Runtime Threat Detection and CloudTrail
parent: Runtime Threat Detection in Cloud
nav_order: 1
---

{: .goal }
> At the end of this module, you will understand that Sysdig can detect threats not only in hosts, but also in cloud environments, by analyzing CloudTrail logs and performing real-time analysis.

## CloudTrail Detections

At this point, you have logged in to AWS with your workshop user. However, you may have noticed that you are not using Multi-Factor Authentication (MFA) to do so. In the real world, you should always use MFA to log in to your AWS account, but in this workshop you don't, simply for ease of use.

But, Sysdig will detect the lack of MFA and alert you on it!

Go to **Threats** -> **Events Feed**.

There, select **AWS** under the **Event Source** filter near the top of the page.

!["cloudtrail-detection"]({{site.baseurl}}/assets/images/aws-no-mfa-alert.png)

In the **Events Feed** pane, you'll see a couple of AWS Cloudtrail alerts. One is for the SSM session where you logged in to your bastion host, and the other is for a login with no MFA.

!["cloudtrail-detection-2"]({{site.baseurl}}/assets/images/aws-no-mfa-alert2.png)

Sysdig will also alert you on the lack of MFA in the login event, as well as the fact that an SSM session was used to log in.

## Completed

You have now completed the CloudTrail detections sections of this module.

{: .value }
> Organizations often have little visibility into what their cloud environments are doing, what permissions they have and what actions are being taken with those resources. Sysdig can provide visibility and threat detection capabilities to your cloud infrastructure.
