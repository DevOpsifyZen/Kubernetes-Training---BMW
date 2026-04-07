## Resource Quotas

In Kubernetes, a **ResourceQuota** is a way to limit the amount of resources (CPU, memory, and persistent storage) that a namespace can consume. It helps in preventing resource exhaustion and ensures that one namespace doesn't negatively impact the performance or stability of others.

Keep in mind that **ResourceQuotas** are a way to define constraints, and they do not actively enforce them on existing resources. They are checked when new resources are created or when existing resources are updated. If a namespace exceeds its quota, the creation or update of resources may be rejected.

### Task 1: Creating a Namespace

Creating a Namespace
```
kubectl create namespace rq-ns
```
Verify the namespace
```
kubectl get ns
```
Describe the namespace
```
kubectl describe ns rq-ns
```
Check for existing ResourceQuota (none expected)
```
kubectl describe quota -n rq-ns
```


### Task 2: Creating Resource Quota 

Create ResourceQuota YAML
```
vi rq.yaml
```
Add the given content, by pressing INSERT
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota
  namespace: rq-ns
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "2"
    services: "1"
    count/deployments.apps: "1"
```
save the file using ESCAPE + :wq!

Apply the ResourceQuota
```
kubectl apply -f rq.yaml
```
Verify quota is applied
```
kubectl describe ns rq-ns
```
```
kubectl describe quota -n rq-ns
```
### Task 3: Verify Resource Quota Functionality

Try creating a Pod without resource requests (will fail)
```
kubectl -n rq-ns run pod1 --image nginx --port 80
```
Create Pod with valid resource requests
```
vi rq-pod-1.yaml
```
Add the given content, by pressing INSERT
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rq-pod-1
  namespace: rq-ns
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "1000m"
      requests:
        memory: "600Mi"
        cpu: "350m"
    ports:
      - containerPort: 80
```
save the file using ESCAPE + :wq!

Apply the pod:
```	  
kubectl apply -f rq-pod-1.yaml
```

Check quota usage
```
kubectl describe quota -n rq-ns
```

```
kubectl get resourcequota -n rq-ns -o yaml
```

Deploy another Pod, such that the total resource exceeds the resoucre quota limit
```
vi rq-pod-2.yaml
```
Add the given content, by pressing INSERT
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rq-pod-2
  namespace: rq-ns
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "1600Mi"
        cpu: "2000m"
      requests:
        memory: "1600Mi"
        cpu: "800m"
    ports:
      - containerPort: 80
```
save the file using ESCAPE + :wq!

Apply the Pod:
```
kubectl apply -f rq-pod-2.yaml
```
❌ Pod creation will FAIL

### Task 4: Cleanup the resources
Delete all the resources created on above steps


