Week 1: February 03 - February 09, 2025

Total 17 problems in 120 minutes with a 66% passing score.
8 questions straight forward with few commands
3-4 require some time with clear instructions
5 problems are the biggest one which can take bulk of your time



| Domain & Competencies                                   | Sub topics                                                                            |
|---------------------------------------------------------|---------------------------------------------------------------------------------------|
| Storage 10%                                             | Implement storage classes and dynamic volume provisioning                             |
|                                                         | Configure volume types, access modes and reclaim policies                             |
| Troubleshoorting 30%                                    | [ ] Troubleshoot cluster and nodes                                                    |
|                                                         | [ ] Troubleshoot cluster components                                                   |
|                                                         | [ ] Monitor cluster and application resource usage                                    |
|                                                         | [ ] Manage and evaluate container output streams                                      |
|                                                         | [ ] Troubleshoot services and networking                                              |
| Workloads & Scheduling 15%                              | Understanding application deployments and how to perform rolling update and rollbacks |
|                                                         | Use ConfigMaps and secrets to configure applications                                  |
|                                                         | Configure workload autoscaling                                                        |
|                                                         | understand the primitives used to create robust, selfhosting, application deployments |
|                                                         | configure pod admisssion and scheduling(limits, mode affinity, etc.                   |
| Cluster architecure, installation and configuration 25% | Manage role based access control                                                      |
|                                                         | prepare underlying infrastructure for installing kubernetes cluster                   |
|                                                         | create and manage kubernetes cluster using kubeadm                                    |
|                                                         | manage the lifecyle of the kubernetes cluster                                         |
|                                                         | Implement and configure a high-available control plane                                |
|                                                         | Use helm and kustomize to install cluster components                                  |
|                                                         | Undersfand extension interfaces (CNI, CSI, CRI, etc.)                                 |
|                                                         | Understand CRDs, install and configure operators                                      |
| Services & Networking 20%                               | Understand connectivity between pods                                                  |
|                                                         | Define and enforce network policies                                                   |
|                                                         | Use clusterIP, NodePort, LoadBalancer service types and endpoints                     |
|                                                         | Use the Gatewat API to manage ingress traffic                                         |
|                                                         | Know how to use INgress controllers and Ingress resources                             |
|                                                         | Understand the use of coreDNS                                                                                      |


##### Storage 10%
###### Implement storage classes and dynamic volume provisioning
###### Configure volume types, access modes and reclaim policies
###### Manage persistent volumes and persistent volume claims

##### Troubleshooting 30%
###### Troubleshoot clusters and nodes
###### Troubleshoot cluster components
###### Monitor cluster and application resource usage
###### Manage and evaluate container output streams
###### Troubleshoot services and networking

##### Workloads & Scheduling 15%
###### Understand application deployments and how to perform rolling update and rollbacks
###### Use ConfigMaps and Secrets to configure applications
###### Configure workload autoscaling
###### Understand the primitives used to create robust, self-healing, application deployments
###### Configure Pod admission and scheduling (limits, node affinity, etc.)


##### Cluster Architecture, Installation & Configuration 25%
Kubernets: container orchesterator, workload placement answer how it is deployed, where the deployment is to be done, abstract the infrastructure from app, where the server will be, which cluster it sits on and maintain desired state
Key elements: 
1. Speed of deployemwnt
2. Ability to absorb change quickly
3. Ability to recover quickly
4. Hide complexity of infrastructure in the sluster

Principles 
1. Desired state, declarative configuration
2. Controllers/control lopp
3. Kubernetes APi. Available via API server.

K8s API
Api objects: allow us to enable the state of the cluster using declarative and imperatove methods.
Pods, controllers, services storage are API objects.

Pods are ephemeral, a pod crashes, the other pod will replaces the pod and service will be provided from the pod, the original pod is never recreated.
K8s ensure the pods keeps on running and it should manage the desired state of application

Controllers: defines the desired state, create and manages the pods, ot responds to pods health and state. Replicaset maintains the no. Of replicas , manage rollout of replicaset. And deployements

Services: add persistence to the emphemeral pods. Dynamicaly update ppds lifecycle.

Storage: volume: directly accessible by pods, tightly coupled with pods, not ideal
Persistent volume claim: ensures that pods created or deleted can ise the storage for keeping the data, without relying on the state, ephemerality of the pod.

Cluster components: 
1. Controller plane node
     1. API server- communication hub, primary access point, allows restful operation, get put, push, delete, 
     2. Etcd: database, persistent state, api objects, stores in key-value pair
     3. Scheduler
     4. Control manager
2. Worker node

###### Manage role based access control (RBAC)
###### Prepare underlying infrastructure for installing a Kubernetes cluster
###### Create and manage Kubernetes clusters using kubeadm
###### Manage the lifecycle of Kubernetes clusters
###### Implement and configure a highly-available control plane
###### Use Helm and Kustomize to install cluster components
###### Understand extension interfaces (CNI, CSI, CRI, etc.)
###### Understand CRDs, install and configure operators

##### Services & Networking 20%
###### Understand connectivity between Pods
###### Define and enforce Network Policies
###### Use ClusterIP, NodePort, LoadBalancer service types and endpoints
###### Use the Gateway API to manage Ingress traffic
###### Know how to use Ingress controllers and Ingress resources
###### Understand and use CoreDNS


Focus area
- Volumes -- PV, PVCs, SCs
- RBAC -- Know how to verify once provisioned
- Network policies ---
- Services -- especially nodeport
- Upgrade
- Troubleshoot unsteady nodes
- MOnitoring and logging
- Sidecars -- Tough ones
- ETCD backup and restore
- Deployment scaling and recording
- drinaing nodes
Network policies: https://github.com/ahmetb/kubernetes-network-policy-recipes

Aliases to create while starting the exam
``` .bashrc
alias gp='sudo kubectl get pods'
alias dp='sudo kubectl describe pod'
alias kc='sudo kubectl create'
alias gl='sudo kubectl logs'
alias gs='sudo kubectl get svc'
alias gn='sudo kubectl get nodes'
alias kr='sudo kubectl run -o yaml --dry-run'

#enable autocompletion
source <(kubectl completion bash)
```


Controllers in Kubernetes
How controllers work
Types of Controllers
Understanding Replicasets

Kubernetes Principles
- Desired state and declarative configuration: K8s maintains the desired state as 
- Controller Control loops - 
- The API Server

Using controllers to deploy applications and introductions to controllers


Creating deployments
- Writing a deployment - Declarative
	- Selector
	- Replicas
	- Pod Template
- Imperative: kubectl create deploymnet hello-world -image
- kubectl scale deployment hello-world --replicas=5

Basic Deployment

below componenet is needed to ensure there is a deployment.

``` YAML
apiVersion: app/v1
kind: Deployment
metadata:
	name: hello-world
spec:
	replicas: 5
	Selector:
		matchlables:
			app: hello-world
	template:
		metadata:
		- labels:
			app: hello-world
	- spec: containers

	- 
```


What controls the pod
- Replicaset
- Deployments
- Daemonset
- Statefulset
- Job
- Cron Job

Types of Controllers
- Node controllers
- service controllers
- Endpoint
- 

Topic 1: Kubernetes Cluster Setup

Wh
1. What is Kubernetes?

- Kubernetes is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications.
- Key components: Master Node (Control Plane) and Worker Nodes.

2. Cluster Architecture

- Master Node Components:

- API Server: Front-end for the Kubernetes control plane.
- etcd: Distributed key-value store for cluster data.
- Scheduler: Assigns workloads to worker nodes.
- Controller Manager: Manages controllers like Node Controller, Replication Controller, etc.

- Worker Node Components:

- Kubelet: Ensures containers are running in a pod.
- Kube Proxy: Manages network rules and communication.
- Container Runtime: Runs containers (e.g., Docker, containerd).

3. Setting Up a Kubernetes Cluster

- Tools for Cluster Setup:

- kubeadm: A tool to bootstrap a Kubernetes cluster.
- minikube: For local single-node clusters.
- kubectl: CLI to interact with the cluster.

- Steps to Set Up a Cluster:

- Install kubeadm, kubectl, and kubelet on all nodes.
- Initialize the master node: kubeadm init.
- Join worker nodes: kubeadm join.
- Deploy a Pod Network (e.g., Calico, Flannel).

4. Hands-On Practice:

- Set up a local cluster using minikube.
- Use kubectl to check cluster status: kubectl get nodes.
- Deploy a sample application: kubectl create deployment nginx --image=nginx.

Topic 2: Kubernetes Networking

5. Networking Basics

- Kubernetes networking ensures that:

- Pods can communicate with each other.
- Pods can communicate with services.
- External systems can communicate with services.

6. Key Concepts:

- Pod Networking: Each pod gets a unique IP address.
- Service Networking: Provides a stable IP and DNS name for a set of pods.
- Network Policies: Define how pods communicate with each other.

7. Service Types:

- ClusterIP: Exposes the service on a cluster-internal IP.
- NodePort: Exposes the service on each node’s IP at a static port.
- LoadBalancer: Exposes the service externally using a cloud provider’s load balancer.
- ExternalName: Maps the service to an external DNS name.

8. Hands-On Practice:

- Create a service: kubectl expose deployment nginx --port=80 --type=NodePort.
- Access the service: kubectl get svc and use the NodePort to access the service.

Topic 3: Kubernetes Storage

9. Storage Basics

- Kubernetes provides persistent storage for applications.
- Key concepts: Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and Storage Classes.

10. Persistent Volumes (PVs):

- A piece of storage in the cluster provisioned by an administrator.
- Example: kubectl apply -f pv.yaml.

11. Persistent Volume Claims (PVCs):

- A request for storage by a user.
- Example: kubectl apply -f pvc.yaml.

12. Storage Classes:

- Defines the "class" of storage (e.g., SSD, HDD).
- Example: kubectl apply -f storageclass.yaml.

13. Hands-On Practice:

- Create a PV and PVC.
- Attach the PVC to a pod: kubectl apply -f pod-with-pvc.yaml.
- Verify the storage: kubectl get pv, kubectl get pvc.

Study Tips for This Week

14. Take Notes:

- Write down key concepts, commands, and their purposes.
- Use flashcards for quick revision.

15. Hands-On Practice:

- Spend at least 60% of your time practicing on a real or local cluster.
- Use kubectl commands frequently to build muscle memory.

16. Resources:

- Official Kubernetes Documentation:  [https://kubernetes.io/docs/](https://kubernetes.io/docs/)
- Interactive Labs:  [https://killercoda.com/](https://killercoda.com/)


Pluralsight Labs
## What I have learnt
### Using Kubernetes to interactl with API Servers
* To interact with the minikube cluster.
```
kubectl get pods -A
```

Kubernetes works with the concept of contexts. Context is a group of access parameters that define which cluster you are interacting with, which user you are using and which namespace you're working. Kubectl is recommended way to access the cluster of SAs

* To list the contexts available
```
kubectl config get-contexts
```
Shows the available context in the environment
The current context is denoted by * in first column
you can switch to *kubectl config use-context <context-name>* in case there are multiple contexts.

- Understanding contexts
	Contexts defines the parametes required to connect to the k8s cluster.
	Usually stored in ~/.kube/config
	Key information
	1. **Server** http endpoint of the kube-api-server
	2. **users**: username for connection to the cluster
	3. **namespace**: default namespace which user will have access to when the context is activates.
	4. **client-certificate** and **client-key: client certificate and key pair used for authentication
- `kubectl cluster-info` provides more information about the cluster.  
Note: coreDNS is in the kube-dns namespace, is a CNCF(Cloud Native Computing Foundation) project.


common commands
- kubectl get po -A : get status of all the components
- kubectl config get-context: list the available contexts in your env.
- $HOME/.kube/config - context of your kubeconfig file, defines the parameters required to connect to a kubernetes cluster.
- kubectl cluster-info: gives information about the cluster, provides the endpoint where the kube-api-server is listening to the requests, and also the end-point of the DNS service.
- kubectl api-resources: list available api resources, wll find commin resources like pods, deployments, secrets and services, leases. *Leases* provides mechanism to lock shared resources.and co-ordinate activity between members of set
- kubectl explain leases: online docs for understanding any API resources you are unfamiliar with
- kubectl explain pods.spec: can also provide details on the objects of the resources.
- kubectl crease ns challenge2: create a namespace
- kubectl config set-context --current --namespace=challenge2: switch the context to challenge2
- kubectl apply -f ~/challenge/02/deployment.yml: create a deployment with 3 replicase
- kubectl apply -f ~/challenge/02/deployment.yml --dry-run=client: this will show the deployment will be successfully configured in the client
- kubectl apply -f ~/challenge/02/deployment.yml --dry-run=server: this will show the deploymebt would be successful.
- kubectl get deployments: to list the deployments and ensure proper replicas
- kubectl diff -F ~/challeneges/02/deployment-v2.yml: tell the difference between existing resource and the new version stored.

#### Interacting with the K8S APi server using curl command
kubectl is just a client sending http request to the API servers. 

- kubectl get namespaces -v 6: We can increase the verbosity of the kubectl command with -v <n> parameter. where <n> is the logging level.
- curl -v https://192.168.49.2:8443/api/v1/namespaces?limit=500 --insecure: same as above, but will error as we did not provide any certificates for authentication
- curl -v https://192.168.49.2:8443/api/v1/namesapces?limit=500 --insecure --cert /home/pslearner/.minikube/client.crt --key /home/pslearner/.minikube/profiles/minikube/client.key: provide client cert and key curl do not take env variable home. it does take but somehow for me it didn't pick may be the command was wrongly typed.
- Create a pod using curl
	- kubectl create ns challenge3
	- kubectl config set-context --current namespace=challenge3: set-context is for switching context, not the switch-context
	- cat ~/challenges/03/pod.yml
	- curl -v --insecure --cert /home/pslearner/.minikube/profiles/minikube/client.crt --key /home/pslearner/.minikube/profiles/minikube/client.key -X POST https://192.168.49.2:8443/api/v1/namespaces/challenge3/pods -H 'Content-type: application/yaml' --data-binary '@/home/pslearner/challenges/03/pod.yml': Should return status code 201(created)
		To create the pod, you will need some information to build the curl request:
		- _Type of request:_ **POST**, as you will be pushing data rather than retrieving as done previously.
		- _Kube API Server and port:_ **https://192.168.49.2:8443**
		- _Endpoint URI:_ **/api/v1/namespaces/challenge3/pods**. This targets the pod resources within the challenge3 namespace.
		- _Client certificate and key files for authentication:_ These are the ones defined in the kubeconfig file, **/home/pslearner/.minikube/profiles/minikube/client.crt** and **/home/pslearner/.minikube/profiles/minikube/client.key**
		- _Resource definition file:_ Stored under **/home/pslearner/challenges/03/pod.yml**
		- _Content-Type header:_ This will be **application/yaml**
- curl -v --insecure --cert /home/pslearner/.minikube/profies/minikube/client.crt --key /home/pslearner/.minikube/profiles/minikube/client.key -X Delete https://192.168.49.2:8443/api/v1/namespaces/challenge3/pods/nginx: this shall delete the pods

#### Use labesl to connect workloads

# Chapter 2. Cluster Architecture, Installation, and Configuration
section also covers the security aspects of a cluster, more specifically role-based access control (RBAC). 
- Understanding RBAC
    
- Installing of a cluster with `kubeadm`
    
- Upgrading a version of a Kubernetes cluster with `kubeadm`
    
- Backing up and restoring etcd with `etcdctl`
    
- Understanding a highly available Kubernetes cluster

Help in varitey of use cases
- Establish a system for users with differenet roles to access a set of kubernetes resources, like namespaces and context.
- control process running in the pod and the operations they can perform via kube API
- limiting visiblity of certain resources per NS.
building blocks of RBAC
- Subject: user, group, serviceid
- API Resources: while kubernetes APi resources to be accessible
- verb: operation which can be executed on the resource

_User and group are not stored in the etcd, exists as objects in kubernetes and are used by processes running inside the cluster_

calls to API server with user need to be authenciated
kubernetes does not represent a user as with an API resource. The user is meant to be managed by the admin of kube cluster
kubernetes offers varitey of authentication methods.

Steps to create a user using openssl client certificates
mkdir cert && cd cert
opessl genrsa -out user.key 2048
** Create a CSR
openssl req -new -key user.key -out user.csr -subj "/CN=user/O=cka-study"

** Sign the certifiate
openssl x509 -req -in user.csr -CA /.minikube/ca.crt -CAkey /.minikube/cakey -CAcreateserial -out user.crt -days 364

** Create user in the k8s cluster as user and create a context entry for the user.
kubectl config set-credentials user --client-certificate=user.crt --client-key=user.key
kubectl config set-context user-context --cluster=minikube --user=johndoekube


ISSUE: Kubectl on my macbook was not connecting to the minikube cluster running on docker
- Restarting the container didn't solve the issue. the container was runniing and was able to see the same woith docker ps command.
- stopping the minikube cluster by running minikube stop and then starting minikube start fix the issue.




