---
title: Runtime Threat Detection in Kubernetes
parent: Runtime Threat Detection
nav_order: 2
---

{: .goal }
> In this module, we'll be exploring Sysdig's capabilities around detecting **and preventing** runtime threats.
>
> Please note that this is an in-depth, hands-on module. It goes into much detail around not only using Sysdig Secure, but also how to secure Kubernetes workloads.
>
> There are a few sections to this module:
>
> 1. How Sysdig Detects Runtime Threats (the current section)
> 2. Simulating an Attack on a Kubernetes Workload
> 3. Securing the Workload
> 4. Conclusion
>
> Please set aside 1-2 hours to complete this module.

## Runtime Threat Detection and Prevention for Kubernetes Workloads

In this module, we'll be exploring Sysdig's capabilities around detecting and preventing runtime threats.

Regardless of how an attacker gets in, they will do many of the same things. It's a predictable sequence of things best explained by the [MITRE ATT&CK Framework](https://attack.mitre.org/). 

![MITRE ATT&CK]({{site.baseurl}}/assets/images/mitre-attacks.png)

## How Sysdig Detects Runtime Threats

[Sysdig's Threat Research Team](https://sysdig.com/threat-research/) runs a large fleet of honeypots around the world to learn first-hand all the things people do once they get break into a system, and then continually updates our library of rules (possible behaviors to look for) and managed policies (which rules to look for, their severity, and what to do when we find them) on behalf of all of our customers. 

{: .note }
> You can also make your own custom ([Falco](https://falco.org/)) Rules and/or Policies beyond what we offer if you'd like. This is fully transparent and based on opensource tooling/standards rather than a magic black box!

When Sysdig see these rules violated (as defined in the policies) we generate **Events** with all the relevant context in real-time. And we can do so against these sources, with more coming soon such as from other popular cloud/SaaS services:

- Linux kernel System Calls of your Nodes/Containers
- The Kubernetes Audit Trail
- The audit trails of AWS, Azure and GCP
- Okta's audit trail
- GitHub's audit Trail
- MS Entra ID's audit Trail

In addition to our rules-based approach, there are more features that round out our threat detection and prevention capabilities:

- **Container Drift Detection/Prevention** - we can look for any executables that are introduced at runtime that were not in the container image as it was pulled - as well as optionally block them from running
- **Malware Detection/Prevention (Preview)** - we can look for Malware (as defined in several threat feeds we watch) that tries to run - as well as optionally block them from running
- **Crypto Mining ML Detection** - we have introduced our first Machine Learning enabled detection with a model specifically focused on detecting crypto-mining.

## Completed

You have completed this section of the module.

From here you can continue to the [Simulating an Attack]({{site.baseurl}}/docs/modules/runtime-threat-detection/runtime-threat-detection-kubernetes/the-attack.html) section.

[Next Section ➜ Simulating an Attack]({{site.baseurl}}/docs/modules/runtime-threat-detection/runtime-threat-detection-kubernetes/the-attack.html){: .btn }


{: .value }
> Sysdig's runtime threat detection capabilities deliver essential business value by:
> - Using a rules-based approach to detect and prevent common attack patterns in real-time
> - Providing other ways to detect and prevent threats, such as container drift detection and malware detection