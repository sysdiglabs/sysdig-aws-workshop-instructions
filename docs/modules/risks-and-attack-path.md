---
title: Risks and Attack Path
parent: Modules
nav_order: 4
---

{: .goal}
> At the end of this module, you will be able to use the Sysdig interface to understand the biggest risks in your environment.

1. TOC
{:toc}

## Risks and Attack Path

Sysdig is a comprehensive Cloud Native Application Protection Platform (CNAPP), which means that we bring all of the metadata and information our platform instruments into one single place, where we can anlayze your entire environment for security posture, risk, and threat detection events.

Where we do that is in the part of the product called **Risks**.

{: .sysdig-gui }
> Go to **Risks** in the left sidebar.

![risks1]({{site.baseurl}}/assets/images/risks1.png)


{: .sysdig-gui }
> **Expand out the carrot of a top-level risk** to see more details. 

The fact that we see the "Live" icon warning shows this is an active risk. Not only does it have insecure configurations and critical vulnerabilities but we also see recent, live, critical threat detection events that may be getting exploited right now!

As well, you see that this risk includes other findings:

* It is exposed
* It has critical vulnerabilities
* It has insecure configurations
* And it has threat detection events where risky behavior has already been detected

{: .sysdig-gui }
> **Click on the risk** to see a smaller version of the attack path visualisation. 

![risks2]({{site.baseurl}}/assets/images/risks2.png)

{: .sysdig-gui }
> **Click Explore in the upper right** to see a bigger version of the attack path visualisation.

!["risks2"]({{site.baseurl}}/assets/images/risks2.png)

Here you can see all of the data Sysdig has about the security-playground workload but all brought together in one visualisation. And that, while any of these things are bad, the fact that this workload has all of them makes it a Critical Risk to prioritise.

Once we are in the larger Attack Path visualisation we can click on any of the icons to drill down and go deeper into that, and perhaps even resolve it right from this UI.

{: .sysdig-gui}
> Click on any of the **icons**, such as a CVE, or event, to go deeper and investigate the issue.   

![risks3]({{site.baseurl}}/assets/images/risks3.png)

![risks4]({{site.baseurl}}/assets/images/risks4.png)

## Completed

You have now completed the Risks and Attack Path module.

{: .value}
> You have now used Sysdig Secure to understand the biggest risks in your environment.
>
> * With this information, you can get the best return on your investment in terms of what to fix first.
>
> * Instead of being inundated with thousands of vulnerabilities and misconfigurations and not being sure where to start, you can now focus on the most critical risks first, and then move on to the less critical ones.
>
> * As well, you are able to investigate the attack via the Sysdig Attack Path, accessing important information about the attack and the assets that were impacted from a single view.
