---
title: Accessing Your AWS Environment
parent: Getting Started
nav_order: 2
---

{: .goal }
>At the completion of this section, you will have access to your AWS environment and all the content you need to complete the workshop.
> 
> By the end of this section you should have a SMS terminal open to the root user's home directory and be able to run commands to interact with your workshop environment.

1. TOC
{:toc}

## Accessing Your AWS Environment

You'll have received your IAM username and password from the facilitator. This environment consists of:

- An EC2 Instance to serve as a "Jumpbox" or Basion host to connect to the environment
  - You'll connect to this via AWS SSM Session Manager via your web browser and the AWS Console

- A single-Node EKS cluster
  - This has a number of workloads in a number of different Namespaces pre-installed

- An S3 bucket (which you'll be using to exfiltrate some data in the workshop)

!["diagram2"]({{site.baseurl}}/assets/images/diagram2.png)

## Signing Into Your AWS Environment

{: .important }
>Your AWS user is restricted to only be able to access the resources that you need access to in order to complete the workshop. You will see many warnings around lack of permissions, but this is OK as you will still be able to complete the workshop.

Open a web browser and go to <https://aws.amazon.com/console/>

If prompted, choose to sign in with an IAM user (as opposed to the Root user) and enter the AWS Account ID of your workshop region, as provided by the facilitator.

Enter the IAM username and password you were provided and click the **Sign in** button.

Pick the region in the drop-down in the upper right of the console that matches the region you were assigned, as provided by the facilitator.

!["region-us"]({{site.baseurl}}/assets/images/region-us.png)

## Accessing the Jumpbox

Go to the EC2 service's console (you can type EC2 in the Search box on top and then click on the EC2 service in the results).

Click on the **Instances (running)** link under **Resources** to be taken to a list of running EC2 Instances.

!["instances1"]({{site.baseurl}}/assets/images/instances1.png)

In the **Find instance by attribute or tag** search box type **CLASS_ID-kraken-XX** (where CLASS_ID is your class ID and where XX is your attendee number at the end of your username) and press enter/return.

Tick the box next to the jumpbox and then click the **Connect** button on top.

!["instances2"]({{site.baseurl}}/assets/images/instances2.png)

Choose the **Session Manager** tab and then click the **Connect** button.

!["connect"]({{site.baseurl}}/assets/images/connect.png)

{: .important }
> If you close and re-open the Session Manager Session/Terminal window then you'll need to rerun those two commands to return to the root user and its home directory.

If you dont see the **Connect** button enabled you are probably trying to access some other attendee's jumpbox.

## Using the Jumpbox Terminal and Accessing the Scripts and Other Content

Once your terminal window opens type the following commands to return to the root user and its home directory:

```bash
sudo bash
cd ~
```

Now type the following command to see the running pods in your EKS cluster:

```bash
kubectl get pods -A
```

You'll see a list of all the running Pods in your EKS cluster.

## Completed

You have now completed the AWS environment setup section and have access to your AWS environment.

{: .value }
>You have now completed the AWS environment setup section and have access to your AWS environment.
> 
> By accessing the jumpbox with Session Manager, you have access to the root user's home directory in a simple browser-based terminal interface.
>
> All of the workshop scripts and content is pre-installed in the root user's home directory.
