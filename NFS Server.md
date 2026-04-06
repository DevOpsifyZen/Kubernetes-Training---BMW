## 📘 Lab: Configuring Pod Using NFS-Based PV and PVC
🎯 Objective

To configure a Pod using NFS-based PersistentVolume (PV) and PersistentVolumeClaim (PVC) for efficient storage management in Kubernetes.

### 🔹 Task 1: Configure the NFS Kernel Server

Perform these steps on **worker-node-1** (which will act as the NFS server):

Create a directory
```
sudo mkdir /mydbdata
```
Install the NFS kernel server
```
sudo apt install -y nfs-kernel-server
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

### 🔹 Task 2: Configure the NFS Common on Client Machines

Perform the following steps on all worker nodes:

Install NFS common
```
sudo apt install -y nfs-common
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

### 🔹 Task 3: Create the PersistentVolume (PV)
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
    storage: 10Gi
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

### 🔹 Task 4: Create the PersistentVolumeClaim (PVC)
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

### 🔹 Task 5: Create a Deployment using PVC
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

🔎 Verification

Enter pod:
```
kubectl exec -it <pod-name> -- bash
```

Inside pod:

```
echo "This is NFS Server LAB" > /usr/share/nginx/html/index.html
```

On NFS server, check storage directory:
```
ls /mydbdata
```

✅ You should see WebApp data files stored inside /mydbdata on the NFS server.

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
