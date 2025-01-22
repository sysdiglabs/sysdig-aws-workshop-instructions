---
title: The Attack
parent: Runtime Threat Detection in Kubernetes  
nav_order: 1
---

{: .goal }
> In this module, we will simulate an attack on a Kubernetes workload, review the types of attacks we make, and how Sysdig detects them.

1. TOC
{:toc}

## Simulating an Attack

In the Sysdig UI hover over **Threats** on the left-hand side and click on Kubernetes under Activity.

Pick the six hour time range (6H) on the bottom to show only the events you are about to generate. This should start out as empty.

Lets's generate some Events!

{: .note}
> This section uses an insecure [Python app](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/main/docker-build-security-playground/app.py) that serves a **very** insecure REST API that will return the contents of any file on the filesystem, write any file to the filesystem and/or execute any file on the filesystem in response to simple **curl** commands.

Go back to the AWS Session Manager terminal browser tab for your workshop jumpbox.

Review the contents of the script that we are going to run:

```bash
sudo bash; cd ~
cat ./01-01-example-curls.sh
```

In it we have some example **curl** commands we are going to run against the insecure security-playground service.

1. Reading the sensitive path `/etc/shadow`
2. Writing a file to `/bin` then `chmod +x`'ing it and running it
3. Installing **nmap** from **apt** and then running a network scan
4. Running the **nsenter** command to 'break out' of our container Linux namespace to the host
5. Running the **crictl** command against the container runtime for the Node (bypassing Kubernetes and the Kubelet to manage it directly)
6. Using the **crictl** command to grab a Kubernetes secret from another Pod on the same Node (that was decrypted to an environment variable there at runtime)
7. Using the **crictl** command to run the Postgres CLI **psql** within another Pod on the same Node to exfiltrate some sensitive data
8. Using the Kubernetes CLI **kubectl** to launch another nefarious workload (leveraging our over-provisioned Kubernetes ServiceAccount that for security-playground)
9. Running a **curl** command against the AWS EC2 Instance Metadata endpoint for the Node from the security-playground Pod
10. Finally run the xmrig crypto miner

Go ahead and execute the attack by running:

```bash
./01-01-example-curls.sh
```

Watch all the output that is returned from the attacker's perspective.

{: .note }
> Note that eventually the Pod is killed a while into the crypto mining triggered by the last curl because the crypto miner (xmrig) tries to use more memory than the limit set for this container (showing another reason it is a good idea to place such limits in your PodSpecs!).

Then go back to the **Sysdig UI** tab and refresh that tab in your browser.

You'll see a heatmap visualisation of which clusters, namespaces and pods the runtime events we've seen are coming from on the left.

And it also gives you either a summary of those events in the **Summary** tab or a full timeline of them in the **Events** tab on the right.

![threats]({{site.baseurl}}/assets/images/threats.png)

Choose the Events tab on the right.

As you can see there are a number of events that Sysdig picked up in real-time!

![threats2]({{site.baseurl}}/assets/images/threats2.png)

If you click into the the top **Detect crypto miners using the Stratum protocol** and then scroll through it you'll see all the context of that event including details of the process, the network, the AWS account, the Kubernetes cluster/namespace/deployment, the host as well as the container.

In particular the process tree view shows us that our Python app (gunicorn) launched a shell that launched the crypto miner xmrig--that looks suspicious!

![processtree]({{site.baseurl}}/assets/images/processtree.png)

You can also click Explore in order to see a more detailed view of this process tree and the history within this environment.

![explore]({{site.baseurl}}/assets/images/explore.png)

Not only does this view show us all the other Events related to this executable (xmrig) on the right, it shows us all the other things that have been happening - the apt-get's, nmap, nsenter's, etc.

![explore2]({{site.baseurl}}/assets/images/explore2.png)

## Understanding These Events

You should scroll down to the oldest/first Event then click into each to reveal all the detail/context of each. 

The things that we picked up here include:

