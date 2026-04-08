**🔐 Security Context***

A *security context* in Kubernetes defines security-related settings for a Pod or a Container.
It tells Kubernetes how processes inside the Pod/Container should run (e.g., as which user, with what permissions, and with what restrictions).

**📌 Key things you can control with Security Context**

**User & Group IDs**

* *runAsUser:* Which user ID (UID) processes should run as.
* *runAsGroup:* Which group ID (GID) processes should run as.
* *fsGroup:* Group ID applied to files in mounted volumes.

**Privilege settings**

* *allowPrivilegeEscalation:* Prevents a process from gaining more privileges than its parent process.

* *privileged:* Allows running the container in privileged mode (like root access to the host).

**Capabilities**

* Add or drop Linux capabilities (fine-grained root privileges).
* Example: *NET_ADMIN* , *SYS_ADMIN* etc..


---------------------------------------------------------------

### Task 1: Set the security context for a Pod

To specify security settings for a Pod, include the securityContext field in the Pod specification. 

The security settings that you specify for a Pod apply to all Containers in the Pod. Here is a configuration file for a Pod that has a securityContext and an emptyDir volume:

```
vi security-context1.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod1
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-pod1-ctr
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
   
```
save the file using `ESCAPE + :wq!`

Apply the yaml
```
kubectl apply -f security-context1.yaml
```
Verify that the Pod's Container is running:
```
kubectl get pod security-context-pod1
```
Get a shell to the running Container:
```
kubectl exec -it security-context-pod1 -- sh
```
In your shell, list the running processes:
```
ps
```
The output shows that the processes are running as user 1000, which is the value of runAsUser:
```
id
```
The Output shows the UID, Group ID and all the groups

In your shell, navigate to /data, and list the one directory:
```
cd /data && ls -l
```
The output shows that the /data/demo directory has group ID 2000, which is the value of fsGroup.

In your shell, navigate to /data/demo, and create a file:
```
cd demo && echo hello > testfile
```
List the file in the /data/demo directory:
```
ls -l
```
The output shows that testfile has group ID 2000, which is the value of fsGroup.

Run the following command:
```
id
```

From the output, you can see that gid is 3000 which is same as the runAsGroup field. If the runAsGroup was omitted, the gid would remain as 0 (root) and the process will be able to interact with files that are owned by the root(0) group and groups that have the required group permissions for the root (0) group.
```
exit
```


## TAsk 2 : Pod & Container Level Security Context
```
vi security-context-new.yaml
```
Add the given content, by pressing `INSERT`

```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-new
spec:
  securityContext:
    fsGroup: 2000     # common group of files
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: ctr-1
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      runAsUser: 1010   #UID
      runAsGroup: 3010  #GID
  - name: ctr-2
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /docs/demo
    securityContext:
      runAsUser: 1020   #UID
      runAsGroup: 3020  #GID
```
save the file using `ESCAPE + :wq!`

Apply the yaml
 
```
kubectl apply -f security-context-new.yaml
```
Verify that the Pod's Containers are running:
```
kubectl get po
```
Get a shell into the ctr-1 Container:
```
kubectl exec -it security-context-new -c ctr-1 -- sh
```
```
id
```
```
cd /data/demo/
```
```
echo hello > file1.txt
```
```
ls -l
```
```
exit
```
Get a shell into the ctr-2 Container:
```
kubectl exec -it security-context-new -c ctr-2 -- sh
```
```
id
```
```
cd /docs/demo/
```
```
echo hello > file2.txt
```
```
ls -l
```
```
exit
```
### Task 3: Cleanup the resources 
```
kubectl delete -f security-context1.yaml
```
```
kubectl delete -f security-context-new.yaml
```

------------------------------------------------------------------

## Security Context (Part 2)


**Privilege Escalation**: Prevents a container from gaining higher privileges (like root).  

  ```yaml
  securityContext:
    allowPrivilegeEscalation: false
  ```

**Linux Capabilities**: Provide fine-grained permissions instead of full root access.

