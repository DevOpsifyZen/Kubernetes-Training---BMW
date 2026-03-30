## Bootstrap a Kubernetes Cluster using Kubeadm 
Tools required: kubeadm, kubectl, kubelet, and containerd

### Task 1: Initializing the Cluster

Start kubeadm only on **master**

Switch to root.
```
sudo su
```
Run the below cmd to initialize the Cluster
```
kubeadm init --ignore-preflight-errors=all
```
If the it runs successfully, it will provide a **join command** which can be used to join the master. Make a note of the highlighted part.
Run the following commands to configure kubectl on master.
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
chown $(id -u):$(id -g) $HOME/.kube/config
```
 
### Task 2: Joining a Cluster


Run the kubeadm join command in **worker nodes**, that was previously noted from the master node in the previous task.
```
kubeadm join --token <your_token> --discovery-token-ca-cert- hash <your_discovery_token> 
```
**Note(optional)
If you want to list and generate tokens again to join worker nodes, then follow the below steps(optional)
```
kubeadm token list
```
```
kubeadm token create  --print-join-command
```

View node information on the **master**
```
kubectl get nodes
```

### Task 3: Deploy Container Networking Interface
Go to **master** node and 
Apply weave CNI (Container Network Interface) as shown below:
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
(reference link1:- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)

View nodes to see that they are ready.
```
kubectl get nodes
```

View all Pods including system Pods and see that dns and weave are running.
```
kubectl get pod -n kube-system
```


If you get the below `error` when running kubectl commands, execute the command given below.

`The connection to the server localhost:8080 was refused - did you specify the right host or port?`
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

#### *** DO NOT EXECUTE THE BELOW COMMAND UNTIL DOUBLY SURE ***
To remove the joins in your cluster
```
kubeadm reset
```


 

