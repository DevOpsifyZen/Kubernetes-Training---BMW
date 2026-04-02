
# Kubernetes Volumes 

## 📌 Introduction

In Kubernetes, containers are **ephemeral by nature**, meaning any data stored inside a container is lost when the container stops or is deleted.

To solve this limitation, Kubernetes provides **Volumes**, which enable:
- Data persistence
- Data sharing between containers
- Decoupling of storage from container lifecycle

---

## What is a Kubernetes Volume?

A **Volume** is a directory that is:
- Accessible to containers in a Pod
- Backed by different storage types
- Managed by Kubernetes

## Types of Volumes (Focus)

This guide focuses on two commonly used volume types:

1. **hostPath Volume**
2. **emptyDir Volume**

---

# hostPath Volume

A **hostPath volume** mounts a file or directory from the **host node’s filesystem** into a Pod.

## How It Works

- Kubernetes directly maps a path from the node (e.g., `/data`)
- This path becomes accessible inside the container
- Any changes made inside the container reflect on the node

## Key Characteristics

- **Node-dependent**: Works only on the specific node where the Pod runs
- **Persistent**: Data remains even after Pod deletion
- **Direct access**: Uses the host machine’s storage
- **No abstraction**: Bypasses Kubernetes storage layers

## Advantages

- Simple and easy to use
- Useful for testing and development
- Enables access to node-level resources

## Limitations

- Not portable across nodes
- Security risks (access to host filesystem)
- Not suitable for production environments
- Tight coupling with node infrastructure

---

# emptyDir Volume

An **emptyDir volume** is a temporary directory created when a Pod is assigned to a node.

## How It Works

- Created automatically when Pod starts
- Exists as long as the Pod is running
- Deleted permanently when Pod is removed

## Key Characteristics

- **Pod-level scope**
- **Ephemeral storage**
- **Shared across containers in the same Pod**
- Can be stored on:
  - Node disk
  - Memory (if configured)

## Advantages

- Simple temporary storage
- Ideal for inter-container communication
- No dependency on external storage
  
## Limitations

- Data is lost when Pod stops or restarts
- Not suitable for persistent storage
- Limited to a single Pod


---

# ⚖️ hostPath vs emptyDir

| Feature        | hostPath                     | emptyDir                   |
|---------------|-----------------------------|----------------------------|
| Scope         | Node                        | Pod                        |
| Persistence   | Persistent                  | Temporary                  |
| Lifecycle     | Independent of Pod          | Tied to Pod lifecycle      |
| Portability   | Low                         | High (within Pod)          |
| Security      | Risky                       | Safer                      |
| Use Case      | Node-level access           | Temporary shared storage   |


---

## Task 1: Creating POD with hostPath
Create a file named mydep-hp.yaml using the content given below
```
vi mypod-hp.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod-hp
spec:
 volumes:
 - name: myvol
   hostPath:
    path: /data
 containers:
  - image: nginx:latest
    name: con1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: myvol
      mountPath: /app
  ```
Save and exit the editor and Apply the Deployment yaml created in the previous step
```
kubectl apply -f mypod-hp.yaml
```
View the objects created by Kubernetes and verify hostPath mounting in POD
```
kubectl get pods -o wide
```
From the above take a note on which pods are running on which particular node, as this would be required in the below steps.
```
kubectl describe pod <pod-name>
```
Access shell on a container running in your Pod to verify volume
```
kubectl exec -it <pod-name> -- /bin/bash
```
```
cd /app
```
```
echo "Welcome to DevOps Training" > index.html
```
```
exit
```
Now delete your pod
```
kubectl delete -f mypod-hp.yaml --force
```
Now ssh into the Node on which the particular pod( in which you went inside and created the index.html file) was running.
```
ssh ubuntu@<External-IP>
```
Navigate to the Source location. In this case /data
```
cd /data
```
```
ls -la
```
You would see the index.html file still existing
```
cat index.html
```
```
exit
```


## Task 2 : Creating Deployment with emptyDir
Create a file named mydep-empty.yaml using content given below
```
vi mydep-empty.yaml
```
Paste the following content into the file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydep-empty
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mydep
  template:
    metadata:
      labels:
        app: mydep
    spec:
      containers:
      - image: nginx:latest
        name: con1
        ports:
        - containerPort: 80
        volumeMounts:
        - name: empty-vol
          mountPath: /data
      - image: tomcat:latest
        name: con2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: empty-vol
          mountPath: /opt/data
      volumes:
      - name: empty-vol
        emptyDir: {}
  ```
Save and exit the editor
Apply the Deployment yaml created in the previous step
```
kubectl apply -f mydep-empty.yaml
```
View the objects created by Kubernetes Deployment and verify hostPath mounting in POD
```
kubectl get deployments
```
```
kubectl get pods
```
```
kubectl describe pod <pod-name>
```
Access shell on a container running in your Pod to verify volume
```
kubectl exec -it <pod-name> -c <ctr-name> -- /bin/bash
```
```
kubectl exec -it <pod-name> -c <ctr-name> -- /bin/bash
```
## Task 3: Cleanup the resources using the below command
Delete the resources created during the lab:
```
kubectl delete -f mydep-empty.yaml
```
```
kubectl delete -f mydep-hp.yaml
```
