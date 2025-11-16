![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)
# Core Concepts (13%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

### Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace

<details><summary>show</summary>
<p>

```bash
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

</p>
</details>

### Create the pod that was just described using YAML

<details><summary>show</summary>
<p>

Easily generate YAML with:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -n mynamespace -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: mynamespace
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
```

Alternatively, you can run in one line

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n mynamespace -f -
```

</p>
</details>

### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --command --restart=Never -it --rm -- env # -it will help in seeing the output, --rm will immediately delete the pod after it exits
# or, just run it without -it
kubectl run busybox --image=busybox --command --restart=Never -- env
# and then, check its logs
kubectl logs busybox
```

</p>
</details>

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
# create a  YAML template with this command
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml
# see it
cat envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
# apply it and then see the logs
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run=client
```

</p>
</details>

### Create the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

</p>
</details>

### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get po --all-namespaces
```
Alternatively 

```bash
kubectl get po -A
```
</p>
</details>

### Create a pod with image nginx called nginx and expose traffic on port 80

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
```

</p>
</details>

### Change pod's image to nginx:1.24.0. Observe that the container will be restarted as soon as the image gets pulled

<details><summary>show</summary>
<p>

*Note*: The `RESTARTS` column should contain 0 initially (ideally - it could be any number)

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.24.0
kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
kubectl get po nginx -w # watch it
```

*Note*: some time after changing the image, you should see that the value in the `RESTARTS` column has been increased by 1, because the container has been restarted, as stated in the events shown at the bottom of the `kubectl describe pod` command:

```
Events:
  Type    Reason     Age                  From               Message
  ----    ------     ----                 ----               -------
[...]
  Normal  Killing    100s                 kubelet, node3     Container pod1 definition changed, will be restarted
  Normal  Pulling    100s                 kubelet, node3     Pulling image "nginx:1.24.0"
  Normal  Pulled     41s                  kubelet, node3     Successfully pulled image "nginx:1.24.0"
  Normal  Created    36s (x2 over 9m43s)  kubelet, node3     Created container pod1
  Normal  Started    36s (x2 over 9m43s)  kubelet, node3     Started container pod1
```

*Note*: you can check pod's image by running

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Alternatively you can also try a more advanced option:

```bash
# Get IP of the nginx pod
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# create a temp busybox pod
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Or just in one line:

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

</p>
</details>

### Get pod's YAML

<details><summary>show</summary>
<p>

```bash
kubectl get po nginx -o yaml
# or
kubectl get po nginx -oyaml
# or
kubectl get po nginx --output yaml
# or
kubectl get po nginx --output=yaml
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
# or
kubectl logs nginx --previous
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# or
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # nowhere to be found :)
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
# or
kubectl run nginx --image nginx --restart=Never --env=var1=val1 -it --rm -- sh -c 'echo $var1'
```

</p>
</details>

![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)
# Multi-container Pods (10%)

### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

The easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
# Connect to the busybox2 container within the pod
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit

# or you can do the above with just a one-liner
kubectl exec -it busybox -c busybox2 -- ls

# you can do some cleanup
kubectl delete po busybox
```

</p>
</details>

### Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using 'echo "Test" > /work-dir/index.html'. Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

<details><summary>show</summary>
<p>

The easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run box --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

Copy/paste the container related values, so your final YAML should contain the volume and the initContainer:

Volume:

```YAML
containers:
- image: nginx
...
  volumeMounts:
  - name: vol
    mountPath: /usr/share/nginx/html
volumes:
- name: vol
  emptyDir: {}
```

initContainer:

```YAML
...
initContainers:
- args:
  - /bin/sh
  - -c
  - echo "Test" > /work-dir/index.html
  image: busybox
  name: box
  volumeMounts:
  - name: vol
    mountPath: /work-dir
```

In total you get:

```YAML

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: box
  name: box
spec:
  initContainers: 
  - args: 
    - /bin/sh 
    - -c 
    - echo "Test" > /work-dir/index.html
    image: busybox 
    name: box 
    volumeMounts: 
    - name: vol 
      mountPath: /work-dir 
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts: 
    - name: vol 
      mountPath: /usr/share/nginx/html 
  volumes: 
  - name: vol 
    emptyDir: {} 
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP
kubectl get po -o wide

# Execute wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- $(kubectl get pod box -o jsonpath='{.status.podIP}')"

# you can do some cleanup
kubectl delete po box
```

</p>
</details>


![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/pod_design&empty)
# Pod design (20%)

