### **Day 1: Kubernetes Cluster Setup on AWS Free Tier**

Since you're using **AWS Free Tier**, we'll set up a **Kubernetes cluster using `kubeadm` on AWS EC2 instances**. This is the closest experience to a real-world **production-like cluster** while staying cost-effective.

---

## **🚀 Step 1: AWS Setup - Provision EC2 Instances**

We'll create:

- **1 Master Node** (t3.micro, Ubuntu 22.04)
- **2 Worker Nodes** (t3.micro, Ubuntu 22.04)

### **1️⃣ Create EC2 Instances**

1. Log into your **AWS Free Tier** account.
2. Go to **EC2 Dashboard → Launch Instances**.
3. Choose **Ubuntu 22.04 LTS (x86_64)** as the OS.
4. Select instance type: **t3.micro (Free Tier)**
5. Configure networking:
    - **Security Group:** Open ports **22 (SSH), 6443 (API Server), 10250-10255 (kubelet & metrics)**
    - Allow **ICMP (ping)** for testing.
6. **Launch 3 instances** (1 master, 2 workers).
7. Attach a **key pair** for SSH access.

---

## **🚀 Step 2: Connect via SSH**

Find the **public IP addresses** of the instances.

bash
`# Connect to Master Node ssh -i your-key.pem ubuntu@<master-node-public-ip>  # Connect to Worker Nodes (in separate terminals) ssh -i your-key.pem ubuntu@<worker-node-1-ip> ssh -i your-key.pem ubuntu@<worker-node-2-ip>`

Three Nodes with one master and two worker nodes
  `ssh -i ~/.ssh/kube-keys.pem ubuntu@44.202.130.30
  `ssh -i ~/.ssh/kube-keys.pem ubuntu@3.93.16.202
  `ssh -i ~/.ssh/kube-keys.pem ubuntu@3.90.115.251`

---

## **🚀 Step 3: Install Kubernetes & Docker on All Nodes**

Run these steps on **ALL** nodes (Master + Workers).

### **1️⃣ Update Packages**

bash

CopyEdit

`sudo apt update && sudo apt upgrade -y`

### **2️⃣ Install Docker**
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
docker --version
sudo docker run hello-world

### **3️⃣ Add Kubernetes Repo & Install Kubeadm, Kubectl, Kubelet**

bash

CopyEdit

`sudo apt install -y apt-transport-https ca-certificates curl 
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo tee /etc/apt/trusted.gpg.d/kubernetes.asc echo "deb https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
kubeadm version
kubectl version --client

Verify:

bash

CopyEdit

`kubeadm version kubectl version --client`

---

## **🚀 Step 4: Initialize Kubernetes on Master Node**

Run **only on the Master Node**:

bash

CopyEdit

`sudo kubeadm init --pod-network-cidr=192.168.0.0/16`

After initialization, **copy and save the join command** (you’ll need it for worker nodes).

### **Set Up Kubectl for Your User**

bash

CopyEdit

`mkdir -p $HOME/.kube sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config sudo chown $(id -u):$(id -g) $HOME/.kube/config`

Verify:

bash

CopyEdit

`kubectl get nodes`

---

## **🚀 Step 5: Apply a Network Plugin (Calico)**

bash

CopyEdit

`kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml`

Verify:

bash

CopyEdit

`kubectl get pods -n kube-system`

---

## **🚀 Step 6: Join Worker Nodes**

On **each Worker Node**, run the **join command** from Step 4.

If you forgot it, retrieve it on Master:

bash

CopyEdit

`kubeadm token create --print-join-command`

Run the output command on each **Worker Node**.

After joining, check nodes on Master:

bash

CopyEdit

`kubectl get nodes`

All nodes should show **"Ready"**.

---

## **🎯 Day 1 Recap**

✅ **Created a Kubernetes cluster (1 Master, 2 Workers) on AWS Free Tier**  
✅ **Installed Docker, Kubernetes, and initialized `kubeadm`**  
✅ **Configured networking with Calico**  
✅ **Successfully joined worker nodes**

---

### **Next Steps (Day 2)**

Tomorrow, we’ll **deep dive into networking**:

- **Pod-to-Pod communication**
- **Service types (ClusterIP, NodePort, LoadBalancer)**
- **Ingress and DNS**

Let me know once you complete today’s setup! 🚀💡


To start using your cluster, you need to run the following as a regular user:

  q

  mkdir -p $HOME/.kube

  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  

Alternatively, if you are the root user, you can run:

  

  export KUBECONFIG=/etc/kubernetes/admin.conf

  

You should now deploy a pod network to the cluster.

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:

  https://kubernetes.io/docs/concepts/cluster-administration/addons/

  

Then you can join any number of worker nodes by running the following on each as root:

  

kubeadm join 172.31.84.44:6443 --token 0i4wbq.wfa7y1i2f3hc54j1 --discovery-token-ca-cert-hash sha256:ae1ae30c40ac9ff341add756fa75bf3bb8ab988bddc923d149f6b8f56d63fa45


ae1ae30c40ac9ff341add756fa75bf3bb8ab988bddc923d149f6b8f56d63fa45

[preflight] Running pre-flight checks

[preflight] Reading configuration from the cluster...

[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"

[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"

[kubelet-start] Starting the kubelet

[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...


Master Node: 172.31.84.44
Worker Node 1 : 172.31.38.121
Worker Node 2: 172.31.35.22

Security group policies with incoming rules are as follows:

CUSTOM TCP Port 10250 - 10250 From:0.0.0.0/0
All ICMP - IPv4 PROTOCOL:ICMP PORT:All FROM:0.0.0.0/0
IPv4 SSH PROTOCOL:TCP PORT:22 FROM:0.0.0.0/0 
IPv4 Custom TCP TCP Port:6443 From:0.0.0.0/0

Kubadm join on worked node times out with following error
```
**ubuntu@ip-172-31-38-121**:**~**$ sudo kubeadm join 172.31.84.44:6443 --token 0i4wbq.wfa7y1i2f3hc54j1 --discovery-token-ca-cert-hash sha256:ae1ae30c40ac9ff341add756fa75bf3bb8ab988bddc923d149f6b8f56d63fa45

[preflight] Running pre-flight checks

[preflight] Reading configuration from the cluster...

[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"

[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"

[kubelet-start] Starting the kubelet

[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

[kubelet-check] Initial timeout of 40s passed.

  

Unfortunately, an error has occurred:

timed out waiting for the condition

  

This error is likely caused by:

- The kubelet is not running

- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

  

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:

- 'systemctl status kubelet'

- 'journalctl -xeu kubelet'

error execution phase kubelet-start: timed out waiting for the condition
```


Your Kubernetes control-plane has initialized successfully!

  

To start using your cluster, you need to run the following as a regular user:

  

  mkdir -p $HOME/.kube

  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  

Alternatively, if you are the root user, you can run:

  

  export KUBECONFIG=/etc/kubernetes/admin.conf

  

You should now deploy a pod network to the cluster.

Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:

  https://kubernetes.io/docs/concepts/cluster-administration/addons/

  

Then you can join any number of worker nodes by running the following on each as root:

  

kubeadm join 172.31.16.102:6443 --token h4klmi.3ch8b67luy4nsdbn --discovery-token-ca-cert-hash sha256:fd34857d817e2080db12323e094c41a70036a8394b0e993568bc0841973c7a55