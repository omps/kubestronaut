``` pod.yml
apiVersion: v1
kind: pod
metadata:
	name: nginx
	labels:
		app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.24.0-alpine-slim
```
