**vagrant@k8s-node1**:**~**$ systemctl status kubelet

**●** kubelet.service - kubelet: The Kubernetes Node Agent

     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; **enabled**; preset: **enabled**)

    Drop-In: /usr/lib/systemd/system/kubelet.service.d

             └─10-kubeadm.conf

     Active: **active (running)** since Thu 2025-03-06 14:27:01 UTC; 14h ago

       Docs: https://kubernetes.io/docs/

   Main PID: 723 (kubelet)

      Tasks: 12 (limit: 2249)

     Memory: 102.2M (peak: 103.0M)

        CPU: 25min 11.936s

     CGroup: /system.slice/kubelet.service

             └─723 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc>

**vagrant@k8s-controller**:**~**$ kubectl get nodes

NAME             STATUS   ROLES           AGE     VERSION

k8s-controller   Ready    control-plane   5d12h   v1.32.2

k8s-node1        Ready    <none>          5d12h   v1.32.2


**vagrant@k8s-controller**:**~**$ kubectl describe node k8s-node1                                                           

Name:               k8s-node1                                                                                       

Roles:              <none>                                                                                          

Labels:             beta.kubernetes.io/arch=amd64                                                                   

                    beta.kubernetes.io/os=linux                                                                     

                    kubernetes.io/arch=amd64                                                                        

                    kubernetes.io/hostname=k8s-node1                                                                

                    kubernetes.io/os=linux                                                                          

Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock              

                    node.alpha.kubernetes.io/ttl: 0                                                                 

                    projectcalico.org/IPv4Address: 192.168.33.20/24                                                 

                    projectcalico.org/IPv4IPIPTunnelAddr: 10.1.36.64                                                

                    volumes.kubernetes.io/controller-managed-attach-detach: true                                    

CreationTimestamp:  Sat, 01 Mar 2025 16:09:42 +0000                                                                 

Taints:             <none>                                                                                          

Unschedulable:      false                                                                                           

Lease:                                                                                                              

  HolderIdentity:  k8s-node1                                                                                        

  AcquireTime:     <unset>                                                                                          

  RenewTime:       Fri, 07 Mar 2025 04:28:06 +0000                                                                  

Conditions:                                                                                                         

  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason           

            Message                                                                                                 

  ----                 ------  -----------------                 ------------------                ------           

            -------                                                                                                 

  NetworkUnavailable   False   Thu, 06 Mar 2025 14:27:45 +0000   Thu, 06 Mar 2025 14:27:45 +0000   CalicoIsUp                   Calico is running on this node

  MemoryPressure       False   Fri, 07 Mar 2025 04:25:21 +0000   Thu, 06 Mar 2025 14:27:14 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available

  DiskPressure         False   Fri, 07 Mar 2025 04:25:21 +0000   Thu, 06 Mar 2025 14:27:14 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure

  PIDPressure          False   Fri, 07 Mar 2025 04:25:21 +0000   Thu, 06 Mar 2025 14:27:14 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available

  Ready                True    Fri, 07 Mar 2025 04:25:21 +0000   Thu, 06 Mar 2025 14:27:14 +0000   KubeletReady                 kubelet is posting ready status

Addresses:

  InternalIP:  10.0.2.15

  Hostname:    k8s-node1

Capacity:

  cpu:                2

  ephemeral-storage:  10218772Ki

  hugepages-2Mi:      0

  memory:             1996656Ki

  pods:               110

Allocatable:

  cpu:                2

  ephemeral-storage:  9417620260

  hugepages-2Mi:      0

  memory:             1894256Ki

  pods:               110

System Info:

  Machine ID:                 5c8e04bf168b4792a2d2622d50ec8c47

  System UUID:                0142a1e4-cea1-1f4d-b0ba-0ea05b6f150f

  Boot ID:                    0ba13414-52e5-4905-ab9f-9d07d1d36d34

  Kernel Version:             6.8.0-54-generic

  OS Image:                   Ubuntu 24.04.1 LTS
  

Mar 07 04:17:38 k8s-node1 kubelet[723]: I0307 04:17:38.981755     723 reconciler_common.go:162] "operationExecutor.>

Mar 07 04:17:38 k8s-node1 kubelet[723]: I0307 04:17:38.986503     723 operation_generator.go:780] UnmountVolume.Tea>

Mar 07 04:17:39 k8s-node1 kubelet[723]: I0307 04:17:39.083982     723 reconciler_common.go:299] "Volume detached fo>

Mar 07 04:17:39 k8s-node1 kubelet[723]: I0307 04:17:39.544182     723 pod_container_deletor.go:80] "Container not f>

Mar 07 04:18:53 k8s-node1 kubelet[723]: I0307 04:18:53.842428     723 kubelet_volumes.go:163] "Cleaned up orphaned >

Mar 07 04:19:20 k8s-node1 kubelet[723]: I0307 04:19:20.837814     723 scope.go:117] "RemoveContainer" containerID=">

