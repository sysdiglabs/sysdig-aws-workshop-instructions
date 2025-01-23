---
title: Exploiting Overpermissioned EKS Workloads
parent: Runtime Threat Detection in Cloud
nav_order: 2
---

{: .goal }
> Using the cloud is complicated, especially now that workloads can have their own identities and permissions. To secure our workloads, we need to use tools to understand exactly what permissions they have and what they, **and attackers**, are doing with them.
>
>At the end of this module, you will have an understanding of how EKS workloads can be over-permissioned with IAM Roles for Service Accounts (IRSA), how this can affect your security posture, and how to detect and prevent this with Sysdig.

1. TOC
{:toc}

## Exploiting Overpermissioned EKS Workloads

AWS EKS has a mechanism for giving Pod's access to the AWS APIs called [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). In short, this binds a particular service account in Kubernetes to an IAM Role in AWS - and will automatically mount credentials for using that AWS IAM role into any Pods that use that Kubernetes service account at runtime.

We've prepared an IRSA mapping already - the **irsa** ServiceAccount in the **security-playground** Namespace is bound to an AWS IAM Role that has the `"Action": "s3:*"` policy applied for an S3 bucket for your Attendee in this account. If you run the command below you'll see an Annotation on the ServiceAccount with the ARN of that IAM Role:

```bash
sudo bash; cd ~
kubectl get serviceaccount irsa -n security-playground -o yaml
```

It has the following in-line policy (check the IAM role in the AWS console) - one which we commonly see which is a `*` for the s3 service (really two to cover the bucket itself as well as the contents). It is properly scoped down to a single bucket Resource, which is better than nothing, but you'll see why a `*` for this service is a bad idea.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::attendeestack1-bucket83908e77-1d84qdfaymy9u",
            "Effect": "Allow"
        },
        {
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::attendeestack1-bucket83908e77-1d84qdfaymy9u/*",
            "Effect": "Allow"
        }
    ]
}
```

You'll also note that, if you look at the trust relationships of the IAM Role in the AWS Console, you'll see that this role can be only be assumed by the **irsa** ServiceAccount in the **security-playground** Namespace within the EKS cluster that has been assigned this particular unique OIDC provider for AWS IAM to integrate with.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::090334159717:oidc-provider/oidc.eks.ap-southeast-2.amazonaws.com/id/25A0C359024FB4B509E838B84988ABB0"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.ap-southeast-2.amazonaws.com/id/25A0C359024FB4B509E838B84988ABB0:aud": "sts.amazonaws.com",
                    "oidc.eks.ap-southeast-2.amazonaws.com/id/25A0C359024FB4B509E838B84988ABB0:sub": "system:serviceaccount:security-playground:irsa"
                }
            }
        }
    ]
}
```

## The Exploit

If we install the AWS CLI into our container at runtime and run some commands we'll see if our Pod has been assigned an IRSA role and they succeed. There is an **02-01-example-curls-bucket-public.sh** file in `/root`. Have a look at it and run it.

```bash
sudo bash; cd ~
cat 02-01-example-curls-bucket-public.sh
./02-01-example-curls-bucket-public.sh
```

The install of the AWS CLI succeeds but the S3 changes fail as we don't have that access. We have an updated manifest for the security-playground Deployment that will use this **irsa** ServiceAccount instead of the **default** one we have been using. 

Apply the IRSA ServiceAccount by running:

```bash
./set-up-irsa.sh
```

Now re-run the script:

```bash
./02-01-example-curls-bucket-public.sh
```

This time they will work!

## The Sysdig Detections

On the host side you'll see many **Drift Detections** which will include the commands being run against AWS - and which we could have blocked rather than just detected with Container Drift. This is a good reason to not include CLIs like the AWS one in your images as well! !["s3drift"]({{site.baseurl}}/assets/images/s3drift.png)

But on the AWS API side (go to Threats -> Cloud Activity) you'll see that the protections against this bucket being made public were removed as well as the new Bucket Policy (making them public) were subsequently applied as well!

!["s3cloudevents"]({{site.baseurl}}/assets/images/s3cloudevents.png)
!["s3cloudevents2"]({{site.baseurl}}/assets/images/s3cloudevents2.png)

{: .note }
> As this is all within one region of one AWS account you'll see that, unlike the Kubernetes events, you'll see the events for the other attendees as well.

## How to Prevent This Attack and Fix This Workload

This IRSA example could have been prevented by being more granular and least-privilege with your IRSA's policy to not use `s3*` and therefore allow the removal of public blocks or applying Bucket Policies (just reading/writing files etc.)

This is where things like [Permission Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) and [Service Control Policies (SCPs)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) can be helpful too in ensuring that Roles don't get created that are this over-privileged.

!["EffectivePermissions-scp-boundary-id"](https://docs.aws.amazon.com/images/IAM/latest/UserGuide/images/EffectivePermissions-scp-boundary-id.png)

As well, enforcing Container Drift with Sysdig so attacker tools and usefull utilities like the AWS CLI aren't able to be downloaded/run at runtime.

## Completed

You have now completed the Exploiting Overpermissioned EKS Workloads section of this module.

{: .value }
> - Many workloads will run in a cloud-based Kubernetes environment and have their own identity and permissions. This solves some issues around the use of API keys and other credentials, but it also introduces new security risks.
> - Sysdig can help you detect and prevent these risks by providing visibility into the permissions and actions that workloads are taking with their workload identities.
> - No matter how well we configure our applications, a malicious actor will eventually find a way to exploit them. Sysdig can help you detect and prevent these exploits by providing visibility into the actions that workloads are taking with their workload identities.