[Labels And Annotations](#labels-and-annotations)

[Deployments](#deployments)

[Jobs](#jobs)

[Cron Jobs](#cron-jobs)

## Labels and Annotations
kubernetes.io > Documentation > Concepts > Overview > Working with Kubernetes Objects > [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)

### Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

<details><summary>show</summary>
<p>

```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
# or
for i in `seq 1 3`; do kubectl run nginx$i --image=nginx -l app=v1 ; done
```

</p>
</details>

### Show all labels of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po --show-labels
```

</p>
</details>

### Change the labels of pod 'nginx2' to be app=v2

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx2 app=v2 --overwrite
# or edit the pod yaml
kubectl edit po nginx2
```

</p>
</details>

### Get the label 'app' for the pods (show a column with APP labels)

<details><summary>show</summary>
<p>

```bash
kubectl get po -L app
# or
kubectl get po --label-columns=app
```

</p>
</details>

### Get only the 'app=v2' pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -l app=v2
# or
kubectl get po -l 'app in (v2)'
# or
kubectl get po --selector=app=v2
```

</p>
</details>

### Get 'app=v2' and not 'tier=frontend' pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -l app=v2,tier!=frontend
# or
kubectl get po -l 'app in (v2), tier notin (frontend)'
# or
kubectl get po --selector=app=v2,tier!=frontend
```

</p>
</details>

### Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels

<details><summary>show</summary>
<p>

```bash
kubectl label po -l "app in(v1,v2)" tier=web
```
</p>
</details>


### Add an annotation 'owner: marketing' to all pods having 'app=v2' label

<details><summary>show</summary>
<p>

```bash
kubectl annotate po -l "app=v2" owner=marketing
```
</p>
</details>

### Remove the 'app' label from the pods we created before

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx1 nginx2 nginx3 app-
# or
kubectl label po nginx{1..3} app-
# or
kubectl label po -l app app-
```

</p>
</details>

### Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

<details><summary>show</summary>
<p>


```bash
kubectl annotate po nginx1 nginx2 nginx3 description='my description'

#or

kubectl annotate po nginx{1..3} description='my description'
```

</p>
</details>

### Check the annotations for pod nginx1

<details><summary>show</summary>
<p>

```bash
kubectl annotate pod nginx1 --list

# or

kubectl describe po nginx1 | grep -i 'annotations'

# or

kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

As an alternative to using `| grep` you can use jsonPath like `kubectl get po nginx1 -o jsonpath='{.metadata.annotations}{"\n"}'`

</p>
</details>

### Remove the annotations for these three pods

<details><summary>show</summary>
<p>

```bash
kubectl annotate po nginx{1..3} description- owner-
```

</p>
</details>

### Remove these pods to have a clean state in your cluster

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx{1..3}
```

</p>
</details>

## Pod Placement

### Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

<details><summary>show</summary>
<p>

Add the label to a node:

```bash
kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels
```

We can use the 'nodeSelector' property on the Pod YAML:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
  nodeSelector: # add this
    accelerator: nvidia-tesla-p100 # the selection label
```

You can easily find out where in the YAML it should be placed by:

```bash
kubectl explain po.spec
```

OR:
Use node affinity (https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/#schedule-a-pod-using-required-node-affinity)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100
  containers:
    ...
```

</p>
</details>

### Taint a node with key `tier` and value `frontend` with the effect `NoSchedule`. Then, create a pod that tolerates this taint.

<details><summary>show</summary>
<p>

Taint a node:

```bash
kubectl taint node node1 tier=frontend:NoSchedule # key=value:Effect
kubectl describe node node1 # view the taints on a node
```

And to tolerate the taint:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "tier"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
```

</p>
</details>

### Create a pod that will be placed on node `controlplane`. Use nodeSelector and tolerations.

<details><summary>show</summary>
<p>

```bash
vi pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    kubernetes.io/hostname: controlplane
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```

```bash
kubectl create -f pod.yaml
```

</p>
</details>

## Deployments

kubernetes.io > Documentation > Concepts > Workloads > Workload Resources > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml > deploy.yaml
vi deploy.yaml
# change the replicas field from 1 to 2
# add this section to the container spec and save the deploy.yaml file
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

or, do something like:

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.18.0/image: nginx:1.18.0\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

or,
```bash
kubectl create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

</p>
</details>

### View the YAML of this deployment

<details><summary>show</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR you can find rs directly by:
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
# you could also just do kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml
```

</p>
</details>

### Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po # get all the pods
# OR you can find pods directly by:
kubectl get po -l run=nginx # if you created deployment by 'run' command
kubectl get po -l app=nginx # if you created deployment by 'create' command
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

</p>
</details>

### Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### Update the nginx image to nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.8
# alternatively...
kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image
```

The syntax of the 'kubectl set image' command is `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

</p>
</details>

### Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # check that a new replica set has been created
kubectl get po
```

</p>
</details>

### Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# wait a bit
kubectl get po # select one 'Running' Pod
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.18.0
```

</p>
</details>

### Do an on-purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# or
kubectl edit deploy nginx
# change the image to nginx:1.91
# vim tip: type (without quotes) '/image' and press Enter, to navigate quickly
```

</p>
</details>

### Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
# or
kubectl get po # you'll see 'ErrImagePull' or 'ImagePullBackOff'
```

</p>
</details>


### Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # Everything should be OK
```

</p>
</details>

### Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # You'll also see the wrong image displayed here
```

</p>
</details>

### Scale the deployment to 5 replicas

<details><summary>show</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
```

</p>
</details>

### Autoscale the deployment, pods between 5 and 10, targeting CPU utilization at 80%

<details><summary>show</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
# view the horizontalpodautoscalers.autoscaling for nginx
kubectl get hpa nginx
```

</p>
</details>

### Pause the rollout of the deployment

<details><summary>show</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### Update the image to nginx:1.19.9 and check that there's nothing going on, since we paused the rollout

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.9
# or
kubectl edit deploy nginx
# change the image to nginx:1.19.9
kubectl rollout history deploy nginx # no new revision
```

</p>
</details>

### Resume the rollout and check that the nginx:1.19.9 image has been applied

<details><summary>show</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # insert the number of your latest revision
```

</p>
</details>

### Delete the deployment and the horizontal pod autoscaler you created

<details><summary>show</summary>
<p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

# or
kubectl delete deploy/nginx hpa/nginx
```
</p>
</details>

### Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

<details><summary>show</summary>
<p>

Deploy 3 replicas of v1:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-1 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
```

Create the service:
```
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  labels:
    app: my-app
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: my-app
```

Test if the deployment was successful from within a Pod:
```
# run a wget to the Service my-app-svc
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox --command -- wget -qO- my-app-svc

version-1
```

Deploy 1 replica of v2:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-2 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
```

Observe that calling the ip exposed by the service the requests are load balanced across the two versions:
```
# run a busyBox pod that will make a wget call to the service my-app-svc and print out the version of the pod it reached.
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox -- /bin/sh -c 'while sleep 1; do wget -qO- my-app-svc; done'

version-1
version-1
version-1
version-2
version-2
version-1
```

If the v2 is stable, scale it up to 4 replicas and shutdown the v1:
```
kubectl scale --replicas=4 deploy my-app-v2
kubectl delete deploy my-app-v1
while sleep 0.1; do curl $(kubectl get svc my-app-svc -o jsonpath="{.spec.clusterIP}"); done
version-2
version-2
version-2
version-2
version-2
version-2
```

</p>
</details>

## Jobs

### Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>show</summary>
<p>

```bash
kubectl create job pi  --image=perl:5.34 -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

</p>
</details>

### Wait till it's done, get the output

<details><summary>show</summary>
<p>

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl get po # get the pod name
kubectl logs pi-**** # get the pi numbers
kubectl delete job pi
```
OR

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl logs job/pi
kubectl delete job pi
```
OR

```bash
kubectl wait --for=condition=complete --timeout=300s job pi
kubectl logs job/pi
kubectl delete job pi
```

</p>
</details>

### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
kubectl get po # find the job pod
kubectl logs busybox-ptx58 -f # follow the logs
```

</p>
</details>

### See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### Delete the job

<details><summary>show</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Add job.spec.completions=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

Verify that it has been completed:

```bash
kubectl get job busybox -w # will take two and a half minutes
kubectl delete jobs busybox
```

</p>
</details>

### Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```bash
vi job.yaml
```

Add job.spec.parallelism=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

It will take some time for the parallel jobs to finish (>= 30 seconds)

```bash
kubectl delete job busybox
```

</p>
</details>

### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```

Add job.spec.activeDeadlineSeconds=30

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```
</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### See its logs and delete it

<details><summary>show</summary>
<p>

```bash
kubectl get po # copy the ID of the pod whose container was just created
kubectl logs <busybox-***> # you will see the date and message 
kubectl delete cj busybox # cj stands for cronjob
```

</p>
</details>

### Create the same cron job again, and watch the status. Once it ran, check which job ran by the created cron job. Check the log, and delete the cron job

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
# Bear in mind that Kubernetes will run a new job/pod for each new cron job
kubectl delete cj busybox
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add cronjob.spec.startingDeadlineSeconds=17

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 # add this line
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add cronjob.spec.jobTemplate.spec.activeDeadlineSeconds=12

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      activeDeadlineSeconds: 12 # add this line
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>

### Create a job from cronjob.

<details><summary>show</summary>
<p>

```bash
kubectl create job --from=cronjob/sample-cron-job sample-job
```
</p>
</details>

![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/configuration&empty)
# Configuration (18%)

[ConfigMaps](#configmaps)

[SecurityContext](#securitycontext)

[Requests and Limits](#requests-and-limits)

[Secrets](#secrets)

[Service Accounts](#serviceaccounts)

<br>#Tips, export to variable<br>
<br>export ns="-n secret-ops"</br>
<br>export do="--dry-run=client -oyaml"</br>
## ConfigMaps

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Create a configmap named config with values foo=lala,foo2=lolo

<details><summary>show</summary>
<p>

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

</p>
</details>

### Display its values

<details><summary>show</summary>
<p>

```bash
kubectl get cm config -o yaml
# or
kubectl describe cm config
```

</p>
</details>

### Create and display a configmap from a file

Create the file with

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml
```

</p>
</details>

### Create and display a configmap from a .env file

Create the file with the command

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml
```

</p>
</details>

### Create and display a configmap from a file, giving the key 'special'

Create the file with

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

<details><summary>show</summary>
<p>

```bash
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4
kubectl get cm configmap4 -o yaml
```

</p>
</details>

### Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

<details><summary>show</summary>
<p>

```bash
kubectl create cm options --from-literal=var5=val5
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env:
    - name: option # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: options # name of config map
          key: var5 # name of the entity in config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep option # will show 'option=val5'
```

</p>
</details>

### Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run --restart=Never nginx --image=nginx -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: anotherone # the name of the config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env 
```

</p>
</details>

### Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

<details><summary>show</summary>
<p>

```bash
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/lala # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # will show var8 var9
cat var8 # will show val8
```

</p>
</details>

## SecurityContext

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  securityContext: # insert this line
    runAsUser: 101 # UID for the user
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>


### Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added to its single container

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    securityContext: # insert this line
      capabilities: # and this
        add: ["NET_ADMIN", "SYS_TIME"] # this as well
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

## Resource requests and limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:    
        memory: "512Mi"
        cpu: "200m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 

</p>
</details>

## Limit Ranges
kubernetes.io > Documentation > Concepts > Policies > Limit Ranges (https://kubernetes.io/docs/concepts/policy/limit-range/)

### Create a namespace named limitrange with a LimitRange that limits pod memory to a max of 500Mi and min of 100Mi

<details><summary>show</summary>
<p>

```bash
kubectl create ns limitrange
```

vi 1.yaml
```YAML
apiVersion: v1
kind: LimitRange
metadata:
  name: ns-memory-limit
  namespace: limitrange
spec:
  limits:
  - max: # max and min define the limit range
      memory: "500Mi"
    min:
      memory: "100Mi"
    type: Pod
```

```bash
kubectl apply -f 1.yaml
```
</p>
</details>

### Describe the namespace limitrange

<details><summary>show</summary>
<p>

```bash
kubectl describe limitrange ns-memory-limit -n limitrange
```
</p>
</details>

### Create an nginx pod that requests 250Mi of memory in the limitrange namespace

<details><summary>show</summary>
<p>

vi 2.yaml
```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: limitrange
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "250Mi"
      limits:
        memory: "500Mi" # limit has to be specified and be <= limitrange
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 

```bash
kubectl apply -f 2.yaml
```
</p>
</details>


## Resource Quotas
kubernetes.io > Documentation > Concepts > Policies > Resource Quotas (https://kubernetes.io/docs/concepts/policy/resource-quotas/)

### Create ResourceQuota in namespace `one` with hard requests `cpu=1`, `memory=1Gi` and hard limits `cpu=2`, `memory=2Gi`.

<details><summary>show</summary>
<p>

Create the namespace:
```bash
kubectl create ns one
```

Create the ResourceQuota
```bash
vi rq-one.yaml
```

```YAML
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-rq
  namespace: one
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

```bash
kubectl apply -f rq-one.yaml
```

or
```bash
kubectl create quota my-rq --namespace=one --hard=requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi
```
</p>
</details>

### Attempt to create a pod with resource requests `cpu=2`, `memory=3Gi` and limits `cpu=3`, `memory=4Gi` in namespace `one`

<details><summary>show</summary>
<p>

```bash
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: one
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "3Gi"
        cpu: "2"
      limits:
        memory: "4Gi"
        cpu: "3"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml
```

Expected error message:
```bash
Error from server (Forbidden): error when creating "pod.yaml": pods "nginx" is forbidden: exceeded quota: my-rq, requested: limits.cpu=3,limits.memory=4Gi,requests.cpu=2,requests.memory=3Gi, used: limits.cpu=0,limits.memory=0,requests.cpu=0,requests.memory=0, limited: limits.cpu=2,limits.memory=2Gi,requests.cpu=1,requests.memory=1Gi
```
</p>
</details>

### Create a pod with resource requests `cpu=0.5`, `memory=1Gi` and limits `cpu=1`, `memory=2Gi` in namespace `one`

<details><summary>show</summary>
<p>

```bash
vi pod2.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: one
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: "0.5"
      limits:
        memory: "2Gi"
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod2.yaml
```

Show the ResourceQuota usage in namespace `one`
```bash
kubectl get resourcequota -n one
```

```
NAME    AGE   REQUEST                                          LIMIT
my-rq   10m   requests.cpu: 500m/1, requests.memory: 1Gi/1Gi   limits.cpu: 1/2, limits.memory: 2Gi/2Gi
```
</p>
</details>


## Secrets

kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### Create a secret called mysecret with the values password=mypass

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>

### Create a secret called mysecret2 that gets key/value from a file

Create a file called username with the value admin:

```bash
echo -n admin > username
```

<details><summary>show</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
```

</p>
</details>

### Get the value of mysecret2

<details><summary>show</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml
echo -n YWRtaW4= | base64 -d # on MAC it is -D, which decodes the value and shows 'admin'
```

Alternative using `--jsonpath`:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}' | base64 -d  # on MAC it is -D
```

Alternative using `--template`:

```bash
kubectl get secret mysecret2 --template '{{.data.username}}' | base64 -d  # on MAC it is -D
```

Alternative using `jq`:

```bash
kubectl get secret mysecret2 -o json | jq -r .data.username | base64 -d  # on MAC it is -D
```

</p>
</details>

### Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret2 # name of the secret - this must already exist on pod creation
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/bash
ls /etc/foo  # shows username
cat /etc/foo/username # shows admin
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env: # our env variables
    - name: USERNAME # asked name
      valueFrom:
        secretKeyRef: # secret reference
          name: mysecret2 # our secret's name
          key: username # the key of the data in the secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # will show 'admin'
```

</p>
</details>

### Create a Secret named 'ext-service-secret' in the namespace 'secret-ops'. Then, provide the key-value pair API_KEY=LmLHbYhsgWZwNifiqaRorH8T as literal.

<details><summary>show</summary>
<p>

```bash
export ns="-n secret-ops"
export do="--dry-run=client -oyaml"
k create secret generic ext-service-secret --from-literal=API_KEY=LmLHbYhsgWZwNifiqaRorH8T $ns $do > sc.yaml
k apply -f sc.yaml
```

</p>
</details>

### Consuming the Secret. Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops' and consume the Secret as an environment variable. Then, open an interactive shell to the Pod, and print all environment variables.
<details><summary>show</summary>
<p>

```bash
export ns="-n secret-ops"
export do="--dry-run=client -oyaml"
k run consumer --image=nginx $ns $do > nginx.yaml
vi nginx.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: consumer
  name: consumer
  namespace: secret-ops
spec:
  containers:
  - image: nginx
    name: consumer
    resources: {}
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: ext-service-secret
          key: API_KEY
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec -it $ns consumer -- /bin/sh
#env
```
</p>
</details>

### Create a Secret named 'my-secret' of type 'kubernetes.io/ssh-auth' in the namespace 'secret-ops'. Define a single key named 'ssh-privatekey', and point it to the file 'id_rsa' in this directory.
<details><summary>show</summary>
<p>

```bash
#Tips, export to variable
export do="--dry-run=client -oyaml"
export ns="-n secret-ops"

#if id_rsa file didn't exist.
ssh-keygen

k create secret generic my-secret $ns --type="kubernetes.io/ssh-auth" --from-file=ssh-privatekey=id_rsa $do > sc.yaml
k apply -f sc.yaml
```
</p>
</details>

### Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops', and consume the Secret as Volume. Mount the Secret as Volume to the path /var/app with read-only access. Open an interactive shell to the Pod, and render the contents of the file.
<details><summary>show</summary>
<p>

```bash
#Tips, export to variable
export ns="-n secret-ops"
export do="--dry-run=client -oyaml"
k run consumer --image=nginx $ns $do > nginx.yaml
vi nginx.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: consumer
  name: consumer
  namespace: secret-ops
spec:
  containers:
    - image: nginx
      name: consumer
      resources: {}
      volumeMounts:
        - name: foo
          mountPath: "/var/app"
          readOnly: true
  volumes:
    - name: foo
      secret:
        secretName: my-secret
        optional: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec -it $ns consumer -- /bin/sh
# cat /var/app/ssh-privatekey
# exit
```
</p>
</details>

## ServiceAccounts

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### See all the service accounts of the cluster in all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get sa --all-namespaces
```
Alternatively 

```bash
kubectl get sa -A
```

</p>
</details>

### Create a new serviceaccount called 'myuser'

<details><summary>show</summary>
<p>

```bash
kubectl create sa myuser
```

Alternatively:

```bash
# let's get a template easily
kubectl get sa default -o yaml > sa.yaml
vim sa.yaml
```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```

```bash
kubectl create -f sa.yaml
```

</p>
</details>

### Create an nginx pod that uses 'myuser' as a service account

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: myuser # we use pod.spec.serviceAccountName
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

or

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccount: myuser # we use pod.spec.serviceAccount
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl get pod nginx -o jsonpath='{.spec.serviceAccountName}' # output: myuser

kubectl exec -it nginx -- cat /var/run/secrets/kubernetes.io/serviceaccount/token # to check if the ServiceAccount token is mounted inside the Pod. Starting from Kubernetes 1.24+, the token is dynamically projected into the Pod

```

</p>
</details>

### Generate an API token for the service account 'myuser'

<details><summary>show</summary>
<p>
  
```bash
kubectl create token myuser
```

</p>
</details>

![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/observability&empty)
# Observability (18%)

## Liveness, readiness and startup probes

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

### Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: # our probe
      exec: # add this line
        command: # command definition
        - ls # ls command
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness # run this to see that liveness probe works
kubectl delete -f pod.yaml
```

</p>
</details>

### Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl explain pod.spec.containers.livenessProbe # get the exact names
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe:
      initialDelaySeconds: 5 # add this line
      periodSeconds: 5 # add this line as well
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe po nginx | grep -i liveness
kubectl delete -f pod.yaml
```

</p>
</details>

### Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never --port=80 > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    ports:
    - containerPort: 80 # Note: Readiness probes runs on the container during its whole lifecycle. Since nginx exposes 80, containerPort: 80 is not required for readiness to work.
    readinessProbe: # declare the readiness probe
      httpGet: # add this line
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness # to see the pod readiness details
kubectl delete -f pod.yaml
```

</p>
</details>

### Lots of pods are running in `qa`,`alan`,`test`,`production` namespaces.  All of these pods are configured with liveness probe.  Please list all pods whose liveness probe are failed in the format of `<namespace>/<pod name>` per line.

<details><summary>show</summary>
<p>

A typical liveness probe failure event
```
LAST SEEN   TYPE      REASON      OBJECT              MESSAGE
22m         Warning   Unhealthy   pod/liveness-exec   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

collect failed pods namespace by namespace

```sh
kubectl get events -o json | jq -r '.items[] | select(.message | contains("Liveness probe failed")).involvedObject | .namespace + "/" + .name'
```

</p>
</details>

## Logging

### Create a busybox pod that runs `i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done`. Check its logs

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # follow the logs
```

</p>
</details>

## Debugging

### Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# show that there's an error
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

</p>
</details>

### Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- notexist
kubectl logs busybox # will bring nothing! container never started
kubectl describe po busybox # in the events section, you'll see the error
# also...
kubectl get events | grep -i error # you'll see the error here as well
kubectl delete po busybox --force --grace-period=0
```

</p>
</details>


### Get CPU/memory utilization for nodes ([metrics-server](https://github.com/kubernetes-incubator/metrics-server) must be running)

<details><summary>show</summary>
<p>

```bash
kubectl top nodes
```

</p>
</details>

![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/services&empty)
# Services and Networking (13%)

### Create a pod with image nginx called nginx and expose its port 80

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
# observe that a pod as well as a service are created
```

</p>
</details>


### Confirm that ClusterIP has been created. Also check endpoints

<details><summary>show</summary>
<p>

```bash
kubectl get svc nginx # services
kubectl get ep # endpoints
```

</p>
</details>

### Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

<details><summary>show</summary>
<p>

```bash
kubectl get svc nginx # get the IP (something like 10.108.93.130)
kubectl run busybox --rm --image=busybox -it --restart=Never --
wget -O- [PUT THE POD'S IP ADDRESS HERE]:80
exit
```

</p>
or
<p>

```bash
IP=$(kubectl get svc nginx --template={{.spec.clusterIP}}) # get the IP (something like 10.108.93.130)
kubectl run busybox --rm --image=busybox -it --restart=Never --env="IP=$IP" -- wget -O- $IP:80 --timeout 2
# Tip: --timeout is optional, but it helps to get answer more quickly when connection fails (in seconds vs minutes)
```

</p>
</details>

### Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

<details><summary>show</summary>

<p>

```bash
kubectl create deploy foo --image=dgkanatsios/simpleapp --port=8080 --replicas=3
kubectl label deployment foo --overwrite app=foo #This is optional since kubectl create deploy foo will create label app=foo by default
```
</p>
</details>

### Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080
<details><summary>show</summary>
<p>


```bash
kubectl get pods -l app=foo -o wide # 'wide' will show pod IPs
kubectl run busybox --image=busybox --restart=Never -it --rm -- sh
wget -O- <POD_IP>:8080 # do not try with pod name, will not work
# try hitting all IPs generated after running 1st command to confirm that hostname is different
exit
# or
kubectl get po -o wide -l app=foo | awk '{print $6}' | grep -v IP | xargs -L1 -I '{}' kubectl run --rm -ti tmp --restart=Never --image=busybox -- wget -O- http://\{\}:8080
# or
kubectl get po -l app=foo -o jsonpath='{range .items[*]}{.status.podIP}{"\n"}{end}' | xargs -L1 -I '{}' kubectl run --rm -ti tmp --restart=Never --image=busybox -- wget -O- http://\{\}:8080
```

</p>
</details>

### Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

<details><summary>show</summary>
<p>


```bash
kubectl expose deploy foo --port=6262 --target-port=8080
kubectl get service foo # you will see ClusterIP as well as port 6262
kubectl get endpoints foo # you will see the IPs of the three replica pods, listening on port 8080
```

</p>
</details>

### Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

<details><summary>show</summary>

<p>

```bash
kubectl get svc # get the foo service ClusterIP
kubectl run busybox --image=busybox -it --rm --restart=Never -- sh
wget -O- foo:6262 # DNS works! run it many times, you'll see different pods responding
wget -O- <SERVICE_CLUSTER_IP>:6262 # ClusterIP works as well
# you can also kubectl logs on deployment pods to see the container logs
kubectl delete svc foo
kubectl delete deploy foo
```

</p>
</details>

### Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the pods in this deployment and apply it

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

> Note that network policies may not be enforced by default, depending on your k8s implementation. E.g. Azure AKS by default won't have policy enforcement, the cluster must be created with an explicit support for `netpol` https://docs.microsoft.com/en-us/azure/aks/use-network-policies#overview-of-network-policy  
  
<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose deployment nginx --port=80

kubectl describe svc nginx # see the 'app=nginx' selector for the pods
# or
kubectl get svc nginx -o yaml

vi policy.yaml
```

```YAML
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx # pick a name
spec:
  podSelector:
    matchLabels:
      app: nginx # selector for the pods
  ingress: # allow ingress traffic
  - from:
    - podSelector: # from pods
        matchLabels: # with this label
          access: granted
```

```bash
# Create the NetworkPolicy
kubectl create -f policy.yaml

# Check if the Network Policy has been created correctly
# make sure that your cluster's network provider supports Network Policy (https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/#before-you-begin)
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2                          # This should not work. --timeout is optional here. But it helps to get answer more quickly (in seconds vs minutes)
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted -- wget -O- http://nginx:80 --timeout 2  # This should be fine
```

</p>
</details>



![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/state&empty)
# State Persistence (8%)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Define volumes 

### Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

<details><summary>show</summary>
<p>

*This question is probably a better fit for the 'Multi-container-pods' section but I'm keeping it here as it will help you get acquainted with state*

The easiest way to do this is to create a template pod with:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```
Copy paste the container definition and type the lines that have a comment in the end:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2 # don't forget to change the name during copy paste, must be different from the first container's name!
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  volumes: #
  - name: myvolume #
    emptyDir: {} #
```
In case you forget to add ```bash -- /bin/sh -c 'sleep 3600'``` in template pod create command, you can include command field in config file

```YAML
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
```

Connect to the second container:

```bash
kubectl exec -it busybox -c busybox2 -- /bin/sh
cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd # instead of cut command you can use awk -F ":" '{print $1}'
cat /etc/foo/passwd # confirm that stuff has been written successfully
exit
```

Connect to the first container:

```bash
kubectl exec -it busybox -c busybox -- /bin/sh
mount | grep foo # confirm the mounting
cat /etc/foo/passwd
exit
kubectl delete po busybox
```

</p>
</details>


### Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

<details><summary>show</summary>
<p>

```bash
vi pv.yaml
```

```YAML
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
```

Show the PersistentVolumes:

```bash
kubectl create -f pv.yaml
# will have status 'Available'
kubectl get pv
```

</p>
</details>

### Create a PersistentVolumeClaim for this PersistentVolume, called 'mypvc', a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

<details><summary>show</summary>
<p>

```bash
vi pvc.yaml
```

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

Create it on the cluster:

```bash
kubectl create -f pvc.yaml
```

Show the PersistentVolumeClaims and PersistentVolumes:

```bash
kubectl get pvc # will show as 'Bound'
kubectl get pv # will show as 'Bound' as well
```

</p>
</details>

### Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

<details><summary>show</summary>
<p>

Create a skeleton pod:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```

Add the lines that finish with a comment:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
  - name: myvolume #
    persistentVolumeClaim: #
      claimName: mypvc #
status: {}
```

Create the pod:

```bash
kubectl create -f pod.yaml
```

Connect to the pod and copy '/etc/passwd' to '/etc/foo/passwd':

```bash
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
```

</p>
</details>

### Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup. Note: If you can't see the file from the second pod, can you figure out why? What would you do to fix that?



<details><summary>show</summary>
<p>

Create the second pod, called busybox2:

```bash
vim pod.yaml
# change 'metadata.name: busybox' to 'metadata.name: busybox2'
kubectl create -f pod.yaml
kubectl exec busybox2 -- ls /etc/foo # will show 'passwd'
# cleanup
kubectl delete po busybox busybox2
kubectl delete pvc mypvc
kubectl delete pv myvolume
```

If the file doesn't show on the second pod but it shows on the first, it has most likely been scheduled on a different node.

```bash
# check which nodes the pods are on
kubectl get po busybox -o wide
kubectl get po busybox2 -o wide
```

If they are on different nodes, you won't see the file, because we used the `hostPath` volume type.
If you need to access the same files in a multi-node cluster, you need a volume type that is independent of a specific node.
There are lots of different types per cloud provider [(see here)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes), a general solution could be to use NFS.

</p>
</details>

### Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
kubectl cp busybox:/etc/passwd ./passwd # kubectl cp command
# previous command might report an error, feel free to ignore it since copy command works
cat passwd
```

</p>
</details>


# Managing Kubernetes with Helm

- Note: Helm is part of the new CKAD syllabus. Here are a few examples of using Helm to manage Kubernetes.

## Helm in K8s

### Creating a basic Helm chart

<details><summary>show</summary>
<p>

```bash
helm create chart-test ## this would create a helm 
```

</p>
</details>

### Running a Helm chart

<details><summary>show</summary>
<p>

```bash
helm install -f myvalues.yaml myredis ./redis
```

</p>
</details>

### Find pending Helm deployments on all namespaces

<details><summary>show</summary>
<p>

```bash
helm list --pending -A
```

</p>
</details>

### Uninstall a Helm release

<details><summary>show</summary>
<p>

```bash
helm uninstall -n namespace release_name
```

</p>
</details>

### Upgrading a Helm chart

<details><summary>show</summary>
<p>

```bash
helm upgrade -f myvalues.yaml -f override.yaml redis ./redis
```

</p>
</details>

### Using Helm repo

<details><summary>show</summary>
<p>

Add, list, remove, update and index chart repos

```bash
helm repo add [NAME] [URL]  [flags]

helm repo list / helm repo ls

helm repo remove [REPO1] [flags]

helm repo update / helm repo up

helm repo update [REPO1] [flags]

helm repo index [DIR] [flags]
```

</p>
</details>

### Download a Helm chart from a repository 

<details><summary>show</summary>
<p>

```bash
helm pull [chart URL | repo/chartname] [...] [flags] ## this would download a helm, not install 
helm pull --untar [rep/chartname] # untar the chart after downloading it 
```

</p>
</details>

### Add the Bitnami repo at https://charts.bitnami.com/bitnami to Helm
<details><summary>show</summary>
<p>
    
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```
  
</p>
</details>

### Write the contents of the values.yaml file of the `bitnami/node` chart to standard output
<details><summary>show</summary>
<p>
    
```bash
helm show values bitnami/node
```
  
</p>
</details>

### Install the `bitnami/node` chart setting the number of replicas to 5
<details><summary>show</summary>
<p>

To achieve this, we need two key pieces of information:
- The name of the attribute in values.yaml which controls replica count
- A simple way to set the value of this attribute during installation

To identify the name of the attribute in the values.yaml file, we could get all the values, as in the previous task, and then grep to find attributes matching the pattern `replica`
```bash
helm show values bitnami/node | grep -i replica
```
which returns
```bash
## @param replicaCount Specify the number of replicas for the application
replicaCount: 1
```
 
We can use the `--set` argument during installation to override attribute values. Hence, to set the replica count to 5, we need to run
```bash
helm install mynode bitnami/node --set replicaCount=5
```

</p>
</details>

# Extend the Kubernetes API with CRD (CustomResourceDefinition)

- Note: CRD is part of the new CKAD syllabus. Here are a few examples of installing custom resource into the Kubernetes API by creating a CRD.

## CRD in K8s

### Create custom object from the CRD

* *Name* : `operator-sample`
* *Kind*: `Operator`
* Spec:
  * email: `operator-sample@stable.example.com`
  * name: `operator sample`
  * age: `30`

<details><summary>show</summary>
<p>

```yaml
apiVersion: stable.example.com/v1
kind: Operator
metadata:
  name: operator-sample
spec:
  email: operator-sample@stable.example.com
  name: "operator sample"
  age: 30
```

```bash
kubectl apply -f operator.yml
```

</p>
</details>

### Listing operator

<details><summary>show</summary>
<p>

Use singular, plural and short forms

```bash
kubectl get operators
or
kubectl get operator
or
kubectl get op
```

</p>
</details>


# Define, build and modify container images

- Note: The topic is part of the new CKAD syllabus. Here are a few examples of using **podman** to manage the life cycle of container images. The use of **docker** had been the industry standard for many years, but now large companies like [Red Hat](https://www.redhat.com/en/blog/say-hello-buildah-podman-and-skopeo) are moving to a new suite of open source tools: podman, skopeo and buildah. Also Kubernetes has moved in this [direction](https://kubernetes.io/blog/2022/02/17/dockershim-faq/). In particular, `podman` is meant to be the replacement of the `docker` command: so it makes sense to get familiar with it, although they are quite interchangeable considering that they use the same syntax.

## Podman basics

### Create a Dockerfile to deploy an Apache HTTP Server which hosts a custom main page

<details><summary>show</summary>
<p>

```Dockerfile
FROM docker.io/httpd:2.4
RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
```

</p>
</details>

### Build and see how many layers the image consists of

<details><summary>show</summary>
<p>

```bash
:~$ podman build -t simpleapp .
STEP 1/2: FROM httpd:2.4
STEP 2/2: RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
COMMIT simpleapp
--> ef4b14a72d0
Successfully tagged localhost/simpleapp:latest
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b

:~$ podman images
REPOSITORY               TAG         IMAGE ID      CREATED        SIZE
localhost/simpleapp      latest      ef4b14a72d02  8 seconds ago  148 MB
docker.io/library/httpd  2.4         98f93cd0ec3b  7 days ago     148 MB

:~$ podman image tree localhost/simpleapp:latest
Image ID: ef4b14a72d02
Tags:     [localhost/simpleapp:latest]
Size:     147.8MB
Image Layers
├── ID: ad6562704f37 Size:  83.9MB
├── ID: c234616e1912 Size: 3.072kB
├── ID: c23a797b2d04 Size: 2.721MB
├── ID: ede2e092faf0 Size: 61.11MB
├── ID: 971c2cdf3872 Size: 3.584kB Top Layer of: [docker.io/library/httpd:2.4]
└── ID: 61644e82ef1f Size: 6.144kB Top Layer of: [localhost/simpleapp:latest]
```

</p>
</details>

### Run the image locally, inspect its status and logs, finally test that it responds as expected

<details><summary>show</summary>
<p>

```bash
:~$ podman run -d --name test -p 8080:80 localhost/simpleapp
2f3d7d613ea6ba19703811d30704d4025123c7302ff6fa295affc9bd30e532f8

:~$ podman ps
CONTAINER ID  IMAGE                       COMMAND           CREATED        STATUS            PORTS                 NAMES
2f3d7d613ea6  localhost/simpleapp:latest  httpd-foreground  5 seconds ago  Up 6 seconds ago  0.0.0.0:8080->80/tcp  test

:~$ podman logs test
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
[Sat Jun 04 16:15:38.071377 2022] [mpm_event:notice] [pid 1:tid 139756978220352] AH00489: Apache/2.4.53 (Unix) configured -- resuming normal operations
[Sat Jun 04 16:15:38.073570 2022] [core:notice] [pid 1:tid 139756978220352] AH00094: Command line: 'httpd -D FOREGROUND'

:~$ curl 0.0.0.0:8080
Hello, World!
```

</p>
</details>

### Run a command inside the pod to print out the index.html file

<details><summary>show</summary>
<p>

```bash
:~$ podman exec -it test cat /usr/local/apache2/htdocs/index.html
Hello, World!
```
</p>
</details>

### Tag the image with ip and port of a private local registry and then push the image to this registry

<details><summary>show</summary>
<p>

> Note: Some small distributions of Kubernetes (such as [microk8s](https://microk8s.io/docs/registry-built-in)) have a built-in registry you can use for this exercise. If this is not your case, you'll have to setup it on your own.

```bash
:~$ podman tag localhost/simpleapp $registry_ip:5000/simpleapp

:~$ podman push $registry_ip:5000/simpleapp
```

</p>
</details>

 Verify that the registry contains the pushed image and that you can pull it

<details><summary>show</summary>
<p>

```bash
:~$ curl http://$registry_ip:5000/v2/_catalog
{"repositories":["simpleapp"]}

# remove the image already present
:~$ podman rmi $registry_ip:5000/simpleapp

:~$ podman pull $registry_ip:5000/simpleapp
Trying to pull 10.152.183.13:5000/simpleapp:latest...
Getting image source signatures
Copying blob 643ea8c2c185 skipped: already exists
Copying blob 972107ece720 skipped: already exists
Copying blob 9857eeea6120 skipped: already exists
Copying blob 93859aa62dbd skipped: already exists
Copying blob 8e47efbf2b7e skipped: already exists
Copying blob 42e0f5a91e40 skipped: already exists
Copying config ef4b14a72d done
Writing manifest to image destination
Storing signatures
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b
```

</p>
</details>

### Create a container without running/starting it

<details><summary>show</summary>
<p>

```bash
:~$ podman create busybox # create
Resolved "busybox" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/busybox:latest...
Getting image source signatures
Copying blob sha256:213a27df5921cd9ae24732504c590bb6408911c20fb50a597f2a40896d554a8f
Copying config sha256:3fba0c87fcc8ba126bf99e4ee205b43c91ffc6b15bb052315312e71bc6296551
Writing manifest to image destination
51b613406e8889213c176523e1c430e4bd00047965b0c22cff5b1c9badfbc452

:~$ podman container ls -a
CONTAINER ID  IMAGE                             COMMAND     CREATED        STATUS      PORTS       NAMES
51b613406e88  docker.io/library/busybox:latest  sh          2 minutes ago  Created                 adoring_almeida
```

</p>
</details>

### Export a container to output.tar file

<details><summary>show</summary>
<p>

```bash
:~$ podman container ls -a # pick the container id
CONTAINER ID  IMAGE                             COMMAND     CREATED        STATUS      PORTS       NAMES
51b613406e88  docker.io/library/busybox:latest  sh          2 minutes ago  Created                 adoring_almeida

:~$ podman export <container id> --output=output.tar

:~$ ls -al output.tar
-rw-r--r--@ 1 limistah  wheel  4272640 28 Aug 13:48 output.tar
```

</p>
</details>

### Run a pod with the image pushed to the registry

<details><summary>show</summary>
<p>

```bash
:~$ kubectl run simpleapp --image=$registry_ip:5000/simpleapp --port=80

:~$ curl ${kubectl get pods simpleapp -o jsonpath='{.status.podIP}'}
Hello, World!
```

</p>
</details>

### Log into a remote registry server and then read the credentials from the default file

<details><summary>show</summary>
<p>

> Note: The two most used container registry servers with a free plan are [DockerHub](https://hub.docker.com/) and [Quay.io](https://quay.io/).

```bash
:~$ podman login --username $YOUR_USER --password $YOUR_PWD docker.io
:~$ cat ${XDG_RUNTIME_DIR}/containers/auth.json
{
        "auths": {
                "docker.io": {
                        "auth": "Z2l1bGl0JLSGtvbkxCcX1xb617251xh0x3zaUd4QW45Q3JuV3RDOTc="
                }
        }
}
```

</p>
</details>

### Create a secret both from existing login credentials and from the CLI

<details><summary>show</summary>
<p>

```bash
:~$ kubectl create secret generic mysecret --from-file=.dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type=kubernetes.io/dockeconfigjson
secret/mysecret created
:~$ kubectl create secret docker-registry mysecret2 --docker-server=https://index.docker.io/v1/ --docker-username=$YOUR_USR --docker-password=$YOUR_PWD
secret/mysecret2 created
```

</p>
</details>

### Create the manifest for a Pod that uses one of the two secrets just created to pull an image hosted on the relative private remote registry

<details><summary>show</summary>
<p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: $YOUR_PRIVATE_IMAGE
  imagePullSecrets:
  - name: mysecret
```

</p>
</details>

### Clean up all the images and containers

<details><summary>show</summary>
<p>

```bash
:~$ podman rm --all --force
:~$ podman rmi --all
:~$ kubectl delete pod simpleapp
```

</p>
</details>


## 🧩 CKAD MOCK EXAM — 17 QUESTIONS (Timed Practice)

> 💡 Tip: Spend **5–7 min per question**.  
> Use `kubectl -n ckad` alias, `--dry-run=client -o yaml`, and `kubectl explain` freely.

---

### 🧱 **1. POD BASICS**

Create a Pod named `frontend` using image `nginx:1.27`, with label `tier=frontend`, running in namespace `ckad`.

---

### 🪣 **2. MULTI-CONTAINER**

Pod `logger` with 2 containers:

- `app` → busybox, runs `while true; do date; sleep 5; done`
    
- `sidecar` → busybox, tails `/var/log/date.log` from a shared `emptyDir`.
    

---

### 🧩 **3. INIT CONTAINER**

Pod `initdemo`: init container writes `Ready` to `/data/ready.txt`, main container waits for it before starting.

---

### 📦 **4. CONFIGMAP & SECRET**

ConfigMap `app-config` → `MODE=prod`  
Secret `app-secret` → `TOKEN=xyz`  
Pod `secureapp` reads both as env vars.

---

### 🧠 **5. SECURITY CONTEXT**

Run container `busybox` as user `1001`, disallow privilege escalation, add command `id`.

---

### 🧭 **6. DEPLOYMENT + UPDATE**

Create a Deployment `webapp` with 3 replicas, image `nginx:1.25`.  
Perform a rolling update to `nginx:1.27`, check rollout, and rollback.

---

### 🌀 **7. CANARY / BLUE-GREEN**

Two Deployments:

- `blue` v1 (3 replicas)
    
- `green` v2 (3 replicas)  
    Service `web` initially points to `blue`, then switch to `green`.
    

---

### 📦 **8. PERSISTENT VOLUME**

Create PVC `mypvc` (storage 100Mi, access `ReadWriteOnce`), mount at `/data` in a Pod.

---

### 🧰 **9. LIVENESS + READINESS**

Pod `healthcheck` — add both probes (GET `/` on 80), with initialDelay 5s.

---

### 🕵️ **10. DEBUGGING**

Pod `crashy` runs busybox and exits immediately.  
Troubleshoot using `kubectl debug` to see why it’s failing.

---

### 🧮 **11. HPA**

Deployment `autoscale-app` (image nginx, replicas 2).  
Create HPA to scale 1–5 replicas at 50% CPU.

---

### 🪪 **12. SERVICE ACCOUNT**

ServiceAccount `reader`.  
Pod `sa-demo` runs with that SA.  
Verify `kubectl exec sa-demo -- cat /var/run/secrets/.../token`.

---

### 🔐 **13. RESOURCE QUOTAS**

Namespace `quota-ns`.  
Create a ResourceQuota limiting 2 Pods and 1 ConfigMap.

---

### 🌐 **14. SERVICE**

Expose Deployment `backend` on port 8080 as ClusterIP Service `backend-svc`.

---

### 🕸 **15. INGRESS**

Ingress `web-ingress` routes `/app` → `backend-svc:8080`.

---

### 🚧 **16. NETWORK POLICY**

Namespace `net-ns`.  
Allow ingress to Pods labeled `role=db` only from `role=api`.

---

### 🧩 **17. CRD + DEBUG**

Install a sample CRD (e.g., `MyResource`) and inspect with `kubectl explain`.  
Then debug a failing Pod using logs and `kubectl exec`.

---

## 🕓 TIMED EXECUTION STRATEGY
