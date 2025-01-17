# Sysdig Hands-on EKS Security Workshop

Welcome to Sysdig's hands-on workshop. In this workshop, you'll experience some of the security challenges of Kubernetes/EKS first-hand - and how Sysdig can help.

We have provisioned a separate EKS cluster and EC2 instance (to serve as a jumpbox/bastion) for each of you. You'll connect to that jumpbox via AWS SSM Session Manager in your browser - and it is preloaded with all the tools that you'll need to interact with your EKS cluster and work through today's labs.

We have also provisioned a user for you within Sysdig Secure. While this Sysdig SaaS tenancy is shared between everyone in the workshop today, your login is tied to a team within it which, in turn, is filtered (via a Zone) to only show you information about your EKS cluster/environment.

## Getting Started

1. [Logging into your environment](modules/logging-in-to-your-environment.md)

## Modules

1. [Runtime Threat Detection and Prevention (Workload/Kubernetes)](modules/runtime-threat-detection-workload.md)
2. [Runtime Threat Detection and Prevention (Cloud/AWS)](modules/runtime-threat-detection-cloud.md)
3. [Host and Container Vulnerability Management](modules/vulnerability-management.md)
4. [Kubernetes Posture/Compliance](modules/kubernetes-posture-management.md)
5. [Risks and Attack Path](modules/risks-and-attack-path.md)
6. [Kubernetes native firewall (NetworkPolicies)](modules/kubernetes-network-policies.md)

## Conclusion

This was just a brief introduction of some of the many capabilities that Sysdig offers customers to help with securing your Kubernetes environments, including AWS EKS, as-a-service.

We'd love to show you more about what Sysdig can do for you in a free trial in your own environment. Reach out to your facilitator for details.

Thank you for coming!