```yaml
securityContext:
  capabilities:
    drop: ["ALL"]
    add: ["NET_ADMIN"]
```

**Key Idea:**
- ***Privilege Escalation →*** controls gaining extra privileges
- ***Capabilities →*** controls specific permissions

**Best Practice:**
Run as non-root, disable escalation, drop all capabilities, and add only what is required.

### Task : Container Security Context Override

Below is the configuration file for a Pod that has one Container. Both the Pod and the Container have a securityContext field:
```
vi security-context-2.yaml
```
```yaml

apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod2
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx-pod2
    image: alpine:3
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```
Create the Pod:
```
kubectl apply -f security-context-2.yaml
```
Verify that the Pod's Container is running:
```
kubectl get pod security-context-pod2
```
Get a shell into the running Container:
```
kubectl exec -it security-context-pod2 -- sh
```
In your shell, list the running processes:
```
ps aux
```
The output shows that the processes are running as user 2000. This is the value of runAsUser specified for the Container. It overrides the value 1000 that is specified for the Pod.

> ***Process runs as UID 2000 → container overrides pod-level setting***

### Task : Set capabilities for a Container

With Linux capabilities, you can grant certain privileges to a process without granting all the privileges of the root user.

To add or remove Linux capabilities for a Container, include the capabilities field in the securityContext section of the Container manifest.


Lets check, what happens when you don't include a capabilities field.
```
vi security-context-3.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod3
spec:
  containers:
  - name: sec-ctx-3
    image: alpine:3
    command: ["sh", "-c", "sleep 7000"]
```
Create the Pod:
```
kubectl apply -f security-context-3.yaml
```
Verify that the Pod's Container is running:
```
kubectl get pod security-context-pod3
```
Get a shell into the running Container:
```
kubectl exec -it security-context-pod3 -- sh
```
In your shell, list the running processes:
```
ps aux
```
In your shell, view the status for process 1:
```
cd /proc/1 && cat status
```
The output shows the capabilities bitmap for the process. Make a note of the capabilities bitmap. 

Inside the pod, you can try to manipulate network-related settings, which require NET_ADMIN capability.
```
ip link set dev eth0 down
```
If you're able to execute this command without any errors, it indicates that the NET_ADMIN capability is working, allowing you to manipulate network interfaces.

You can also try to manipulate system time, which requires SYS_TIME capability.
```
date -s "2024-02-07 12:00:00"
```
If you're able to execute this command without any errors, it indicates that the SYS_TIME capability is working, allowing you to manipulate system time.
```
exit
```
Now, run a Container that is the same as the preceding container, except that it has additional capabilities set.

```
vi security-context-4.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod4
spec:
  containers:
  - name: sec-ctx-4
    image: alpine:3
    command: ["sh", "-c", "sleep 7000"]
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```
```
kubectl apply -f security-context-4.yaml
```
```
kubectl exec -it security-context-pod4 -- sh
```
In your shell, view the capabilities for process 1:
```
cd /proc/1 && cat status
```
Compare the capabilities of the two Containers:

In the capability bitmap of the first container, bits 12 and 25 are clear. In the second container, bits 12 and 25 are set. Bit 12 is CAP_NET_ADMIN, and bit 25 is CAP_SYS_TIME. 

Note: Linux capability constants have the form CAP_XXX. But when you list capabilities in your container manifest, you must omit the CAP_ portion of the constant. For example, to add CAP_SYS_TIME, include SYS_TIME in your list of capabilities.

Inside the pod, you can try to manipulate network-related settings, which require NET_ADMIN capability.
```
ip link set dev eth0 down
```
If you're able to execute this command without any errors, it indicates that the NET_ADMIN capability is working, allowing you to manipulate network interfaces.

You can also try to manipulate system time, which requires SYS_TIME capability.
```
date -s "2024-02-07 12:00:00"
```
If you're able to execute this command without any errors, it indicates that the SYS_TIME capability is working, allowing you to manipulate system time.
```
exit
```