Mar 07 04:22:16 k8s-node1 kubelet[723]: I0307 04:22:16.524851     723 memory_manager.go:355] "RemoveStaleState remo>

Mar 07 04:22:16 k8s-node1 kubelet[723]: I0307 04:22:16.701638     723 reconciler_common.go:251] "operationExecutor.>

Mar 07 04:22:19 k8s-node1 kubelet[723]: I0307 04:22:19.076267     723 pod_startup_latency_tracker.go:104] "Observed>

Mar 07 04:24:07 k8s-node1 kubelet[723]: I0307 04:24:07.910249     723 reconciler_common.go:251] "operationExecutor.>

lines 1-23/23 (END)


**vagrant@k8s-controller**:**~**$ kubectl get pods -n kube-system

NAME                                      READY   STATUS    RESTARTS      AGE

calico-kube-controllers-f64c9d84c-29wt5   1/1     Running   2 (10h ago)   5d12h

calico-node-k9zkd                         1/1     Running   2 (10h ago)   5d12h

calico-node-q6bgt                         1/1     Running   2 (14h ago)   5d12h

coredns-668d6bf9bc-lnd8q                  1/1     Running   2 (10h ago)   5d12h

coredns-668d6bf9bc-s9jbf                  1/1     Running   2 (10h ago)   5d12h

etcd-k8s-controller                       1/1     Running   2 (10h ago)   5d12h

kube-apiserver-k8s-controller             1/1     Running   2 (10h ago)   5d12h

kube-controller-manager-k8s-controller    1/1     Running   2 (10h ago)   5d12h

kube-proxy-c8fmb                          1/1     Running   2 (10h ago)   5d12h

kube-proxy-n5qpk                          1/1     Running   2 (14h ago)   5d12h

kube-scheduler-k8s-controller             1/1     Running   5 (10h ago)   5d12h

**vagrant@k8s-controller**:**~**$ kubectl describe pod busybox                                                              

Name:             busybox                                                                                           

Namespace:        default                                                                                           

Priority:         0                                                                                                 

Service Account:  default

Node:             k8s-node1/10.0.2.15

Start Time:       Fri, 07 Mar 2025 04:24:07 +0000

Labels:           run=busybox 

Annotations:      cni.projectcalico.org/containerID: ac69210598977789d203aa53dd1fbe2563ddd064ca93feb662ae4b91e6e44b67

                  cni.projectcalico.org/podIP: 10.1.36.77/32

                  cni.projectcalico.org/podIPs: 10.1.36.77/32

Status:           Running

IP:               10.1.36.77

IPs:

  IP:  10.1.36.77

Containers:

  busybox:

    Container ID:  containerd://35194228b374503df95e09f3fc22a51d5a984456cd2f76e728f40e001c5b58ad

    Image:         busybox

    Image ID:      docker.io/library/busybox@sha256:498a000f370d8c37927118ed80afe8adc38d1edcbfc071627d17b25c88efcab0

    Port:          <none>

    Host Port:     <none>

    Args:

      sleep

      3600

    State:          Running

      Started:      Fri, 07 Mar 2025 04:24:10 +0000

    Ready:          True

    Restart Count:  0

    Environment:    <none>

    Mounts:

      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-98k56 (ro)

Conditions:

  Type                        Status

  PodReadyToStartContainers   True 

  Initialized                 True 

  Ready                       True 

  ContainersReady             True 

  PodScheduled                True 

Volumes:

  kube-api-access-98k56:

    Type:                    Projected (a volume that contains injected data from multiple sources)

    TokenExpirationSeconds:  3607

    ConfigMapName:           kube-root-ca.crt

    ConfigMapOptional:       <nil>

    DownwardAPI:             true

QoS Class:                   BestEffort

Node-Selectors:              <none>

Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s

                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s

Events:

  Type    Reason     Age    From               Message

  ----    ------     ----   ----               -------

  Normal  Scheduled  6m18s  default-scheduler  Successfully assigned default/busybox to k8s-node1

  Normal  Pulling    5m31s  kubelet            Pulling image "busybox"

  Normal  Pulled     5m30s  kubelet            Successfully pulled image "busybox" in 1.592s (1.592s including waiting). Image size: 2167176 bytes.

  Normal  Created    5m29s  kubelet            Created container: busybox

  Normal  Started    5m29s  kubelet            Started container busybox

**vagrant@k8s-controller**:**~**$ nc -zv 192.168.33.20 10250

Connection to 192.168.33.20 10250 port [tcp/*] succeeded!




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

  

kubeadm join 192.168.33.10:6443 --token jv928h.v02hxx2cal6jol6l \

        --discovery-token-ca-cert-hash sha256:f62de2fbfbdb2c1c59bbbed53f777434983f075af4b6fbd67a617df0e63e02ab