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



>[!Lab Setup using Vagrant]

- Brings up 3 VirtualBox VMs (k8s-controlplane, k8s-node-1, k8s-node-2)
    
- Uses Ubuntu 22.04 cloud image (`ubuntu/jammy64`)
    
- Sets static private IPs on a host-only/private network (172.16.1.11/21 etc.)
    
- Installs containerd, kubeadm/kubelet/kubectl, configures sysctl & disables swap
    
- Initializes the control plane with `kubeadm init`, installs a Pod network (Flannel), and writes a `kubeadm join` command to `/vagrant/kubeadm_join.sh`
    
- Each worker waits for `/vagrant/kubeadm_join.sh` and then executes it (so joins the cluster)
    

Drop this file in an empty directory, run `vagrant up` and it will attempt to perform the full setup. I included safety/wait loops so nodes wait for the controlplane join script.

> Notes:
> 
> - This is intended for local dev/testing only.
>     
> - The shared `/vagrant` folder is used to pass the join command from controlplane -> nodes.
>     
> - If you prefer Calico or a different CNI, change the pod-network manifest step in the controlplane provisioner.
>     
> - You may want to increase memory/CPUs depending on host capacity.
>     

---

```ruby
# Vagrantfile - Kubernetes cluster (kubeadm) on VirtualBox, Ubuntu 22.04
# Save as Vagrantfile in an empty directory and run `vagrant up`

Vagrant.configure("2") do |config|
  # Basic common config
  config.vm.box = "ubuntu/jammy64"
  config.ssh.insert_key = false   # use default vagrant key for convenience
  config.vm.synced_folder ".", "/vagrant" # project dir shared with guests

  # Reusable shell to install containerd + kubeadm/kubelet/kubectl
  common_install = <<-SHELL
#!/usr/bin/env bash
set -eux

# set noninteractive
export DEBIAN_FRONTEND=noninteractive

# update and install prerequisites
apt-get update -y
apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Disable swap (kubeadm requirement)
swapoff -a
sed -i.bak '/ swap / s/^/#/' /etc/fstab || true

# sysctl for bridge traffic
modprobe overlay
modprobe br_netfilter
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl --system

# Install containerd
if ! command -v containerd >/dev/null 2>&1; then
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  echo \
    "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
    > /etc/apt/sources.list.d/docker.list
  apt-get update -y
  apt-get install -y containerd.io

  # configure containerd and restart
  mkdir -p /etc/containerd
  containerd config default >/etc/containerd/config.toml
  systemctl restart containerd
  systemctl enable containerd
fi

# Install kubeadm, kubelet, kubectl
if ! dpkg -l | grep -q kubeadm; then
  curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
  apt-get update -y
  apt-get install -y kubelet kubeadm kubectl
  apt-mark hold kubelet kubeadm kubectl
fi

# ensure /home/vagrant owns .kube later
chown -R vagrant:vagrant /home/vagrant || true
SHELL

  # Define control plane
  config.vm.define "k8s-controlplane" do |cp|
    cp.vm.hostname = "k8s-controlplane"
    cp.vm.network "private_network", ip: "172.16.1.11", netmask: "255.255.255.0"
    cp.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-controlplane"
      vb.memory = 4096
      vb.cpus = 2
    end

    # Install common prerequisites
    cp.vm.provision "shell", inline: common_install

    # Control plane specific provisioning
    cp.vm.provision "shell", inline: <<-SHELL
#!/usr/bin/env bash
set -eux

# Only run kubeadm init once
if [ ! -f /etc/kubernetes/admin.conf ]; then
  # kubeadm init
  kubeadm init --apiserver-advertise-address=172.16.1.11 --pod-network-cidr=10.244.0.0/16

  # configure kubeconfig for vagrant user
  mkdir -p /home/vagrant/.kube
  cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  chown vagrant:vagrant /home/vagrant/.kube -R

  # Install flannel as CNI (simple for demo). Replace if you prefer calico.
  su - vagrant -c "kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml"

  # Create a join script that workers can execute (written to /vagrant shared folder)
  # Wait until the control plane is ready to give token (simple sleep + retry)
  for i in $(seq 1 30); do
    if kubectl get nodes >/dev/null 2>&1; then break; fi
    sleep 2
  done

  kubeadm token create --print-join-command > /vagrant/kubeadm_join.sh
  chmod +x /vagrant/kubeadm_join.sh

  # optionally expose kubeconfig for host in /vagrant/admin.conf
  cp /etc/kubernetes/admin.conf /vagrant/admin.conf
  chown $(stat -c '%u:%g' /vagrant/admin.conf) /vagrant/admin.conf || true
else
  echo "kubeadm already initialized - skipping"
fi
SHELL
  end

  # Worker 1
  config.vm.define "k8s-node-1" do |node|
    node.vm.hostname = "k8s-node-1"
    node.vm.network "private_network", ip: "172.16.1.21", netmask: "255.255.255.0"
    node.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-node-1"
      vb.memory = 2048
      vb.cpus = 2
    end

    node.vm.provision "shell", inline: common_install

    # Wait for /vagrant/kubeadm_join.sh, then join
    node.vm.provision "shell", inline: <<-SHELL
#!/usr/bin/env bash
set -eux

# Wait up to 5 minutes for the join script produced by controlplane
JOIN_SCRIPT="/vagrant/kubeadm_join.sh"
i=0
until [ -f "${JOIN_SCRIPT}" ]; do
  i=$((i+1))
  echo "Waiting for ${JOIN_SCRIPT} to appear... ($i)"
  if [ $i -ge 150 ]; then
    echo "Timed out waiting for ${JOIN_SCRIPT}"
    exit 1
  fi
  sleep 2
done

chmod +x "${JOIN_SCRIPT}"
# run the join script (may require sudo)
sudo bash "${JOIN_SCRIPT}" --v 5 || true

# After join ensure kubelet running
systemctl enable --now kubelet
SHELL
  end

  # Worker 2
  config.vm.define "k8s-node-2" do |node|
    node.vm.hostname = "k8s-node-2"
    node.vm.network "private_network", ip: "172.16.1.22", netmask: "255.255.255.0"
    node.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-node-2"
      vb.memory = 2048
      vb.cpus = 2
    end

    node.vm.provision "shell", inline: common_install

    node.vm.provision "shell", inline: <<-SHELL
#!/usr/bin/env bash
set -eux

# Wait up to 5 minutes for the join script produced by controlplane
JOIN_SCRIPT="/vagrant/kubeadm_join.sh"
i=0
until [ -f "${JOIN_SCRIPT}" ]; do
  i=$((i+1))
  echo "Waiting for ${JOIN_SCRIPT} to appear... ($i)"
  if [ $i -ge 150 ]; then
    echo "Timed out waiting for ${JOIN_SCRIPT}"
    exit 1
  fi
  sleep 2
done

chmod +x "${JOIN_SCRIPT}"
sudo bash "${JOIN_SCRIPT}" --v 5 || true

systemctl enable --now kubelet
SHELL
  end

  # Provider-level config: ensure VirtualBox provider is used and VMs not headless unless desired
  config.vm.provider :virtualbox do |vb|
    # global VirtualBox defaults (can override per-VM)
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end
end
```

---

>[!note] How to use

1. Put the `Vagrantfile` into an empty directory on a machine with:
    - VirtualBox installed
    - Vagrant installed
2. Run:
```bash
vagrant up
    ```
    This will create three VMs and run the provisioning. Expect it to take ~15–40 minutes depending on host resources and network speed.
3. After `vagrant up` completes:
    - Control plane kubeconfig is available on the host at `./admin.conf`. You can use it with:
```bash
export KUBECONFIG=$(pwd)/admin.conf
kubectl get nodes
        ```
    - Or `vagrant ssh k8s-controlplane` and `kubectl get nodes`.
4. If a worker did not join: `vagrant ssh k8s-node-1` and look at `/var/log/` and `sudo kubeadm join ...` can be executed manually (the join command is also in `./kubeadm_join.sh`).
