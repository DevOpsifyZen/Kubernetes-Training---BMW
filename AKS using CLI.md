



## 🧪 Lab Guide: Create AKS Cluster Using Azure Cloud Shell

🎯 Objective:
Use Azure Cloud Shell to create a Resource Group, deploy an AKS cluster, connect to it, and verify that it's working.

🛠️ Prerequisites:
An active Azure subscription

Browser access to https://portal.azure.com

###  ✅ Step 1: Open Azure Cloud Shell
Open your browser and go to 👉 https://portal.azure.com

* In the top-right corner, click the Cloud Shell icon (🖥️ >_).

* If prompted:
*   Select Bash.
*   Choose Create Storage if required (first-time users only).
*   Wait a few seconds for the shell to initialize.

### ✅ Step 2: Set Your Environment Variables
📘 These variables help you reuse values without typing them again.

👨‍💻 In Cloud Shell, type:
```
RESOURCE_GROUP="K8S-RG"
CLUSTER_NAME="aks-cluster"
LOCATION="centralus"
```
✅ You’ve now stored your desired group name, cluster name, and region in memory.

### ✅ Step 3: Create a Resource Group
📘 A resource group is a container that holds related Azure resources.

👨‍💻 Type the following command:
```
az group create --name $RESOURCE_GROUP --location $LOCATION
```

✅ Expected output:
A JSON response confirming that the resource group K8S-RG has been created.

### ✅ Step 4: Create the AKS Cluster
📘 Now, create the actual Kubernetes cluster with 2 worker nodes.

👨‍💻 Type:
```
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --node-count 2 \
  --node-vm-size Standard_DS2_v2 \
  --generate-ssh-keys \
  --location $LOCATION
```
⏳ This takes about 3–7 minutes.

✅ You’ll see details like fqdn, kubeConfig, etc., once the cluster is created.

###  ✅ Step 5: Connect to the AKS Cluster
📘 You need to connect kubectl to your new cluster.

👨‍💻 Type:
```
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```
✅ You’ll see a message like:

Merged "aks-cluster" as current context in /home/<user>/.kube/config

### ✅ Step 6: Verify Your Cluster is Running
📘 This command lists all nodes in your AKS cluster.

👨‍💻 Type:
```
kubectl get nodes
```
✅ You should see 2 nodes with STATUS = Ready.

🧪  Lab: Namespaces, Pods, YAMLs, and Labels

### ✅ Step 6: (Optional) Delete Cluster

Verify Existing AKS Clusters
```bash
az aks list -o table
```
Delete the AKS Cluster
```bash
az aks delete --resource-group <ResourceGroupName> --name <ClusterName> --yes --no-wait
```
Delete Resource Group
```bash
az group delete --name K8S-RG --yes --no-wait
```
⚠️ This will delete all resources inside K8S-RG, not just AKS.
