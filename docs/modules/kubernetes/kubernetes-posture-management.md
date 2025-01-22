---
title: Kubernetes Posture Management
parent: Kubernetes
nav_order: 1
---

{: .goal}
> By the end of this module, you will be able to use the Sysdig UI to understand the posture of your Kubernetes/EKS clusters and workloads, and how to fix any misconfigurations.

1. TOC
{:toc}

## Kubernetes Posture and Compliance

It's important that your Kubernetes/EKS clusters and the workloads on them are properly configured. This is referred to as either Posture or Compliance, and it is utlimately about whether your are compliant with various standards and benchmarks...or not.

Sysdig can ensure you are compliant with many common standards such as CIS, NIST, SOC 2, PCI DSS, ISO 27001, and many more. To see the whole current list you can go to **Policies** on the left then **Policies** again under the **Posture** heading.

The Center for Internet Security (CIS) publishes a security benchmark for many common resources, including EKS. 

{: .note }
> Learn more at <https://www.cisecurity.org/benchmark/kubernetes>.

We'll be looking at your cluster and its workloads to see if they are compliant with that standard in this module.

First, go to the Sysdig tab in your browser.

Hover over **Compliance** on the left navigation pane and then click **Overview**.



Click on the **CIS Amazon Elastic Kubernetes Service Benchmark** under your heading.

{: .note }
> We have used our [Team and Zone-based authorization](https://docs.sysdig.com/en/docs/sysdig-secure/policies/zones/) so that your Team can only see your own cluster/Zone.
>
> The CIS EKS Benchmark is the only compliance standard we've set against your Zone here, but we have many others such as NIST, SOC2, PCI-DSS, etc.

![posture1]({{site.baseurl}}/assets/images/posture1.png)

There are some controls here that would have prevented our attack. 

If you click into the **Show Results** link for each you'll see the list of failing resources then you can click **View Remediation** next to the **security-playground** Resource to see the Remediation instructions:

- 4.2.1 Minimize the admission of privileged containers
    - Container running as privileged
- 4.1.5 Ensure that the default service accounts are not actively used
    - Access granted to "default" account directly

![posture2]({{site.baseurl}}/assets/images/posture2.png)
![posture3]({{site.baseurl}}/assets/images/posture3.png)

If these settings for **security-playground** were configured to be passing CIS' EKS Benchmark, then it would be just like the **security-playground-unprivileged** workload which, as we saw, fared **much** better in our attack.

As well as helping you to remediate any security issues with your workload(s) and cluster(s), this tool will help you to prove to your auditors that they are compliant with any standards you need to adhere to as well.

There is another view of the same data which may prove more useful in many situations - **Inventory**.

This is the same information but from the perspective of the resource rather than from the compliance standard, meaning that the Compliance view is "show me what is passing or failing the standard" whereas the Inventory view is *"show me how my resource is doing against the standard(s) applied to it (by the Zone)"*.

Here we are looking at the security-playground deployment and seeing how it is doing first for its posture.

!["inventory1"]({{site.baseurl}}/assets/images/inventory1.png)

You can even click through to the same remediation steps right in this view too.

Hover your mouse over the control to see View Remediation.

!["inventory2"]({{site.baseurl}}/assets/images/inventory2.png)

!["inventory3"]({{site.baseurl}}/assets/images/inventory3.png)

Finally, one of the common things we are is *"How can I see what workloads have a particular CVE?"*. 

This filter is not possible in the Vulnerability section as those filters are more about the workloads than the vulnerabilities, but it is possible here in Inventory. 

Put in a filter for **Vulnerability in CVE-2023-45853** as an example.

!["inventory4"]({{site.baseurl}}/assets/images/inventory4.png)

{: .note }
> As a reminder, all workshop participants are in one Sysdig account but are only seeing their own clusters/workloads. So this is something we can easily restrict via our built-in Authorization (via Zones tied to Teams) so that people will only see as much or as little of the environment in Sysdig as you'd like.

## Shifting Left by Scanning Your Infrastructure Code

It is also possible to use the same Sysdig CLI scanner we used to scan for container image vulnerabilities to also scan your Infrastructure as Code (by adding the `--iac` flag) to ensure that is secure before deploying it.

In order to do so you can run the following command:

```bash
sudo bash; cd ~
~/sysdig-cli-scanner --apiurl $SYSDIG_SECURE_URL --iac example-scenarios/k8s-manifests/04-security-playground-deployment.yaml
```

You could add this as a stage of a pipeline or as a git merge test where, if the scan failed, it would stop the pipeline/merge until the security misconfigurations were resolved.

Setting up such pipeline scans/gates is often referred to as "shifting left" (further into the development stages/lifecycle) or "DevSecOps".

## Completed

You have now completed the Kubernetes Posture Management module.

{: .value}
> - You have now used Sysdig Secure to understand the posture of your Kubernetes/EKS clusters and workloads.
> - You have seen how we can use the Sysdig UI to understand the posture of your Kubernetes/EKS clusters and workloads.
> - You have seen how we can use the Sysdig UI to fix any misconfigurations.
> - You could now more easily show your auditors that you are compliant with various standards/benchmarks.
