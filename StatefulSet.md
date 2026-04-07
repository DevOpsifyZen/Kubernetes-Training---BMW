## StatefulSet

A **StatefulSe**t is similar to a Deployment but is designed for applications where ***each pod must maintain a unique identity and its own persistent data,*** even if the pod is deleted or restarted.

**It is mainly used for:**
* Databases (MySQL, MongoDB, PostgreSQL)
* Distributed systems (Kafka, Zookeeper, Cassandra, Redis)
* Any workload that needs stable storage + predictable pod names

<img width="945" height="268" alt="image" src="https://github.com/user-attachments/assets/47a2b659-d725-435e-9ed6-9e8478ab3474" />

**Key Features of StatefulSet:**
* Stable, predictable pod names (pod-0, pod-1, pod-2).
* Ordered deployment (pods start sequentially: 0 → 1 → 2).
* Ordered termination (pods stop in reverse order: 2 → 1 → 0).
* Stable network identity for each pod via DNS.
* Requires a headless service (clusterIP: None) for pod DNS resolution.
* Persistent storage per pod using volumeClaimTemplates.
* PVCs remain even after pods are deleted (data is preserved).
* Each pod gets its own dedicated volume (no sharing unless explicitly allowed).
* Rolling updates happen one pod at a time for stability.
Best suited for stateful applications like databases and distributed systems.

---

### Task 1: Create Stateful Set
Create the Stateful Set yaml file
```
vi webapp-sts.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-svc
  labels:
    app: webapp-svc
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: webapp-sts
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: webapp-sts
spec:
  serviceName: webapp-svc
  replicas: 2
  selector:
    matchLabels:
      app: webapp-sts
  template:
    metadata:
      labels:
        app: webapp-sts
    spec:
      containers:
      - name: nginx
        image: nginx:1.24.0
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
save the file using `ESCAPE + :wq!`

Create a Stateful Set and  headless service by applying the yaml
```
kubectl apply -f webapp-sts.yaml
```

Validate the headless service 
```
kubectl get service webapp-svc
```

Validate the stateful set creation
```
kubectl get statefulset webapp-sts
```

Watch the pods getting created in an ordinal index fashion
```
kubectl get pods -w -l app=webapp-sts
```

Create a busybox pod to test the one of the pods by using DNS name 
```
kubectl run -it --image busybox:1.28 test-pod --restart=Never --rm
```
```
nslookup webapp-sts-0.webapp-svc
```
```
nslookup webapp-sts-1.webapp-svc
```
```
exit
```

Delete the pods for a stateful set
```
kubectl delete pod -l app=webapp-sts
```

In another window notice the new pods getting created in a proper order
```
kubectl get pod -w -l app=webapp-sts
```

### Task 2: Scaling a Stateful Set
Scale the Stateful Set to 5 replicas using below.
```
kubectl scale sts webapp-sts --replicas=5
```

Verify the pods getting created in ordinal way
```
kubectl get pods -w -l app=webapp-sts
```

Verify the PV Claim getting created in ordinal fashion
```
kubectl get pvc -l app=webapp-sts
```

Edit the stateful Set yam and reduce replicas to 3 
```
kubectl edit sts webapp-sts
```

Notice that the controller deletes the pods one at a time. It waits for one to completely shut down before going to next
```
kubectl get pods -w -l app=webapp-sts
```

Verify statefulSet’s PersistentVolumeClaims and verify that are not deleted on scaling down. 
```
kubectl get pvc -l app=webapp-sts
```

### Task 3: Cleanup the resources using below command 
Delete a Stateful Set
```
kubectl delete -f webapp-sts.yaml
```
List all the PV and PVC’s that has been allocated to Statefulset pods and delete them as below.
```
kubectl get pvc
```
```
kubectl delete pvc --all
```