- **Read sensitive file untrusted** - reading the `/etc/shadow` file which a web service shouldn't be doing
- **Drift Detection** - every time an executable was added to the container at runtime (it wasn't in the image) and then it was run
    - It is not best practice to make changes to containers at runtime - rather you should build a new image and redeploy the service in an immutable pattern
- **Launch Package Management Process in Container** - just like with **Drift Detection**, you shouldn't be adding or updating packages in running containers with apt/yum/dnf - but instead do it in your **Dockerfile** as part of the container image build process
- **Suspicious network tool downloaded and launched in container** - it is a common early step for attackers to run a scan to try to work out what network the workload they've exploited is in, and thus, what else they can get to
- **The docker client is executed in a container** - this fires not just on the **docker** CLI but also other container CLIs such as **crictl** and **kubectl**.
    - It is unusual for a container to be trying to talk directly to the container runtime/socket on a Kubernetes cluster - and that you can is actually proof a container escape has happened!
    - Note that if you expand out the Process section it'll show the commands that were run such as that `psql` that was exfiltrating our data

!["psql"]({{site.baseurl}}/assets/images/psql.png)

- **Contact EC2 Instance Metadata Service From Container** - your EKS Pods should be using other means such as [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) to interact with AWS. It going through the Node to use its credentials instead is suspicious
- **Malware Detection** - we look for many malware filenames and hashes from our threat feeds, including crypto miners such as the **xmrig** here
    - We can even block malware from running, as you'll see later on!
- **DNS Lookup for Miner Pool Domain Detected** - we look at network traffic (at Layer 3) and when the destination are suspicious things like crypto miner pools or [Tor](https://www.torproject.org/) entry nodes

And this is only a small sample of the Rules we have out-of-the-box as part of the service!

{: .highlight }
> **Optional:** Feel free to copy `01-01-example-curls.sh` to a new file and play with generating your own curls if you want to see whether Sysdig will pick up various other things you may want to try!
>
>**Optional:** Have a look at all our Managed Policies (go to **Policies** on the left and then **Runtime Policies**) as well as our Rules Library (go to **Policies** then expand out the **Rules** carrot menu and choose **Rules Library**). Drill down into the Falco YAML (noting that this is not a "magic black box" and you can write your own Rules and Policies). Focus on the Policies and Rules that you saw fire in our example.

## Why Did This Attack Work?

In order for this attack to succeed many things had to be true.

- **Our service was vulnerable to remote code execution** - this could be either due to our own code being vulnerable (as was the case here) or an opensource package our app uses (from pip, npm, maven, nuget, etc.) being vulnerable

- **Our service that we were was running as root** - so, not only could it read/write everything within the container's filesystem, but it was also root when it escaped out of the container to the host!

The PodSpec had [**hostPID: true**](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/3da34f8429bd26b82a3ee2f052d2b654d308990f/k8s-manifests/04-security-playground-deployment.yaml#L18) as well as [privileged **securityContext**](https://github.com/sysdiglabs/kraken-hunter-example-scenarios/blob/3da34f8429bd26b82a3ee2f052d2b654d308990f/k8s-manifests/04-security-playground-deployment.yaml#L35) which allowed it to escape its container boundary (the Linux namespace it was being run in) to the host and then control that hosts's container runtime (bypassing Kubernetes and the [kubelet](https://kubernetes.io/docs/concepts/overview/components/#kubelet)). That in turn lets it control all the other containers that happened to be running on that Node. 

!["diagram1"]({{site.baseurl}}/assets/images/diagram1.png)

- **The `nsenter` command lets us switch Linux namespaces** - which containers use to isolate us from the other containers. We can only successfully run this if we are root, have hostPID as well as a privileged security context.

- **The `crictl` command is like the Docker CLI but for containerd** (which is the container runtime used these days by Kubernetes Nodes). We can only successfully run this if we are root as well as on the host (such as breaking out with nsenter).

- **The attacker was able to add new executables like `nmap` and the crypto miner `xmrig` to the container at runtime and run them.**

- **The attacker was able to download those things from the Internet** (because this Pod was able to reach everywhere on the Internet via its egress).

- **The ServiceAccount for our service was over-provisioned and could call the K8s API to do things like launch other workloads** (which it didn't need).

Run:

```bash
kubectl get rolebindings -o yaml -n security-playground && kubectl get roles -o yaml -n security-playground
```

to see that the default ServiceAccount has a Role bound to it with it with the following rules/permissions:

```yaml
rules:
- apiGroups:
    - '*'
    resources:
    - '*'
    verbs:
    - '*'
```

At least it was a Role rather than a ClusterRole - meaning it can only do things with this **security-playground** Namespace. But there is plenty of damage you can do with just full admin within a Namespace!

- **The attacker was able to reach the EC2 Metadata endpoint (`169.254.0.0/16`)**, which is intended just for the EKS Node, from within the Pod.

## Completed

You've now seen how Sysdig Secure can detect and prevent runtime threats and completed this section.

Please continue to the [Securing the Workload]({{site.baseurl}}/docs/modules/runtime-threat-detection/runtime-threat-detection-kubernetes/securing-the-workload.html) section.

[Next Section ➜ Securing the Workload]({{site.baseurl}}/docs/modules/runtime-threat-detection/runtime-threat-detection-kubernetes/securing-the-workload.html){: .btn }

{: .value }
> At some point, every organisation will experience a breach. While we can do our best to prevent attacks from succeeding, and Sysdig Secure can help achieve compliance to enable this prevention, we also need to be able to detect and respond to breaches quickly.
>
> Sysdig Secure's runtime threat detection enables real-time detection of attacks and the ability to take action.
