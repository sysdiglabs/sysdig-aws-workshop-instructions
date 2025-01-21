---
title: Securing the Workload
parent: Runtime Threat Detection in Kubernetes
nav_order: 2
---

{: .goal }
> This section is about how to secure the workload.
>
> As well, we will re-run the attack we performed in the previous section, but now with various prevention strategies in place.

1. TOC
{:toc}

## How to Secure This Workload

For each of the security issues we have covered so far in this module, there are solutions.

- **Fix vulnerabilities:** To fix the vulnerabilities in our case here, we can use a Static application security testing (SAST) product to identify our insecure code. Our partners like [Snyk](https://snyk.io/product/snyk-code/) and [Checkmarx](https://checkmarx.com/cxsast-source-code-scanning/) can help.

!["Snyk-SAST"]({{site.baseurl}}/assets/images/Snyk-SAST.png)

Alternatively, if this was based on a known/public CVE within the app/container (such as Log4J etc.) instead, Sysdig's Vulnerability Management (which we'll explore in a future Module) would have detected it and let us know to patch either the base layer of our container or the code package to an updated version without the vulnerability

- **Run as non-root:** To run this container as non-root we actually need to change the Dockerfile in the following ways. Here is the [Dockerfile](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/docker-build-security-playground/Dockerfile) before these changes - and [here](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged) it is after.

