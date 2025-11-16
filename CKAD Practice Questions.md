Excellent 💪 — scenario-based practice is _exactly_ how you’ll level up for your **CKAD retake on Nov 2**.  
Below is a curated list of **realistic, exam-style CKAD scenarios** designed to build both _speed_ and _muscle memory_ — the two things that will win you the exam this time.

All of these can be done on **any Kubernetes cluster** (Minikube, Kind, KodeKloud labs, etc.).  
They’re divided by **CKAD domains**, with each task having a _goal_, _hint_, and an estimated _weight / time_ to mimic the real exam.

---

# 🧠 CKAD Scenario Practice Set (40 Scenarios)

---

## 🧱 1️⃣ Core Concepts (Pods, Multi-container, Namespace, Labels)

**Weight:** ~13% of exam

1. **Create a Pod with specific labels**
    
    - Create a Pod named `frontend` using image `nginx:1.25` in namespace `project-a`.
        
    - Add labels `tier=frontend`, `env=prod`.
        
    - Verify pod is running and labeled correctly.
        
2. **Multi-container Pod (sidecar pattern)**
    
    - Pod name: `logger-pod`.
        
    - Main container: `busybox` runs `echo "hello" > /var/log/app.log`.
        
    - Sidecar: `nginx` serves `/var/log` over HTTP.
        
    - Shared volume: `emptyDir`.
        
3. **Pod with environment variable from Downward API**
    
    - Pod name: `meta-reader`.
        
    - Inject pod name into environment variable `MY_POD_NAME`.
        
4. **Pod restart policy check**
    
    - Create a Pod that fails intentionally (`busybox false`).
        
    - Set `restartPolicy: Never` and verify completion status.
        

---

## ⚙️ 2️⃣ Configuration (ConfigMaps, Secrets, Env Vars, Probes)

**Weight:** ~18%

5. **Inject config values**
    
    - Create ConfigMap `app-config` with `APP_COLOR=red` and `APP_MODE=dev`.
        
    - Create Pod `config-demo` that reads both as env variables.
        
6. **Mount ConfigMap as volume**
    
    - Create ConfigMap `html-content` from literal `index.html="Welcome to CKAD!"`.
        
    - Mount to `/usr/share/nginx/html` in Pod `web`.
        
7. **Use Secrets for credentials**
    
    - Create Secret `db-secret` with `username=admin` and `password=s3cr3t`.
        
    - Pod `mysql-client` reads them as env variables.
        
8. **Liveness and Readiness probes**
    
    - Deployment `api` with 2 replicas (image: nginx).
        
    - Liveness probe: HTTP GET `/healthz` on port 80.
        
    - Readiness probe: TCP check port 80.
        
9. **Resource limits**
    
    - Pod `limited` with image `nginx`.
        
    - Add resource requests (cpu: 100m, memory: 128Mi) and limits (cpu: 200m, memory: 256Mi).
        

---

## 🌐 3️⃣ Multi-Container & Observability

**Weight:** ~10%

10. **Logging Sidecar**
    
    - Pod `logger` with:
        
        - main: busybox runs `while true; do echo "$(date)" >> /var/log/app.log; sleep 3; done`
            
        - sidecar: nginx serving `/var/log` via HTTP.
            
        - Shared volume `emptyDir`.
            
11. **Exec into Pod and check logs**
    
    - Run `kubectl exec` and `kubectl logs` to verify multi-container logging.
        
12. **Add lifecycle hooks**
    
    - Pod `hook-demo` with preStart hook `echo preStart called` and postStart hook `echo postStart done`.
        

---

## 🚀 4️⃣ Application Design & Build (Deployments, Rollouts, Scaling)

**Weight:** ~20%

13. **Basic Deployment**
    
    - Deployment `webapp` (image: nginx, replicas: 3).
        
    - Expose via ClusterIP `webapp-svc` on port 80.
        
14. **Rolling update**
    
    - Update image from `nginx:1.25` → `nginx:1.26`.
        
    - Verify rollout status and history.
        
15. **Rollback Deployment**
    
    - Revert to previous image using `kubectl rollout undo`.
        
16. **Canary rollout**
    
    - Deployment `webapp-canary` with label `version=canary`, image `nginx:1.27`, replicas: 1.
        
    - Service should route to both stable + canary pods (use selector `app=webapp`).
        
17. **Autoscaling**
    
    - Enable HPA for `webapp`:
        
        - min=2, max=5, target CPU utilization=60%.
            
        - Verify HPA object created.
            

---

## 🔄 5️⃣ Services & Networking

