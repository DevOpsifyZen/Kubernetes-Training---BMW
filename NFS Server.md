## Lab: Configuring Pod Using NFS-Based PV and PVC

Configure a Pod using **NFS-based PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)** for efficient storage management in Kubernetes.

---

## Architecture Overview

- One node acts as **NFS Server**
- All nodes act as **NFS Clients**
- Kubernetes uses:
  - PV → NFS storage
  - PVC → request storage
  - Pod → consumes PVC

---

## NFS-Based Persistent Storage in Kubernetes


## What is NFS?

**NFS (Network File System)** is a distributed file system that allows multiple systems to access shared storage over a network.

👉 It enables **multiple Pods on different nodes to share the same data**

---

## 🔹 Why Use NFS in Kubernetes?

- Supports **ReadWriteMany (RWX)**
- Shared storage across multiple Pods
- Simple to set up (compared to cloud storage)
- Useful for:
  - Web content sharing
  - Logs
  - Shared configuration

---

## 🔹 How NFS Works in Kubernetes

### Flow:
> **Pod → PVC → PV → NFS Server → Shared Directory**


### 1. NFS Server
- Stores actual data
- Exposes directory over network
- Example: `/mydbdata`

### 2. PersistentVolume (PV)
- Points to NFS server
- Defines:
  - Server IP
  - Path
  - Access mode (RWX)

### 3. PersistentVolumeClaim (PVC)
- Requests storage
- Gets bound to PV

### 4. Pod / Deployment
- Mounts PVC as a volume
- Reads/writes data to NFS


## Access Mode Importance

### ReadWriteMany (RWX)
- Multiple Pods can:
  - Read
  - Write
- Across multiple nodes

👉 This is why NFS is commonly used

---

## Advantages of NFS

- Shared storage across nodes
- Easy to configure
- Cost-effective (no cloud dependency)
- Supports scaling applications


## Limitations

- Performance depends on network
- Single point of failure (if one server)
- Not ideal for high-performance workloads
- Security needs proper configuration

---

## NFS vs hostPath

| Feature | NFS | hostPath |
|--------|-----|---------|
| Scope | Network-wide | Single node |
| Sharing | Multi-node | Not shareable |
| Use case | Production | Testing |

---

## When to Use NFS?

- Multi-pod shared storage
- Stateless apps needing shared files
- On-prem Kubernetes clusters

---

👉 **One-line explanation:**  
*NFS allows multiple Kubernetes Pods to share the same storage across nodes*

---


### Task 1: Configure the NFS Server

Perform these steps on **worker-node-1** (which will act as the NFS server):

Create a directory
```
sudo mkdir /mydbdata
```
Install the NFS kernel server
```
sudo apt update && sudo apt install -y nfs-kernel-server
```

🔹Set the Permissions:

Edit the exports file
```
sudo nano /etc/exports
```

Append the following line:
```
/mydbdata *(rw,sync,no_root_squash)
```
Verify configuration
```
sudo cat /etc/exports
```
Export the directories
```
sudo exportfs -rv
```
Set ownership and permissions
```
sudo chown nobody:nogroup /mydbdata/
```
```
sudo chmod 777 /mydbdata/
```
Restart NFS server
```
sudo systemctl restart nfs-kernel-server
```
```
sudo systemctl status nfs-kernel-server
```
Get NFS server IP
```
ip a
```

👉 Save this internal IP — it will be used in the PV configuration.

### Task 2: Configure the NFS Common on Client Machines (All Worker Nodes)

Perform the following steps on all worker nodes:

Install NFS common
```
sudo apt update && sudo apt install -y nfs-common
```
Refresh NFS common service
```
sudo rm /lib/systemd/system/nfs-common.service
```
```
sudo systemctl daemon-reload
```

Restart and check status
```
sudo systemctl restart nfs-common
```
```
sudo systemctl status nfs-common
```

### Task 3: Create the PersistentVolume (PV)
Create the PV YAML file
```bash
vi nfs-pv.yaml
```
Add the following code
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
  labels:
    app: nfs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: <YOUR_NFS_SERVER_IP>
    path: "/mydbdata"

```
🔑 Replace <YOUR_NFS_SERVER_IP> with the IP from **Task 1**

Apply and check PV
```
kubectl apply -f nfs-pv.yaml
```
```
kubectl get pv
```

### Task 4: Create the PersistentVolumeClaim (PVC)
Create the PVC YAML file
```
vi nfs-pvc.yaml
```
Add the following code
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
  labels:
    app: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```
Apply and check PVC
```
kubectl apply -f nfs-pvc.yaml
```
```
kubectl get pv
```
```
kubectl get pvc
```

### Task 5: Create a Deployment using PVC
Create a deployment YAML
```
vi nfs-deployment.yaml
```
Add the following code

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-pod
  template:
    metadata:
      labels:
        app: nfs-pod
    spec:
      volumes:
      - name: nfs-pv-storage
        persistentVolumeClaim:
           claimName: nfs-pvc
      containers:
      - name: nfs-container
        image: nginx
        ports:
          - containerPort: 80
        volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: nfs-pv-storage

```
Apply and verify
```
kubectl apply -f nfs-deployment.yaml
```
```
kubectl get pods
```
```
kubectl describe pod <pod-name>
```

**🔎 Verification**

Enter pod:
```
kubectl exec -it <pod-name> -- bash
```

Inside pod:

Create file

```
echo "This is NFS Server LAB" > /usr/share/nginx/html/index.html
```
```
exit
```

Exit and verify on **NFS server**

On NFS server, check storage directory:
```
ls /mydbdata
```

✅ You should see *WebApp data files* stored inside */mydbdata* on the NFS server.

### 🧹 Task 6:  Cleanup 
```
kubectl delete -f nfs-deployment.yaml
```
```
kubectl delete -f nfs-pvc.yaml
```
```
kubectl delete -f nfs-pv.yaml
```