- **Add a user and group to use:** We need to [add a user and group to use](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged#L3) as part of the docker build

- **Specify in the Dockerfile to run as that User by default:** We need to [specify in the Dockerfile to run as that User by default](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged#L8) (note this is just the default and can be overridden at runtime - unless a restricted PSA or other admission controller blocks that)

- **Location of app:** We need to put our app in a folder that user/group has permissions to read and execute (and perhaps write to as well) - [in this case we use our new user's home directory](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged#L9) as opposed to the original /app

There was a great talk about building least privilege containers from the recent KubeCon Europe that goes much deeper here - <https://youtu.be/uouH9fsWVIE>

- **Remove the insecure options from our PodSpec:** We just need to remove the insecure options from our PodSpec. But we also need to, ideally, prevent people from putting those sorts of options in them as well.

There is now a feature built-in to Kubernetes (which GAed in 1.25) to enforce that they don't - [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/).

This works by [adding labels onto each Namespace](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/). There are two standards that it can warn about and/or enforce for you - baseline and restricted.

- [baseline](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline) - this prevents the worst of the parameters in the PodSpec such as hostPid and Privileged but still allows the container to run as root
- [restricted](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted) - this goes further and blocks all insecure options including running as non-root

And Sysdig has a Posture/Compliance feature that can help both catch the IaC before it is deployed as well as remediate any issues at runtime, which we'll look at in a future Module.

1. We can block the execution of any new scripts/binaries added at runtime with Container Drift (in this case we only had it detecting not preventing Drift)
1. We can limit the egress access of Pod(s) to the Internet via either Kubernetes NetworkPolicy (which we cover in a future Module) or by making each thing go through an explicit authenticated proxy to reach the Internet with an allow-list of what that service is able to reach etc.
1. We can remove the Role and RoleBinding to the Kubernetes API by our default ServiceAccount that lets it have unnecessary access to to the Kubernetes API.
1. We can either block egress access for the Pod to `169.254.0.0/16` via NetworkPolicy as described above and/or ensure a maximum of 1 hop with IDMSv2 as AWS describes in their [documentation](https://docs.aws.amazon.com/whitepapers/latest/security-practices-multi-tenant-saas-applications-eks/restrict-the-use-of-host-networking-and-block-access-to-instance-metadata-service.html)

## Seeing the Fixes in Action

We have an example workload where some of the security issues have been fixed.

It was built with our new non-root Dockerfile and it is running in the security-playground-restricted Namespace where a PSA is enforcing a restricted security standard, meaning it can't run as root or have the options such as hostPID or privileged SecurityContext to allow for container escapes.

You can see the labels on this namespace implementing the PSA by running:

```bash
kubectl describe namespace security-playground-restricted
```

Note the **pod-security** Labels.

{: .note }
> You can see the original Kubernetes PodSpec [here](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/k8s-manifests/04-security-playground-deployment.yaml) and the updated one with all the required changes to pass the restricted PSA [here](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/k8s-manifests/07-security-playground-restricted-deployment.yaml).

To see how our attack fares with 1-3 fixed run (it is the same as the last file just pointed at the different port/service for security-playground-restricted)>

 ```bash
 ./01-02-example-curls-restricted.sh
 ```

You'll note:

- Anything that required root within the container (reading `/etc/shadow`, writing to `/bin`, installing packages from apt, etc.) fails with a **500 Internal Server Error** because our Python app didn't have permissions to do it.
- Without **root**, **hostPid** and **privileged** it couldn't escape the container
- The only things that worked were hitting the Node's EC2 Metadata endpoint and downloading/running the xmrig crypto miner into the user's home directory (where it still had rights to do so.)

If we also add in Sysdig enforcing that any Container Drift is prevented (that no new executables added at runtime can be run), then that blocks *everything* but the EC2 Instance Metadata access (which we'll block with NetworkPolicies in a future Module). To see that:

Go to **Policies** -> **Runtime Policies** and then look at **security-playground-restricted-nodrift**.

Note that rather than just detecting drift (as in the other Namespaces) we are blocking it if the workload is in the **security-playground-restricted-nodrift** Namespace.

And we have a another copy of our security-playground-restricted service running there on a different HostPort.

Run:

```bash
./01-03-example-curls-restricted-nodrift.sh 
```

This runs all those same curls but against a workload that is both restricted like the last example but also has Sysdig preventing Container Drift (rather than just detecting it).

If you look at the resulting Events in our Threats UI you'll see the Drift was **prevented** rather than just detected this time.

!["driftprevented"]({{site.baseurl}}/assets/images/driftprevented.png)

And, we also can now block instead of just detecting Malware.

Go to **Policies** -> **Runtime Policies** and then look at **security-playground-restricted-nomalware**.

Note that rather than just detecting malware (as in the other Namespaces) we are blocking it if the workload is in the **security-playground-restricted-nomalware** namespace.

And we have a another copy of our security-playground-restricted service running there on a different HostPort.

Run:

```bash
./01-04-example-curls-restricted-nomalware.sh
```

This runs all those same curls but against a workload that is both restricted but also has Sysdig preventing malware,rather than just detecting it. But not blocking Container Drift - as we want to show that the malware tries to run so we can block it with that.

If you look at the resulting Events in our Threats UI you'll see the Malware was **prevented** from running rather than just detected this time.

!["malware"]({{site.baseurl}}/assets/images/malware.png)

As you can see, a combination of fixing the posture of the workload as well as Sysdig's Container Drift and Malware Detection goes a **long** way to preventing so many common attacks, even against workload with such critical vulnerabilities!

One last thing you can try is to test trying to change security-playground-restricted to undermine its security like security-playground. Run the following command to try to deploy the insecure container image and PodSpec to that namespace:

```bash
kubectl apply -f 01-cfg-security-playground-test.yaml
```

Note how we're warned that is not allowed in the **security-playground-restricted** Namespace due to the restricted PSA in place there. Even though it let the Deployment create - you'll note that it (actually its ReplicaSet) is unable to actually launch the Pods.
!["psa"]({{site.baseurl}}/assets/images/psa.png)

Run:

```bash
kubectl events security-playground -n security-playground-restricted
```

to see the Pod creation failures.

This is why blocking at runtime with PSAs are a bit of a blunt instrument. You should also let people know earlier in the pipeline that this is going to happen (and they need to fix the PodSpecs) rather than have them scratch their head on why their pods are not launching at run/deploy time.

## Completed

You have now completed the Securing the Workload section of this module.

Please continue to the [Conclusion]({{site.baseurl}}/docs/modules/runtime-threat-detection/runtime-threat-detection-kubernetes/conclusion.html) section.

[Next Section ➜ Conclusion]({{site.baseurl}}/docs/modules/runtime-threat-detection/runtime-threat-detection-kubernetes/conclusion.html){: .btn }

{: .value }
> In this section we have given you some ideas as to how to secure Kubernetes workloads. Kubernetes is initially a blank slate, and you need to apply security policies to it in order to make it safe to use.
>
> However, a prevention-only strategy is not enough. We need to detect and respond to threats as well.