**Weight:** ~13%

18. **ClusterIP Service**
    
    - Expose `webapp` as ClusterIP `webapp-svc` on port 8080 → targetPort 80.
        
19. **NodePort Service**
    
    - Expose same deployment as NodePort 30080.
        
20. **Ingress creation**
    
    - Create Ingress `web-ingress` mapping `/` to service `webapp-svc:80`.
        
    - Verify with `curl` from test Pod.
        
21. **DNS resolution**
    
    - Create Pod `dns-test` using image `busybox`.
        
    - Resolve service `webapp-svc` via `nslookup`.
        
22. **NetworkPolicy**
    
    - Only allow traffic to `nginx` pods from pods labeled `access=granted`.
        

---

## 🧩 6️⃣ State Persistence

**Weight:** ~8%

23. **Persistent VolumeClaim**
    
    - PVC `data-pvc` (storage: 100Mi, accessMode: ReadWriteOnce).
        
    - Pod `pvc-demo` mounts at `/data`.
        
24. **Init container to prepare data**
    
    - Init container `init` writes `hello.txt` into `/data`.
        
    - Main container `nginx` serves `/data` directory.
        

---

## 🔒 7️⃣ Security & ServiceAccounts

**Weight:** ~10%

25. **ServiceAccount usage**
    
    - Create SA `custom-sa`.
        
    - Run Pod `sa-pod` using that SA.
        
26. **RBAC Role & RoleBinding**
    
    - Role `pod-reader` (verbs: get, list, watch pods).
        
    - Bind it to `custom-sa` in namespace `default`.
        
27. **Run container as non-root**
    
    - Pod `secure` using `busybox`, `securityContext.runAsUser: 1000`.
        

---

## 🧮 8️⃣ Scheduling

**Weight:** ~8%

28. **NodeSelector**
    
    - Deployment `backend` should run only on nodes with label `role=backend`.
        
29. **Tolerations**
    
    - Pod `tolerate` runs on node tainted with `key=debug:NoSchedule`.
        
30. **Affinity**
    
    - Pod `affinity-demo` must run on same node as pods labeled `app=db`.
        

---

## ⚡ 9️⃣ Troubleshooting (High-value domain)

**Weight:** ~8%

31. **CrashLoopBackOff**
    
    - Create faulty Pod that fails due to wrong command.
        
    - Debug with `kubectl logs` and fix.
        
32. **Pending Pod**
    
    - Pod can’t schedule due to missing PVC.
        
    - Identify cause using `describe`.
        
33. **Service misconfiguration**
    
    - Fix typo in selector label causing `Endpoints` = 0.
        
34. **Wrong image**
    
    - Fix “ImagePullBackOff” by correcting image name.
        
35. **Broken probes**
    
    - Liveness probe points to wrong path — fix it.
        

---

## 🧭 10️⃣ Exam Simulation (Mixed)

**Each of these is a 10–12 pt composite question.**

36. **Web stack setup**
    
    - Create ConfigMap `web-config` with `COLOR=blue`.
        
    - Deploy `frontend` (nginx) using env from ConfigMap.
        
    - Expose as NodePort 30080.
        
37. **Secure API deployment**
    
    - Secret `api-cred` (key + token).
        
    - Deployment `api-server` mounts Secret as file.
        
    - ServiceAccount `api-sa` + Role to list pods.
        
    - Liveness probe `/health`.
        
38. **Logging pattern**
    
    - Multi-container Pod with busybox generating logs and nginx exposing them.
        
    - Add ConfigMap for message frequency.
        
    - Expose using ClusterIP.
        
39. **Upgrade workflow**
    
    - Deployment `nginx-app:1.25`.
        
    - Apply rolling update → 1.26 → rollback.
        
    - Verify revision history.
        
40. **Ingress route**
    
    - Two services (`app1`, `app2`).
        
    - Ingress routes `/app1` to service1, `/app2` to service2.
        

---

# ⏱️ Practice Tips

✅ **Simulate exam environment:**

```bash
kubectl run busybox --image=busybox -it --restart=Never -- sh
```

Use that as your test pod for curl/ping.

✅ **Time yourself:**  
Each scenario = 5–10 min.  
Track time + correctness → aim for **< 6 min average per task**.

✅ **Daily goal (till Nov 1):**

- 10 scenarios/day
    
- 2 mock exams (Nov 1)
    

---

Would you like me to make this into a **Google Sheet / Notion tracker** (columns: _Scenario | Domain | Target Time | Status | Notes | % Accuracy_) so you can check off each task you finish between today and Nov 1?