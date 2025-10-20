
[Open in app](https://rsci.app.link/?%24canonical_url=https%3A%2F%2Fmedium.com%2Fp%2Ff3eeca9e95f1&%7Efeature=LoOpenInAppButton&%7Echannel=ShowPostUnderUser&source=---top_nav_layout_nav----------------------------------)

Sign up

[Sign in](https://medium.com/m/signin?operation=login&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fdeploy-kubernetes-cluster-on-aws-ec2-instances-f3eeca9e95f1&source=post_page---top_nav_layout_nav-----------------------global_nav-----------)

[](https://medium.com/?source=---top_nav_layout_nav----------------------------------)

[

Write



](https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2Fnew-story&source=---top_nav_layout_nav-----------------------new_post_topnav-----------)

![](https://miro.medium.com/v2/resize:fill:64:64/1*dmbNkD5D-u45r44go_cf0g.png)

# Deploy Kubernetes cluster on AWS EC2 instances

## Bootstrap single control plane cluster on EC2 instance using kubeadm. Latest version with fix for ‘pod sandbox changed’ error.

[

![Mahesh](https://miro.medium.com/v2/resize:fill:88:88/1*wvFXUaXI0WWsbpuyT2iTEQ.jpeg)





](https://mrmaheshrajput.medium.com/?source=post_page---byline--f3eeca9e95f1--------------------------------)

[Mahesh](https://mrmaheshrajput.medium.com/?source=post_page---byline--f3eeca9e95f1--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2Ff529a0088953&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fdeploy-kubernetes-cluster-on-aws-ec2-instances-f3eeca9e95f1&user=Mahesh&userId=f529a0088953&source=post_page-f529a0088953--byline--f3eeca9e95f1---------------------post_header-----------)

23 min read

·

Dec 22, 2023

2

7

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Ff3eeca9e95f1&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fdeploy-kubernetes-cluster-on-aws-ec2-instances-f3eeca9e95f1&source=---header_actions--f3eeca9e95f1---------------------bookmark_footer-----------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2Fplans%3Fdimension%3Dpost_audio_button%26postId%3Df3eeca9e95f1&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fdeploy-kubernetes-cluster-on-aws-ec2-instances-f3eeca9e95f1&source=---header_actions--f3eeca9e95f1---------------------post_audio_button-----------)

![](https://miro.medium.com/v2/resize:fit:1400/0*oT8d8HoEvAwF929K)

Photo by [Antônia Felipe](https://unsplash.com/@a_mfelipe?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

Kubernetes, often abbreviated as K8s, has emerged as a pivotal technology in the realm of container orchestration, revolutionizing the deployment and management of applications at scale.

![](https://miro.medium.com/v2/resize:fit:1400/1*zTvudIggjSAp5rNN_aGASQ.jpeg)

Shared DNA

Born out of Google’s internal container orchestration system, Borg, Kubernetes was open-sourced in 2014, quickly gaining widespread adoption for its robust and flexible architecture.

In the era of microservices, where applications are decomposed into smaller, independently deployable units, Kubernetes serves as a powerful orchestrator, providing automated scaling, load balancing, and seamless deployment across distributed environments.

This article explores the steps to deploy a Kubernetes cluster on AWS EC2 instances, unlocking the potential of containerized applications in the dynamic landscape of modern software development.

## Why Bootstrap your own K8s Cluster

Bootstrapping your own test Kubernetes cluster on AWS EC2 instances offers a valuable and cost-effective approach for testing microservices, especially when compared to fully-managed services like AWS EKS or Google Cloud Hosted Kubernetes. While managed services provide simplicity and ease of use, there are compelling reasons to consider a DIY approach for testing purposes.

1. **Customization and Control**:  
    Setting up your Kubernetes cluster allows for greater customization and control over the cluster configuration. This level of control is essential when fine-tuning the environment to match specific testing requirements or when integrating with other tools and services.
2. **Cost Efficiency**:  
    Building and managing your own cluster on EC2 instances can be more cost-effective for testing purposes. Managed Kubernetes services typically come with additional costs (AWS EKS costs $0.20 per hour), and for non-production environments or small-scale testing, these expenses might outweigh the benefits of convenience.
3. **Learning Experience**:  
    Creating your Kubernetes cluster is an excellent learning opportunity for developers and administrators. It provides hands-on experience with the intricacies of Kubernetes architecture, deployment strategies, and troubleshooting, fostering a deeper understanding of the technology.
4. **Offline Testing**:  
    Bootstrapping your cluster allows for testing in offline or air-gapped environments where direct access to managed services may not be feasible. This is crucial for organizations with stringent security and compliance requirements.
5. **No Vendor Lock-In for Testing**:  
    Building your cluster minimizes the risk of vendor lock-in during the testing phase. As your microservices evolve, having the freedom to switch cloud providers or Kubernetes distributions becomes easier without being tied to a specific managed service.

Despite the advantages, bootstrapping your own test Kubernetes cluster on AWS EC2 has its drawbacks like **Complexity, Operational Overhead, Potential for Errors, Security Concerns and Man-power skills**.

Therefore take it only as one more tool in your arsenal.

**Responsible language** — The text in this article follows guidelines issued by the [Inclusive Naming Initiative](https://inclusivenaming.org/) which promotes responsible language and tries to avoid harmful terms. As an example, the Kubernetes project has replaced the potentially harmful term “master” with “control plane node”. This articles does the same.

# Overview

![](https://miro.medium.com/v2/resize:fit:1400/1*gOLTPinbP-Dfvx7lQssr-Q.png)

Components installation overview

6. First, we will create two security groups in the AWS VPC, adding the necessary ports for the control plane and worker nodes.
7. Subsequently, we will launch three instances: one for the control plane and two instances as worker nodes. All worker nodes will run Ubuntu 20.04 x84 on `t2.large` and control plane on `t2.medium` featuring 4GB RAM and 2 vCPUs. Although the [official kubeadm installation instructions](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin)recommend a Linux Debian or Red Hat-based distribution with at least 2GB RAM and 2 CPUs, we have opted for Ubuntu for our instances.
8. After launching the instances, we will SSH into the control plane node to disable swap and install the container runtime, kubeadm, kubelet, and kubectl.
9. Following this, we will initialize the cluster using the `kubeadm init`command.
10. To establish pod networking, we will install the [Weave Net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/) Container Network Interface (CNI).
11. Additionally, we’ll address the “Pod sandbox changed; it will be killed and re-created” error associated with the `1.29.0-00` version, ensuring a smoother operation. Although it’s unclear whether this issue occurred in previous versions, it certainly did not manifest until `v1.24.0-00`.
12. To enhance the cluster’s functionality, we will deploy an Ingress service and subsequently deploy nginx application.
13. This application will be invoked via NodePort service on different paths, providing a comprehensive test of the cluster’s capabilities.

# Create Security groups

When running Kubernetes in an environment with strict network boundaries, such as on-premises datacenter with physical network firewalls or Virtual Networks in Public Cloud, Kubernetes uses specific ports and protocols.

We will create 2 security groups and add rules for control plane and worker nodes.

[Control Plane](https://kubernetes.io/docs/reference/networking/ports-and-protocols/#control-plane)

![](https://miro.medium.com/v2/resize:fit:1400/1*iYHOW84BYKMhvKbQV-Gq0Q.png)

Source: [https://kubernetes.io/docs/reference/networking/ports-and-protocols/#control-plane](https://kubernetes.io/docs/reference/networking/ports-and-protocols/#control-plane)

[Worker Nodes](https://kubernetes.io/docs/reference/networking/ports-and-protocols/#node)

![](https://miro.medium.com/v2/resize:fit:1400/1*qXPQ2p9tU_nRsLnZtEWcNg.png)

Source: [https://kubernetes.io/docs/reference/networking/ports-and-protocols/#node](https://kubernetes.io/docs/reference/networking/ports-and-protocols/#node)

In AWS Virtual Private Cloud (VPC), security groups act as virtual firewalls for EC2 instances, controlling inbound and outbound traffic. They function at the instance level, specifying allowed traffic based on rules. Security groups enhance network security within the VPC by permitting or denying traffic based on defined rules for communication between instances and external networks.

14. Login to your AWS account and search Security groups

![](https://miro.medium.com/v2/resize:fit:1400/1*823B61Ki1_T0ND5qSE7wSQ.png)

15. Click on Security groups service and then on the orange “Create security group” button.

16. First we are creating security group for control plane, so under Basic details section enter “control-plane-sg” name for easier identification at the time of instance creation.

![](https://miro.medium.com/v2/resize:fit:1400/1*yMUyHC9jePo91N-iB27FEw.png)

17. Under Inbound rules we will add three rules based on official kubernetes ports and protocols documentation:

- SSH - port 22 so that we can ssh from our local machine to control plane
- TCP - port 6443 for API server that can be access by anyone
- TCP -port range 2379–2380 for etcd database server client API on the private network only
- TCP -port range 10250–10259 for kubelet, kube-scheduler, and kube-controller, also on private network only

By private network, I mean our source will be the VPC CIDR block. It can be found by going to the VPC service from search menu and scrolling horizontally until you see IPv4 CIDR.

![](https://miro.medium.com/v2/resize:fit:1400/1*qVmwxyb4Bff1dq0w9TghWg.png)

VPC CIDR

Kubernetes also supports dual stack networking with the options of `PreferDualStack` and `RequireDualStack` . But in order to create IPv6 cluster you must have/create a VPC with IPv6 CIDR block enabled.

After you have added all four rules in the security group, it must resemble the below screenshot:

![](https://miro.medium.com/v2/resize:fit:1400/1*Jg8RGzju-txYa6LG-RusDw.png)

**_Note: The source for rules with port range 10250–10259 and rule 2379–2380 is the our VPC IPv4 CIDR block. Port 22 and 6443 rules have destination “Anywhere-IPv4”._**

18. Add an outbound rule of all traffic (destination “Anywhere-IPv4”)

![](https://miro.medium.com/v2/resize:fit:1400/1*O8kuSK-TPKEukaivwqC0mA.png)

19. Click on “Create security group” and then create a new security group for worker nodes.

20. Give a suitable name like “workers-sg”, and choose your VPC.

21. Add three Inbound rules

- SSH — port 22 so that we can ssh from our local machine to all worker nodes
- TCP -port 10250 with your VPC CIDR block
- TCP -port range 30000–32767 with the source as anywhere IPv4.

![](https://miro.medium.com/v2/resize:fit:1400/1*vBneH8eQvnaeg5Z_roSqfA.png)

![](https://miro.medium.com/v2/resize:fit:1400/1*DazSu2ZWSCYCEhZ8hYTYgA.png)

If you are not familiar with specific ports and protocols, consider creating a single security group. Set both Inbound and Outbound traffic sources to ‘Anywhere-IPv4,’ and use this same security group for the control plane and worker nodes.

# Launch EC2 instances

Search ec2 in search bar and click on EC2 service.

![](https://miro.medium.com/v2/resize:fit:1400/1*uOmTEcMeja8tZRyPe5E1Bw.png)

Click on ‘Instances’ from left hand side vertical menu and then on the “Launch Instances” orange button.

![](https://miro.medium.com/v2/resize:fit:1048/1*Y6NOc0DR-VtcEBsMtEmIYg.png)

## Control Plane Node

We want an Ubuntu OS running on `t2.medium` with our pre-created security group for control plane and we will give this instance 12 GiB of storage.

22. Give an optional suitable name (tag) to the instance and select “Ubuntu” from OS panel. At the time of writing this, AWS auto selected Ubuntu Server 22.04 LTS image, and we will proceed with that.

![](https://miro.medium.com/v2/resize:fit:1400/1*Un_iX1q6a8xb0yTKbhsxpg.png)

23. Under instance type section select `t2.medium` that has 2 vCPU and 4 GiB Memory. Create a new key pair, or choose from one of your existing pairs.

_Note: An on-demand_ `_t2.medium_` _instance costs $0.0464 per hour._

![](https://miro.medium.com/v2/resize:fit:1400/1*PdygTkxl3MHWkguSnwSXRQ.png)

24. Under network setting panel, if you have only 1 VPC then it will be selected by default else change it to your desired VPC by clicking on “Edit” button.

In the same panel, under Firewall (security groups) setting, click on “Select existing security group” radio option and below that select the security group that we created for control plane. If you created only 1 security group with wildcard then please select that.

![](https://miro.medium.com/v2/resize:fit:1400/1*mumtnETaUNBWHhUipNRS2w.png)

25. Under configure storage panel, I selected 12 GiB of gp2 storage. I did that because our 2 more worker nodes will have at 8 + 8 GiB, making our aggregated cluster memory consumption less than 30 GiB. You can go higher and lower without any problems for this demo.

![](https://miro.medium.com/v2/resize:fit:1400/1*FByuPqWTh1TL1HtVsgNGGw.png)

After verifying all details, click on “Launch instance” orange button.

## Worker Nodes

26. For worker nodes, don’t give a name instead change ‘Number of instances’ to 2 in Summary section.

![](https://miro.medium.com/v2/resize:fit:1400/1*x4atXL3rKu82xoj5EsmBNQ.png)

OS will be “Ubuntu”, same as what we selected for control plane node.

27. In instance type panel, I selected `t2.large` just for demonstration. However depending on your app(s) `t2.micro` should also be able to hold well.

You can check the memory load on any node with `kubectl describe node -n NODE_NAME` command.

28. Select one of your existing key pair to be able to ssh from your local machine. You can also skip this and use EC2 Connect to ssh using your web browser.

29. Under network settings panel, click on “Select existing security group” just like we did while launching instance for control plane. Select the security group created for worker node or choose the sg that you created for all traffic.

![](https://miro.medium.com/v2/resize:fit:1400/1*qwKvAzWoVcH8oHnANuZo3w.png)

30. Under configure storage panel I chose 8 GiB of storage, you can go higher as you deem fit.

Whenever you are ready click on “Launch instance” orange button.

Now go back to Instances list page and rename worker nodes for easier identification.

![](https://miro.medium.com/v2/resize:fit:1400/1*nvc-cxSOWEj1WB3gzNN8LA.png)

_Note: these names are just tags and they do not reflect the actual host names that are used by Kubernetes. We will give host names in the next section._

# Installing CRI, kubeadm, kubelet and kubectl

## SSH into instances

31. You can move your ec2 key pair to `~/.ssh/KEY_NAME.pem` or `cd` into the directory that has that key.
32. If you are using this key for the first time then it will have read and write permission so we will have to change permissions before using it. Execute the following command:

chmod 400 KEY_DIR/KEY_NAME.pem

Your key permission must reflect like below:

-r--------

33. Copy the public IP of each of the three servers and replace them in the following command along with your key:

ssh -i KEY_NAME.pem ubuntu@PUBLIC_IP

![](https://miro.medium.com/v2/resize:fit:1400/1*IoUt5h-9Dp02T_a9tzyUgw.png)

![](https://miro.medium.com/v2/resize:fit:1400/1*TWC8S2ZuX8zl8bn3zl8K9g.png)

SSH into control plane

In new terminal tabs you can ssh similarly into both worker nodes.

34. At this time it is difficult to identify the nodes with IP address in the terminal, so we will give them host names but first we must turn swap off. Swap off is required by Kubernetes.

Run the following command in all three nodes:

sudo swapoff -a

35. [Optional] Now we will add ip to host name mapping in the hosts file of all three nodes.

Edit the `/etc/hosts` file and add the public ip and a suitable host name at the end of the file in all three nodes.

![](https://miro.medium.com/v2/resize:fit:1400/1*7nYxZjvVKazegFHHvNp33g.png)

You must make sure that the last three lines in `/etc/hosts` file are same on every node.

![](https://miro.medium.com/v2/resize:fit:1400/1*LKyF83hfDWx-IEyaD2Qreg.png)

**Why not give master name instead of control-plane?**

**Responsible language** — The text in this article follows guidelines issued by the [Inclusive Naming Initiative](https://inclusivenaming.org/) which promotes responsible language and tries to avoid harmful terms. As an example, the Kubernetes project has replaced the potentially harmful term “master” with “control plane node”. This articles does the same.

36. [Optional] In the terminal we will replace the ip with a friendly host name. Run the following command, but replace `control-plane` with `worker1` and `worker2` , when running in respective terminals.

37. Not `exit` and ssh again, you should see the host name instead of ip.

![](https://miro.medium.com/v2/resize:fit:1400/1*szqsh3W5cctLh7QQM131sw.png)

Worker 1 terminal:

![](https://miro.medium.com/v2/resize:fit:1400/1*cuLrPlPgcpBUOEl41cZcUw.png)

Worker 2 terminal:

![](https://miro.medium.com/v2/resize:fit:1400/1*u2DmWNmrbIg-MMZcvZui9w.png)

## Install a container runtime

To run containers in Pods, Kubernetes uses a [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes).

By default, Kubernetes uses the [Container Runtime Interface](https://kubernetes.io/docs/concepts/overview/components/#container-runtime) (CRI) to interface with your chosen container runtime.

If you don’t specify a runtime, kubeadm automatically tries to detect an installed container runtime by scanning through a list of known endpoints.

If multiple or no container runtimes are detected kubeadm will throw an error and will request that you specify which one you want to use.

Some of the container runtimes:

38. containerd (pronounced _container-dee_)
39. CRI-O
40. Docker Engine (using cri-dockerd)

In this demo we will install containerd.

To make things simple, I prepared an automated script based on official documentation that will install containerd in all three nodes.

Note: run `sudo apt-get update` in all three nodes before executing next commands.

41. Create a new `containerd-install.sh` file in all three nodes and paste the above gist contents.

vim ./containerd-install.sh

42. Make the file executable

chmod u+x ./containerd-install.sh

![](https://miro.medium.com/v2/resize:fit:1400/1*Xtw2U72MSzbh3GPRnINiYg.png)

43. Execute the file

![](https://miro.medium.com/v2/resize:fit:1400/1*iB6AofZMp036LnKBzUCIAQ.png)

44. Check status of containerd, it should be “active”

service containerd status

![](https://miro.medium.com/v2/resize:fit:1400/1*hxwcy5AHVRPLdLvyui1iYw.png)

## Install kubeadm, kubelet and kubectl

I prepared a similar script to install kubeadm, kubelet and kubectl in all three nodes.

45. Create a new `k8s-install.sh` file in all three nodes and paste the contents of above gist.

vim ./k8s-install.sh

![](https://miro.medium.com/v2/resize:fit:1400/1*qcRQNkDZ_Og8CkaAQSasaQ.png)

46. Make it executable

chmod u+x ./k8s-install.sh

![](https://miro.medium.com/v2/resize:fit:1400/1*aEVS7p0bHlAdzk9EZgW6JQ.png)

47. Execute the script in all three nodes

![](https://miro.medium.com/v2/resize:fit:1400/1*iiXYamrThwkNhFshgWJrBQ.png)

Control Plane

![](https://miro.medium.com/v2/resize:fit:1400/1*74OxM92PbLFt5Gv_IFeZkA.png)

Worker 1

![](https://miro.medium.com/v2/resize:fit:1400/1*CvGLhcDx02WYGVCCzU9_eA.png)

Worker 2

48. Test it with the command

kubeadm version

![](https://miro.medium.com/v2/resize:fit:1400/1*OSE6uFEjl7yyQ0az7lPhHA.png)

At this point kubelet service should be inactive because we haven’t initialised the cluster. Confirm it with the following command:

service kubelet status

![](https://miro.medium.com/v2/resize:fit:1400/1*Z4pHH3m84YrsECitUm9wyA.png)

Congratulations on getting so far, give yourself a pat on the back )

# Initialise Cluster

From the control plane node, please run

sudo kubeadm init

![](https://miro.medium.com/v2/resize:fit:1400/1*D2zi15mS1Xa08DBQqCKxhg.png)

Once it finishes executing, copy the commands after “To start using your cluster, you need to run the following as a regular user:” line

![](https://miro.medium.com/v2/resize:fit:1400/1*1GjxAuHltS5--VvuQu4zEQ.png)

When we will execute commands using `kubectl` , kubectl will look for certificates in the configuration file at `~/.kube/config` location. But kubernetes puts the default admin config file in `/etc/kubernetes/admin.conf`location.

The above commands copies the files from kubernetes default location and copies them to `~/.kube/config` file (and also gives current user permissions).

mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config

If you do not want to persist the config file between server restarts using above commands then you must follow the below synax for every kubectl command (which is adding suffix `--kubeconfig /etc/kubernetes/admin.conf` ):

sudo kubectl -n kube-system get pod --kubeconfig /etc/kubernetes/admin.conf

_Note: also copy_ `_kubeadm join_` _command in the last line from the output of_ `_sudo kubeadm init_`_._

Check the status of the static pods

kubectl get pod -A

At this point, if you’re fortunate, the API server should respond. However, after a while or immediately, it might throw a “connection refused” or “incorrect host” error.

During container restarts, when the kubelet is attempting to revive crashed containers, accessing the logs of core components and resource like the API server or etcd server may only display a “pod sandbox changed” error.

The ability to access kubelet and containerd logs occurs when the API server pod doesn’t respond because, at that juncture, the cluster becomes inaccessible. In the backend, the kubelet continually restarts the pods.

If any of your ‘get pod’ commands are successful, you’ll notice the number of restarts or crash loops for all pods in the kube-system namespace.

![](https://miro.medium.com/v2/resize:fit:1400/1*K5B-CKcL5kuvsz7LylYS5g.png)

Pod restarts

It didn’t used to be like that before, but if you are working with the same stack as described in this article, your static pods may also be experiencing failures and throwing the “pod sandbox changed” error.

# Fix cluster (by fixing containerd)

Numerous online solutions address the “pod sandbox changed” error, but most of them focus on generic issues such as incorrect hostnames, memory or storage pressure, and networking errors.

In our particular scenario, this is not the case. However, when similar errors occur, and containers are also failing, the kubelet will persistently restart them. In such instances, we can uncover at least one clue in the haystack by examining the error logs.

As per the [github’s containerd installation instructions](https://github.com/containerd/containerd/blob/main/docs/cri/config.md#cgroup-driver):

> While containerd and Kubernetes use the legacy `cgroupfs` driver for managing cgroups by default, it is recommended to use the `systemd` driver on systemd-based hosts for compliance of [the "single-writer" rule](https://systemd.io/CGROUP_DELEGATION/) of cgroups.
> 
> To configure containerd to use the `systemd` driver, set the following option in `/etc/containerd/config.toml`:

version = 2  
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]  
  SystemdCgroup = true

And [this github issue](https://github.com/kubernetes/kubernetes/issues/111469#issuecomment-1216683304) on the same error also referred to the above documentation page.

So we will apply this fix, which bascially means we will default to `systemd`driver in our containerd runtime which kubernetes already expects.

49. Edit `/etc/containerd/config.toml` file from control plane node

sudo vim /etc/containerd/config.toml

50. Change the `SystemCgroup` key from `false` to `true` under `plugins` object

![](https://miro.medium.com/v2/resize:fit:1400/1*fm0zHKs0j-vQ0N6wKHPtjw.png)

_Note: In later section we will need to apply the exact same fix to both worker nodes._

51. Restart containerd and kubelet services

sudo service containerd restart  
sudo service kubelet restart

Now when you will run kubctl commands, the api server will always respond, plus container restart counts should not increase.

![](https://miro.medium.com/v2/resize:fit:1400/1*mQ3GdlOfgk71VaC0leqG_w.png)

_Note: the number of restarts, was because of the containerd conf file. Kubelet can only try to restart the container, it is not responsible for container runtime._

You will notice now that none of the core-dns pods are running. It is because we haven’t deployed a pod network yet. So let’s deploy one.

![](https://miro.medium.com/v2/resize:fit:1400/1*ylYfDhgTALgb5NlFPHMbLg.png)

# Deploy CNI Weave net

Weave Net is a popular Kubernetes add-on that provides networking and network policy solutions for containerized applications. It facilitates communication between containers across different hosts, creating a virtual network overlay.

Weave Net offers features like DNS-based service discovery, automatic IP address management, and encryption for communication between containers. It is a much better then using route tables.

![](https://miro.medium.com/v2/resize:fit:1400/1*3duE6b90TsWRszFHXUCMhQ.png)

Pod network

Alternative networking solutions for Kubernetes include Flannel, Calico, Cilium and others.

## Add ports in security groups

As per the [documentation](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation) weave net requires TCP 6783 and UDP 6783/6784 to be open in the firewall. So we must add them in our inbound rules in security group of control plane and worker.

Final security group of control plane:

![](https://miro.medium.com/v2/resize:fit:1400/1*NTRWGC2dixPUgBu0rjTC8g.png)

Control plane security group with Weave net port additions

Final security group of worker nodes:

![](https://miro.medium.com/v2/resize:fit:1400/1*O0sbTqVvUjrjXKjjgk7Hbw.png)

Security group for worker nodes with Weave net port additions

## Apply weave net

From the [documentation](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation) execute the following command:

kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

Now if you will run `kubectl get pod -A` , you will see that not only coredns pods are up but 2 new weave nets pods are created.

![](https://miro.medium.com/v2/resize:fit:1400/1*8JyB2SdkQUF2gFeE7qv8XQ.png)

So far, we have a stable control plane with pod networking. Now we need to add worker nodes and see if control plane can deploy them to both worker nodes successfully before deploying ingress service.

Good job on getting so far :)

# Join worker nodes

## Fix containerd

To prevent crash loop containers, we will apply the same fix to containerd conf file that we did in control plane node.

52. Edit `/etc/containerd/config.toml` file from control plane node

sudo vim /etc/containerd/config.toml

53. Change the `SystemCgroup` key from `false` to `true` under `plugins` object

![](https://miro.medium.com/v2/resize:fit:1400/1*fm0zHKs0j-vQ0N6wKHPtjw.png)

## Join worker nodes

When we executed `sudo kubeadm init` command, at the end of the verbose output, kubeadm gave a joining commands for the worker nodes.

![](https://miro.medium.com/v2/resize:fit:1400/1*a_i8Vjqelg8_D1yfEqWkLA.png)

If you missed that command then you can generate join command again by executing the following command from control plane:

kubeadm token create --print-join-command

![](https://miro.medium.com/v2/resize:fit:1400/1*yHxu9WEcoD0lM7Js6J4KsA.png)

Copy the command and paste it in the terminal of both worker nodes one by one.

## Check worker nodes cluster joining status

When we executed `kubeadm join` command on both worker nodes, our control plane took hold of both worker nodes and started kube-proxy pods.

kube-proxy is responsible for analysing whether a new pod/deployment manifest received from api server, can be started on the node or not. If it can be then it passes the conf to container runtime. If node do not has enough resources to deploy the resource then it will inform the same to the control plane.

Check whether kube-proxy pods are running on both worker nodes or not by executing the following command from control plane node:

kubectl get pod -A -o wide

![](https://miro.medium.com/v2/resize:fit:1400/1*T_3V5d1R37DMmcBYnQMZsA.png)

You should see that each of the worker nodes has one kube-proxy pod running.

We also see weave net pods running on both worker nodes.

kubectl get pod -A -o wide | grep weave-net

![](https://miro.medium.com/v2/resize:fit:1400/1*HUqqmQo1lvhhy1wgq0robA.png)

_Note: We don’t have to apply the same weave net yaml file in worker nodes that we applied on control plane._

We can also check kubelet service on both worker nodes, earlier it was in inactive state, but now after `kubeadm join` command it should be running:

![](https://miro.medium.com/v2/resize:fit:1400/1*ShqCzdVXjg2Wt0gOZBLJtw.png)

kublet service worker 1

![](https://miro.medium.com/v2/resize:fit:1400/1*gyQkZ3432SMqcxQaTbWwAQ.png)

kubelet service worker 2

_Note: kubectl commands like get pod, get node will not work on any of the worker nodes._

Let’s check weave net container logs from control plane:

kubectl logs -n kube-system weave-net-mkm6d

![](https://miro.medium.com/v2/resize:fit:1400/1*m7YVYxEQt7o8TYxCWaHFPA.png)

## Test nginx static pods

Now we can test whether our apps can be deployed onboth worker nodes or not. If you have come so far and observed no errors then there is no reason for worker nodes to not run your deployments.

Run a test nginx pod

kubectl run test --image=nginx

Get pods without `-A` or namespace flag

kubectl get pod -o wide

![](https://miro.medium.com/v2/resize:fit:1400/1*2heajh141WWu4JzcvH_AyA.png)

Now let’s deploy another nginx pod to test worker 2 node.

Note: control plane will automatically run it the new pod on worker2 to balance the cluster

kubectl run test2 --image=nginx

Check pods

kubectl get pod -o wide

![](https://miro.medium.com/v2/resize:fit:1400/1*QhNbIaBlABt0z8AOgrwfSw.png)

Congratulations, our control plane and both the worker nodes are working as intended.

# Deploy ingress

In Kubernetes, an Ingress is an API object that manages external access to services within a cluster. It serves as a way to expose HTTP and HTTPS routes to applications, allowing external traffic to reach services based on specified rules. In comparison to other service types like LoadBalancer and NodePort, Ingress provides more advanced routing capabilities and is especially suited for managing HTTP/S traffic.

**LoadBalancer vs. Ingress:**

- LoadBalancer typically works at the service level and requires cloud-provider integration. It provisions an external load balancer to distribute traffic to the associated services.
- Ingress provides HTTP/S routing and allows more sophisticated path-based and host-based routing rules. It does not create an external load balancer by itself but can leverage an existing one.

**NodePort vs. Ingress:**

- Exposes a service on a static port on each cluster node, making the service accessible externally but only on specific node ports.
- Suitable for scenarios where external access is required, but without the advanced routing capabilities provided by Ingress.

## How Ingress Works

**Ingress Resource:**

- Users define Ingress rules in Kubernetes using the Ingress resource. Rules specify the backend services, paths, and hostnames for routing.

**Ingress Controller:**

- An Ingress Controller, a software component deployed in the cluster, interprets and enforces Ingress rules. It could be Nginx, Traefik, or another controller depending on the choice made.

**Routing:**

- Ingress controllers manage external access by routing incoming requests based on defined rules to the appropriate services within the cluster.

## Installing Ingress Controller using Helm

There are multiple ways to install the Ingress-Nginx Controller:

- with [Helm](https://helm.sh/), using the project repository chart
- with `kubectl apply`, using YAML manifests
- with specific addons (e.g. for [minikube](https://kubernetes.github.io/ingress-nginx/deploy/#minikube) or [MicroK8s](https://kubernetes.github.io/ingress-nginx/deploy/#microk8s))

We will install ingress-nginx controller using helm.

Helm, a Kubernetes package manager, simplifies the installation of an Ingress Controller. Users can use Helm charts provided by the Ingress Controller project to deploy and configure the controller in the cluster.

- For cloud-managed Kubernetes clusters like AWS EKS or GCP GKE, they often come with their own native Ingress controllers.
- AWS EKS, for instance, can integrate with AWS Network Load Balancer (NLB) for external access.
- GCP GKE uses the GKE Ingress controller to handle external traffic routing.
- And if your on service mesh, then you don’t need to use ingress.

**Install Helm**

The [helm official documentation](https://helm.sh/docs/intro/install/) provides an [apt-get installation method for ubuntu OS](https://helm.sh/docs/intro/install/#from-apt-debianubuntu), and we will use that:

curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null  
sudo apt-get install apt-transport-https --yes  
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt-get update  
sudo apt-get install helm

![](https://miro.medium.com/v2/resize:fit:1400/1*sTl6bcxlHwjzYebZ9gHzkA.png)

Inspect installation by executing command `helm version`

![](https://miro.medium.com/v2/resize:fit:1400/1*TPMkjmk6H0IiMFWws1aB_w.png)

**Deploy ingress-nginx controller**

Use the following commands from the [official kubernetes ingress-nginx github repo](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start):

helm upgrade --install ingress-nginx ingress-nginx \  
  --repo https://kubernetes.github.io/ingress-nginx

![](https://miro.medium.com/v2/resize:fit:1400/1*RndhUU4R4um7Lyj_bHsWvA.png)

List the services

kubectl get svc

![](https://miro.medium.com/v2/resize:fit:1400/1*llTARgNeIwfqcPVFaxlp1w.png)

Check the pods

kubectl get pod -o wide

![](https://miro.medium.com/v2/resize:fit:1400/1*Cet3GRcVYJ-uDPeeoQVBTQ.png)

_Note: We are not deploying LoadBalancer, so our ingress will only be accessible using node ip._

## Deploy a path based ingress service

In this example we will deploy a path based ingress service. You can modify the below yaml to deploy a host based service too.

Above file is taken from [minikube example](https://minikube.sigs.k8s.io/docs/start/) and can also be downloaded from : `[https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml](https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml)`

In the yaml file we have:

- 2 static pods (static means without any controllers)
- 2 ClusterIP services, one for each pod (if don’t mention NodePort or LoadBalancer then service defaults to ClusterIP)
- One Ingress service called `example-ingress`

![](https://miro.medium.com/v2/resize:fit:1400/1*KPAQQwYiuXAPVOkspsBBrw.png)

Ingress path based routing

The `example-ingress` service has two paths `/foo` and `/bar` , each forwarding requests to pod services `foo-service` and `bar-service` , both on port `8080`.

The `foo-service` and `bar-service` are listening on port `8080` and directing requests to pods that have labels `app: foo` and `app: bar` respectively.

**Deploy ingress service**

kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml

![](https://miro.medium.com/v2/resize:fit:1400/1*A7rV76jP6e4QmxA6A8wP5w.png)

**Check services**

kubectl get svc

![](https://miro.medium.com/v2/resize:fit:1400/1*q_ZnDEwr3I0MuBoLzAP4Rw.png)

**Pods**

![](https://miro.medium.com/v2/resize:fit:1400/1*Cet3GRcVYJ-uDPeeoQVBTQ.png)

## Access service from inside the cluster

Copy the node ip of the cluster and invoke the following command

curl CLUSTER_IP/foo  
curl CLUSTER_IP/bar

![](https://miro.medium.com/v2/resize:fit:1400/1*Q4pBjKbfSDKpjiGciGJQGg.png)

## Access service from outside the cluster

To access our ingress-nginx service from outside the cluster we will need to a add a application load balancer and add our worker nodes in its target group.

![](https://miro.medium.com/v2/resize:fit:1400/1*LOduv3cmXSP2VeiXT3zkMQ.png)

Ingress service with AWS Load balancer

After that we can copy the **F**ully **Q**ualified **D**omain **N**ame (FQDN) of any of the worker nodes, and pass either `/foo` or `/bar` in the URL.

![](https://miro.medium.com/v2/resize:fit:1400/1*-CFKwbSujxG68_-SEYQ6Yg.png)

Example of `/foo` path:

![](https://miro.medium.com/v2/resize:fit:1400/1*EbaFy47eUwh5R14qAuaL7w.png)

Example of `/bar` path:

![](https://miro.medium.com/v2/resize:fit:1400/1*XUnjjx0dxTWZnhcSj6PgVQ.png)

_Note: the root URL will be unavailable._

![](https://miro.medium.com/v2/resize:fit:1400/1*jTCVi67ETR1fEJYQZUZa5g.png)

**How worker 2 responds when only worker node 1 has ingress nginx pods?**

That happens due to service discovery. In a setup with an Ingress service and a NodePort service (without a LoadBalancer), when a request is made, it reaches one of the worker nodes’ IP addresses with the designated NodePort.

The Ingress Controller deployed within the Kubernetes cluster, residing on the chosen node, interprets the request based on rules defined in the Ingress resource and routes the traffic to the appropriate backend service. Subsequently, the NodePort service, which allocates a static port on each node, directs the request to the specific worker node and pod running the application, facilitating external access without the need for an external LoadBalancer.

Congratulations! You have successfully deployed a bootstrap kubernetes cluster and added ingress service for path based routing.

Deploy other apps like ml models, analytical apps, using kubernetes manifests and monitor your cluster using [Kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

![](https://miro.medium.com/v2/resize:fit:1400/1*faKikYgA_qCqcjxzaVITzw.png)

Kubernetes dashboard UI

# Use cases for data science

Bootstrapping your own Kubernetes cluster on EC2 instances with Ingress service empowers data scientists and machine learning engineers to efficiently test various models and assess their scalability.

For instance, using Ingress path-based routing, you can easily evaluate different machine learning models by directing traffic to specific versions or instances for comparative analysis.

Additionally, this cluster facilitates seamless experimentation with distributed training, data processing, and real-time analytics, offering practical applied examples such as deploying scalable data pipelines, conducting resource-intensive simulations, and customizing data services tailored to specific research needs.

Besides all the help one can get from such an arrangement, it is a phenomenal undertaking to understand how kubernetes works under the hood.

We have barely scratched the surface in this article because we did not deploy any stateful sets, set up RBAC, used any jump pods or even touched upon many production essential kubernetes resources. But you can play around with your cluster at your own time and stop all the instances when required. Just edit the ip address of all nodes in `/etc/hosts` file on all nodes after reboot to take advantage of the node names.

You can connect with me on linkedin: [https://linkedin.com/in/maheshrajput](https://linkedin.com/in/maheshrajput)

Github repo that has `containerd-install.sh`, `k8s-install.sh` and `ingress-example.yaml` files:

[

## mrmaheshrajput/bootstrap-k8s

### Bootstrapped Kubernetes cluster on ec2.

github.com



](https://github.com/mrmaheshrajput/bootstrap-k8s?source=post_page-----f3eeca9e95f1--------------------------------)

[

Kubernetes

](https://medium.com/tag/kubernetes?source=post_page-----f3eeca9e95f1--------------------------------)

[

DevOps

](https://medium.com/tag/devops?source=post_page-----f3eeca9e95f1--------------------------------)

[

Mlops

](https://medium.com/tag/mlops?source=post_page-----f3eeca9e95f1--------------------------------)

[

Container Orchestration

](https://medium.com/tag/container-orchestration?source=post_page-----f3eeca9e95f1--------------------------------)

[

Microservices

](https://medium.com/tag/microservices?source=post_page-----f3eeca9e95f1--------------------------------)

2

7

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Ff3eeca9e95f1&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fdeploy-kubernetes-cluster-on-aws-ec2-instances-f3eeca9e95f1&source=---footer_actions--f3eeca9e95f1---------------------bookmark_footer-----------)

[

![Mahesh](https://miro.medium.com/v2/resize:fill:96:96/1*wvFXUaXI0WWsbpuyT2iTEQ.jpeg)



](https://mrmaheshrajput.medium.com/?source=post_page---post_author_info--f3eeca9e95f1--------------------------------)

[

## Written by Mahesh

](https://mrmaheshrajput.medium.com/?source=post_page---post_author_info--f3eeca9e95f1--------------------------------)

[270 Followers](https://mrmaheshrajput.medium.com/followers?source=post_page---post_author_info--f3eeca9e95f1--------------------------------)

·[3 Following](https://mrmaheshrajput.medium.com/following?source=post_page---post_author_info--f3eeca9e95f1--------------------------------)

Machine Learning Engineer | ML Ops | [https://mrmaheshrajput.github.io/](https://mrmaheshrajput.github.io/)

Follow

## Responses (7)

[](https://policy.medium.com/medium-rules-30e5502c4eb4?source=post_page---post_responses--f3eeca9e95f1--------------------------------)

[

What are your thoughts?



](https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fdeploy-kubernetes-cluster-on-aws-ec2-instances-f3eeca9e95f1&source=---post_responses--f3eeca9e95f1---------------------respond_sidebar-----------)

Cancel

Respond

Respond

Also publish to my profile

[

![Karthik Potnuru](https://miro.medium.com/v2/resize:fill:64:64/1*dmbNkD5D-u45r44go_cf0g.png)





](https://medium.com/@gclouduser143?source=post_page---post_responses--f3eeca9e95f1----0----------------------------)

[

Karthik Potnuru



](https://medium.com/@gclouduser143?source=post_page---post_responses--f3eeca9e95f1----0----------------------------)

[

Nov 2, 2024

](https://medium.com/@gclouduser143/hello-mahesh-d9cbc286c50c?source=post_page---post_responses--f3eeca9e95f1----0----------------------------)

Hello Mahesh

Thank you so much for such detailed article....

that containerd "pod sandbox change" issue resolved after reading your article.

you took time to explain the deployment so clear.

And I took my time to create medium account just to post this comment.

Thanks again...more

1

1 reply

Reply

[

![Boyapati Nagendra](https://miro.medium.com/v2/resize:fill:64:64/0*YKl29fMeMyWUy0L5)





](https://medium.com/@boyapatinagen?source=post_page---post_responses--f3eeca9e95f1----1----------------------------)

[

Boyapati Nagendra



](https://medium.com/@boyapatinagen?source=post_page---post_responses--f3eeca9e95f1----1----------------------------)

[

May 10, 2024

](https://medium.com/@boyapatinagen/it-is-one-of-the-best-article-for-k8s-thank-you-b916eacbf46a?source=post_page---post_responses--f3eeca9e95f1----1----------------------------)

It is one of the best article for K8s. Thank you.

1

Reply

[

![Premkumardivakaran](https://miro.medium.com/v2/resize:fill:64:64/1*dmbNkD5D-u45r44go_cf0g.png)



](https://medium.com/@premkumardivakaran10?source=post_page---post_responses--f3eeca9e95f1----2----------------------------)

[

Premkumardivakaran



](https://medium.com/@premkumardivakaran10?source=post_page---post_responses--f3eeca9e95f1----2----------------------------)

[

Apr 11, 2024

](https://medium.com/@premkumardivakaran10/thank-you-for-sharing-such-a-comprehensive-and-insightful-article-43b9ac08b097?source=post_page---post_responses--f3eeca9e95f1----2----------------------------)

Thank you for sharing such a comprehensive and insightful article! Your clear and in-depth explanation really helped me.

1

Reply

See all responses

## More from Mahesh

![LLM Cache: The Secret Weapon You’re Overlooking](https://miro.medium.com/v2/resize:fit:1358/0*oHlkKlEh4EjhG77x)

[

![Mahesh](https://miro.medium.com/v2/resize:fill:40:40/1*wvFXUaXI0WWsbpuyT2iTEQ.jpeg)



](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----0---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

Mahesh

](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----0---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

## LLM Cache: The Secret Weapon You’re Overlooking

### Discover how LLM Caching can supercharge your AI performance and slash response times by up to 80%!



](https://mrmaheshrajput.medium.com/llm-cache-the-secret-weapon-youre-overlooking-a26b6ddab7cc?source=post_page---author_recirc--f3eeca9e95f1----0---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

Jul 28, 2024

[

28





](https://mrmaheshrajput.medium.com/llm-cache-the-secret-weapon-youre-overlooking-a26b6ddab7cc?source=post_page---author_recirc--f3eeca9e95f1----0---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fa26b6ddab7cc&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fllm-cache-the-secret-weapon-youre-overlooking-a26b6ddab7cc&source=---author_recirc--f3eeca9e95f1----0-----------------bookmark_preview----4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

![Deploy LLM with vLLM on SageMaker in only 13 lines of code](https://miro.medium.com/v2/resize:fit:1358/0*Dm5TenthKty-vA4T)

[

![Mahesh](https://miro.medium.com/v2/resize:fill:40:40/1*wvFXUaXI0WWsbpuyT2iTEQ.jpeg)



](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----1---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

Mahesh

](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----1---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

## Deploy LLM with vLLM on SageMaker in only 13 lines of code

### Effortless deployment of HuggingFace models in production on SageMaker endpoint with streaming output.



](https://mrmaheshrajput.medium.com/deploy-llm-with-vllm-on-sagemaker-in-only-13-lines-of-code-1601f780c0cf?source=post_page---author_recirc--f3eeca9e95f1----1---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

Jul 11, 2024

[

50

1





](https://mrmaheshrajput.medium.com/deploy-llm-with-vllm-on-sagemaker-in-only-13-lines-of-code-1601f780c0cf?source=post_page---author_recirc--f3eeca9e95f1----1---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F1601f780c0cf&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fdeploy-llm-with-vllm-on-sagemaker-in-only-13-lines-of-code-1601f780c0cf&source=---author_recirc--f3eeca9e95f1----1-----------------bookmark_preview----4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

![How to Productionize Large Language Models (LLMs)](https://miro.medium.com/v2/resize:fit:1358/1*-_O4sOfkV2PBO1VoVBgcmg.png)

[

![Mahesh](https://miro.medium.com/v2/resize:fill:40:40/1*wvFXUaXI0WWsbpuyT2iTEQ.jpeg)



](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----2---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

Mahesh

](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----2---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

## How to Productionize Large Language Models (LLMs)

### Understand LLMOps, architectural patterns, how to evaluate, fine tune & deploy HuggingFace generative AI models locally or on cloud.



](https://mrmaheshrajput.medium.com/how-to-productionize-large-language-models-llms-060a4cb1a169?source=post_page---author_recirc--f3eeca9e95f1----2---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

Mar 27, 2024

[

333

6





](https://mrmaheshrajput.medium.com/how-to-productionize-large-language-models-llms-060a4cb1a169?source=post_page---author_recirc--f3eeca9e95f1----2---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F060a4cb1a169&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fhow-to-productionize-large-language-models-llms-060a4cb1a169&source=---author_recirc--f3eeca9e95f1----2-----------------bookmark_preview----4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

![How to Build an Intelligent QA Chatbot on your data with LLM or ChatGPT](https://miro.medium.com/v2/resize:fit:1358/0*TcqXKTh4R6_msoTX)

[

![Mahesh](https://miro.medium.com/v2/resize:fill:40:40/1*wvFXUaXI0WWsbpuyT2iTEQ.jpeg)



](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----3---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

Mahesh

](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1----3---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

## How to Build an Intelligent QA Chatbot on your data with LLM or ChatGPT

### Welcome to the world of intelligent chatbots empowered by large language models (LLMs)!



](https://mrmaheshrajput.medium.com/how-to-build-an-intelligent-qa-chatbot-on-your-data-with-llm-or-chatgpt-d0009d256dce?source=post_page---author_recirc--f3eeca9e95f1----3---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

Jul 2, 2023

[

148

4





](https://mrmaheshrajput.medium.com/how-to-build-an-intelligent-qa-chatbot-on-your-data-with-llm-or-chatgpt-d0009d256dce?source=post_page---author_recirc--f3eeca9e95f1----3---------------------4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fd0009d256dce&operation=register&redirect=https%3A%2F%2Fmrmaheshrajput.medium.com%2Fhow-to-build-an-intelligent-qa-chatbot-on-your-data-with-llm-or-chatgpt-d0009d256dce&source=---author_recirc--f3eeca9e95f1----3-----------------bookmark_preview----4a96c226_6cf1_4dde_91cb_0ff77076825a-------)

[

See all from Mahesh

](https://mrmaheshrajput.medium.com/?source=post_page---author_recirc--f3eeca9e95f1--------------------------------)

## Recommended from Medium

![Technical Guide: End-to-End CI/CD DevOps with Jenkins, Docker, Kubernetes, ArgoCD, Github Actions , AWS EC2 and Terraform by Joel .O Wembo](https://miro.medium.com/v2/resize:fit:1358/1*rlPRYAfJ_aFOfoj52U7y2A.png)

[

![Django Unleashed](https://miro.medium.com/v2/resize:fill:40:40/1*nXyvQ3PIVmE63EPQ1uSIHw.png)



](https://medium.com/django-unleashed?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

In

[

Django Unleashed

](https://medium.com/django-unleashed?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

by

[

Joel Wembo

](https://joelotepawembo.medium.com/?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

## Technical Guide: End-to-End CI/CD DevOps with Jenkins, Docker, Kubernetes, ArgoCD, Github Actions …

### Building an end-to-end CI/CD pipeline for Django applications using Jenkins, Docker, Kubernetes, ArgoCD, AWS EKS, AWS EC2



](https://joelotepawembo.medium.com/technical-guide-end-to-end-ci-cd-devops-with-jenkins-docker-kubernetes-argocd-github-actions-fee466fe949e?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

Apr 12, 2024

[

1K

21





](https://joelotepawembo.medium.com/technical-guide-end-to-end-ci-cd-devops-with-jenkins-docker-kubernetes-argocd-github-actions-fee466fe949e?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Ffee466fe949e&operation=register&redirect=https%3A%2F%2Fmedium.com%2Fdjango-unleashed%2Ftechnical-guide-end-to-end-ci-cd-devops-with-jenkins-docker-kubernetes-argocd-github-actions-fee466fe949e&source=---read_next_recirc--f3eeca9e95f1----0-----------------bookmark_preview----895847f3_adfb_4268_b26a_ee94586dd911-------)

![Create an EKS Cluster Using Terraform](https://miro.medium.com/v2/resize:fit:1358/1*_xufsgFYLBKREZP6m4PBGw.png)

[

![Ravindra singh](https://miro.medium.com/v2/resize:fill:40:40/1*EWUZl3EXtSOa96Uus1LfFg.jpeg)



](https://medium.com/@rvisingh1221?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

Ravindra singh

](https://medium.com/@rvisingh1221?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

## Create an EKS Cluster Using Terraform

### In this blog, we’ll explore how to create an EKS cluster using a Terraform module, including setting up a node group, , ECR, ACM, and…



](https://medium.com/@rvisingh1221/create-an-eks-cluster-using-terraform-329b9dde068f?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

Aug 24, 2024

[

15





](https://medium.com/@rvisingh1221/create-an-eks-cluster-using-terraform-329b9dde068f?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F329b9dde068f&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40rvisingh1221%2Fcreate-an-eks-cluster-using-terraform-329b9dde068f&source=---read_next_recirc--f3eeca9e95f1----1-----------------bookmark_preview----895847f3_adfb_4268_b26a_ee94586dd911-------)

## Lists

[

![](https://miro.medium.com/v2/resize:fill:96:96/1*zPtGTCNOwu1p3kzn_sZFVQ.png)

![](https://miro.medium.com/v2/da:true/resize:fill:96:96/0*kQIvhDkl0ixPpv4z)

![](https://miro.medium.com/v2/resize:fill:96:96/1*ERYx0IB1pN-5ZX98cKAoUw.png)

## General Coding Knowledge

20 stories·1906 saves



](https://eddiebarth.medium.com/list/general-coding-knowledge-f2d429d4f0cd?source=post_page---read_next_recirc--f3eeca9e95f1--------------------------------)

[

![AI opportunities are found at the intersection of problem space and information space. Problems that involved cognitive tasks are “worth solving with AI”. Problems for which sufficient data is available are “feasible to be solved with AI”](https://miro.medium.com/v2/resize:fill:96:96/1*1Fr7fPf0BXKpGFwf29U5uQ.png)

![](https://miro.medium.com/v2/resize:fill:96:96/1*Da9W2a6Oo90h_TKXNH6y4Q.png)

![](https://miro.medium.com/v2/resize:fill:96:96/1*bhNlT_X-caEz7uBUObvURQ.png)

## Business

41 stories·176 saves



](https://medium.com/@bramdevogel/list/business-b13ff2fbaf67?source=post_page---read_next_recirc--f3eeca9e95f1--------------------------------)

[

![](https://miro.medium.com/v2/resize:fill:96:96/1*d9bMmPsCC0PWpq_hmdwtSw.jpeg)

![ACO-ToT is a novel algorithm that enhances LLM reasoning by integrating Ant Colony Optimization (ACO) with a Tree of Thought (ToT) framework. Inspired by Hebbian learning, it uses fine-tuned LLM “ants” to explore reasoning paths, reinforcing effective ones with pheromones. A mixture-of-experts scoring function evaluates solutions, optimizing complex problem-solving. Experiments show ACO-ToT outperforms existing methods, highlighting the potential of biologically inspired search in LLM inference.](https://miro.medium.com/v2/resize:fill:96:96/1*iGCsR8hjV9D0-H3BF2wanQ.png)

![](https://miro.medium.com/v2/resize:fill:96:96/1*uN0srATFTg47adbEkuk3Xw.png)

## Natural Language Processing

1928 stories·1583 saves



](https://medium.com/@AMGAS14/list/natural-language-processing-0a856388a93a?source=post_page---read_next_recirc--f3eeca9e95f1--------------------------------)

[

![](https://miro.medium.com/v2/resize:fill:96:96/1*9zbsamK8FsVPes5pYkZbWw.png)

![Searching](https://miro.medium.com/v2/da:true/resize:fill:96:96/0*CDDpt-pT5GHM2_oO)

![](https://miro.medium.com/v2/da:true/resize:fill:96:96/1*vPnRqbnFYiKz30kKrOCd5Q.gif)

## Productivity

244 stories·673 saves



](https://dvarlot.medium.com/list/productivity-9a1d5b7c96e7?source=post_page---read_next_recirc--f3eeca9e95f1--------------------------------)

![Deploy a Kubernetes Application With Terraform and AWS EKS](https://miro.medium.com/v2/resize:fit:1358/1*eXC923lioX1x1BfAYeNcoQ.jpeg)

[

![Container Talks](https://miro.medium.com/v2/resize:fill:40:40/1*Q5-VXdsrRXF25Tf8qU5pUw.png)



](https://medium.com/container-talks?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

In

[

Container Talks

](https://medium.com/container-talks?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

by

[

Alfonso Valdes Carrales

](https://medium.com/@ponchovaldescarrales?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

## Deploy a Kubernetes Application With Terraform and AWS EKS

### Let’s take a look at the steps to provision an EKS Cluster using Terraform.



](https://medium.com/@ponchovaldescarrales/deploy-a-kubernetes-application-with-terraform-and-aws-eks-1085662213dc?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

Oct 5, 2024

[

6





](https://medium.com/@ponchovaldescarrales/deploy-a-kubernetes-application-with-terraform-and-aws-eks-1085662213dc?source=post_page---read_next_recirc--f3eeca9e95f1----0---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F1085662213dc&operation=register&redirect=https%3A%2F%2Fmedium.com%2Fcontainer-talks%2Fdeploy-a-kubernetes-application-with-terraform-and-aws-eks-1085662213dc&source=---read_next_recirc--f3eeca9e95f1----0-----------------bookmark_preview----895847f3_adfb_4268_b26a_ee94586dd911-------)

![Building a Real-Time CI/CD Pipeline: Setting Up Development Infrastructure for Microservices…](https://miro.medium.com/v2/resize:fit:1358/1*yuVzMhCJyDENbyhwAsrkwA.png)

[

![Adnan Turgay Aydin](https://miro.medium.com/v2/resize:fill:40:40/1*V6gC3EcXGB4b91zTn_jMbQ.jpeg)



](https://medium.com/@adnanturgayaydin?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

Adnan Turgay Aydin

](https://medium.com/@adnanturgayaydin?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

## Building a Real-Time CI/CD Pipeline: Setting Up Development Infrastructure for Microservices…

### In the previous part of this guide, we laid the groundwork for building a CI/CD pipeline for microservices projects. We covered how to…



](https://medium.com/@adnanturgayaydin/building-a-real-time-ci-cd-pipeline-setting-up-development-infrastructure-for-microservices-c079739248f2?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

Sep 6, 2024

[

13





](https://medium.com/@adnanturgayaydin/building-a-real-time-ci-cd-pipeline-setting-up-development-infrastructure-for-microservices-c079739248f2?source=post_page---read_next_recirc--f3eeca9e95f1----1---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fc079739248f2&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40adnanturgayaydin%2Fbuilding-a-real-time-ci-cd-pipeline-setting-up-development-infrastructure-for-microservices-c079739248f2&source=---read_next_recirc--f3eeca9e95f1----1-----------------bookmark_preview----895847f3_adfb_4268_b26a_ee94586dd911-------)

![Terraform Mastery : An In-Depth Exploration of Key Concepts and Best Practices](https://miro.medium.com/v2/resize:fit:1358/0*tNeMRLyLd5CAjzgK.png)

[

![Kvs Vishnu Kumar](https://miro.medium.com/v2/resize:fill:40:40/2*rnuFFIycn7RC3qdOcttzNw.jpeg)



](https://kvs-vishnu23.medium.com/?source=post_page---read_next_recirc--f3eeca9e95f1----2---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

Kvs Vishnu Kumar

](https://kvs-vishnu23.medium.com/?source=post_page---read_next_recirc--f3eeca9e95f1----2---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

## Terraform Mastery : An In-Depth Exploration of Key Concepts and Best Practices

### We will be going through all terraform topics in this article.



](https://kvs-vishnu23.medium.com/terraform-mastery-an-in-depth-exploration-of-key-concepts-and-best-practices-4c717bb3b6bc?source=post_page---read_next_recirc--f3eeca9e95f1----2---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

Oct 19, 2024

[](https://kvs-vishnu23.medium.com/terraform-mastery-an-in-depth-exploration-of-key-concepts-and-best-practices-4c717bb3b6bc?source=post_page---read_next_recirc--f3eeca9e95f1----2---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F4c717bb3b6bc&operation=register&redirect=https%3A%2F%2Fkvs-vishnu23.medium.com%2Fterraform-mastery-an-in-depth-exploration-of-key-concepts-and-best-practices-4c717bb3b6bc&source=---read_next_recirc--f3eeca9e95f1----2-----------------bookmark_preview----895847f3_adfb_4268_b26a_ee94586dd911-------)

![Terraform — Setting Up WAF on CloudFront](https://miro.medium.com/v2/resize:fit:1358/1*qCecOEt0Y-qleUjvWF7P1A.png)

[

![DevOps Pro](https://miro.medium.com/v2/resize:fill:40:40/1*cB1Iw24oelrpD4_Qhu5ATw.png)



](https://medium.com/devops-pro?source=post_page---read_next_recirc--f3eeca9e95f1----3---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

In

[

DevOps Pro

](https://medium.com/devops-pro?source=post_page---read_next_recirc--f3eeca9e95f1----3---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

by

[

Prashant Bhatasana

](https://prashant-48386.medium.com/?source=post_page---read_next_recirc--f3eeca9e95f1----3---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[

## Terraform — Setting Up WAF on CloudFront

### Setting up a Web Application Firewall (WAF) on Amazon CloudFront using Terraform is a powerful way to protect your web applications from…



](https://prashant-48386.medium.com/terraform-setting-up-waf-on-cloudfront-ee565d83615b?source=post_page---read_next_recirc--f3eeca9e95f1----3---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

Oct 3, 2024

[

17





](https://prashant-48386.medium.com/terraform-setting-up-waf-on-cloudfront-ee565d83615b?source=post_page---read_next_recirc--f3eeca9e95f1----3---------------------895847f3_adfb_4268_b26a_ee94586dd911-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fee565d83615b&operation=register&redirect=https%3A%2F%2Fmedium.com%2Fdevops-pro%2Fterraform-setting-up-waf-on-cloudfront-ee565d83615b&source=---read_next_recirc--f3eeca9e95f1----3-----------------bookmark_preview----895847f3_adfb_4268_b26a_ee94586dd911-------)

[

See more recommendations

](https://medium.com/?source=post_page---read_next_recirc--f3eeca9e95f1--------------------------------)