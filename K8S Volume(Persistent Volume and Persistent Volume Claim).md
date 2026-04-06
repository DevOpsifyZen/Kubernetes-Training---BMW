# Kubernetes Storage:

## 🔹 Persistent Volume (PV)

A **Persistent Volume (PV)** is a **cluster-level storage resource** in Kubernetes.

- Represents actual physical storage (disk, NFS, cloud, etc.)
- Created and managed by the administrator
- Independent of Pods
- Has its own lifecycle

### Key Characteristics:
- Pre-provisioned storage
- Defined by:
  - Capacity (e.g., 1Gi)
  - Access Modes (ReadWriteOnce, ReadWriteMany, etc.)
  - Storage type (hostPath, NFS, EBS, etc.)
- Can be reused depending on reclaim policy

*A PV is like a storage box already available in the cluster*

---

## 🔹 Persistent Volume Claim (PVC)

A **Persistent Volume Claim (PVC)** is a **request for storage** by a user or application.

- Acts as a bridge between Pod and PV
- Users don’t need to know storage details
- Kubernetes binds PVC to a matching PV

### Key Characteristics:
- Requests:
  - Storage size (e.g., 1Gi)
  - Access mode
  - Storage class
- Once matched → PVC is **bound** to PV
 
*A PVC is like a request form asking for a storage box*

---

## 🔹 PV and PVC Binding

Kubernetes automatically matches PVC with a suitable PV based on:

- Storage Class
- Capacity
- Access Modes

### Flow:

> **Pod → PVC → PV → Physical Storage**


---

## Static Provisioning

**Static Provisioning** means:

👉 Storage (PV) is created manually **before** it is requested.

### Workflow:
1. Admin creates PV
2. User creates PVC
3. Kubernetes binds PVC to PV
4. Pod uses PVC

### Key Points:
- Manual process
- Requires pre-planning
- Common in on-prem environments
- Less flexible than dynamic provisioning

*Pre-built storage rooms waiting to be claimed*

---

## 🔹 Access Modes

- **ReadWriteOnce (RWO)**  
  Mounted by one node

- **ReadWriteMany (RWX)**  
  Mounted by multiple nodes

- **ReadOnlyMany (ROX)**  
  Read-only access by multiple nodes

---

## 🔹 Why Use PV & PVC?

### Without PV/PVC:
- Data is lost when Pod is deleted

### With PV/PVC:
- Data persists beyond Pod lifecycle
- Storage is reusable
- Separation of responsibilities:
  - Developers → request storage (PVC)
  - Administrators → manage storage (PV)

---

## 🔹 Static vs Dynamic Provisioning

| Feature | Static Provisioning | Dynamic Provisioning |
|--------|-------------------|---------------------|
| Creation | Manual | Automatic |
| Flexibility | Low | High |
| StorageClass | Optional | Required |
| Use Case | On-prem / small setups | Cloud / scalable systems |

---


## Static Provisioning

### Task 1: Get Node Label and Create Custom Index.html on Node
Label the node 
```
kubectl label node <node name> role=node
```
View worker nodes and their labels
```
kubectl get nodes --show-labels | grep role=node
```
ssh to the labeled node using below command
```
ssh -t userName@<node_public_IP> 
```
Switch to root and run the following commands. A directory with custom index.html is created for PersistentVolume mount 
```
sudo su
```
```
mkdir /pvdir
```
```
echo Hello World! > /pvdir/index.html
```
### On the Master Node
### Task 2: Create a Local Persistent Volume
```
vi pv-volume.yaml
```
Add the given content, by pressing `INSERT`

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pvdir"
```
save the file using `ESCAPE + :wq!`

Apply the **YAML**

```
kubectl apply -f pv-volume.yaml
```
```
kubectl get pv
```
```
kubectl describe pv pv-volume
```

### Task 3: Create a PV Claim
```
vi pv-claim.yaml
```
Add the given content, by pressing `INSERT`

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
save the file using `ESCAPE + :wq!`

Apply the **YAML**
```
kubectl apply -f pv-claim.yaml
```

### Task 4: Create nginx Pod with NodeSelector
```
vi pv-pod.yaml
```
Add the given content, by pressing `INSERT`

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
     - name: pv-container
       image: nginx
       ports:
          - containerPort: 80
            name: "http-server"
       volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: pv-storage
  nodeSelector:
    role: node
```
save the file using `ESCAPE + :wq!`

Apply the **YAML**

```
kubectl apply -f pv-pod.yaml
```
View Pod details and see that is created on the required node
```
kubectl get pods -o wide
```
Access shell on a container running in your Pod
```
kubectl exec -it pv-pod -- /bin/bash
```
```
exit
```
**delete the resources created in this lab.**
```
kubectl delete -f pv-pod.yaml
```
```
kubectl delete -f pv-claim.yaml
```
```
kubectl delete -f pv-volume.yaml
```



