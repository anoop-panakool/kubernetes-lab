## Namespaces
### *A ways to split a cluster into different segments as an alternative to having multiple clusters*

### Why to create Multiple Clusters?

Applications and their objects often need to be separated from each other to avoid conflicts and other undesired effects.

### Benefits of creating muliple cluster: 

- We might need to separate objects created by different teams. We can, for example, give each team a separate cluster so that they can “experiment” without affecting others.

- We might want to create different clusters that will be used for various purposes. For example, we could have a production and a testing cluster.

- We might be afraid that a team will accidentally replace a production release of an application with an untested beta.

- We might be concerned that performance tests will slow down the whole cluster.

### What is the Problem with Multiple Clusters?

- The problem with having many Kubernetes clusters is that each has an operational and resource overhead. Managing one cluster is often very hard. Having a few is complicated. Having many can become a nightmare and requires quite a significant investment in hours dedicated to operations and maintenance.

- If that overhead is not enough, we must also be aware that each cluster needs resources dedicated to Kubernetes. The more clusters we have, the more resources (CPU, memory, IO) are spent. While that can be said for big clusters as well, the fact remains that the resource overhead of having many smaller clusters is higher than having a single big one.

## Getting the Existing Namespaces
```
kubectl get namespace
```
> **The outputis as follows.**
```
    NAME              STATUS   AGE
    default           Active   23m
    kube-node-lease   Active   23m
    kube-public       Active   23m
    kube-system       Active   23m
```
### The Default Namespace 

The default Namespace is the one we used all this time. If we do not specify any other namespace, all the kubectl commands will operate against the objects in the default Namespace. That’s where our go-demo-2 application is running. Even though we were not aware of its existence, we now know that’s where the objects we created are placed.

![namespaces-with-pod](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/namespaces-with-pod.png)

To specify a Namespace use `-ns` or `--namespace` argument for all `kubectl` commands

### The kube-public Namespace
    
kube-public This namespace is created automatically and is readable by all users (including those not authenticated) from all Namespaces. This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster.

```bash
    kubectl --namespace kube-public get all
```
> **The primary reason for kube-public's existence is to provide space where we can create objects that should be visible throughout the whole cluster.**

> **Anexample is ConfigMaps. When we create one in, let’s say, the default Namespace, it is accessible only by the other objects in the same Namespace. Those residing somewhere else would be oblivious of its existence. If we’d like such a ConfigMap to be visible to all objects no matter where they are, we’d put it into the kube-public Namespace instead. We won’t use this Namespace much (if at all).**

### The kube-system Namespace

The kube-system Namespace is critical.

> Almost all the objects and resources Kubernetes needs are running inside kube-system Namespace.

Check that by executing the command as follows.
```
    kubectl --namespace kube-system get all
```
> **The outputis as follows.**

```yaml
kubectl --namespace kube-system get all
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/coredns-66bff467f8-h9hx2                           1/1     Running   5          32h
pod/coredns-66bff467f8-nrzfr                           1/1     Running   1          9h
pod/etcd-master                                        1/1     Running   12         3d1h
pod/kube-apiserver-master                              1/1     Running   12         3d1h
pod/kube-controller-manager-master                     1/1     Running   12         3d1h
pod/kube-flannel-ds-amd64-8b45j                        1/1     Running   15         3d1h
pod/kube-flannel-ds-amd64-llmrn                        1/1     Running   13         3d1h
pod/kube-flannel-ds-amd64-njr9z                        1/1     Running   13         3d1h
pod/kube-proxy-lxxb8                                   1/1     Running   12         3d1h
pod/kube-proxy-pptlt                                   1/1     Running   12         3d1h
pod/kube-proxy-sb7rg                                   1/1     Running   12         3d1h
pod/kube-scheduler-master                              1/1     Running   12         3d1h
pod/kubernetes-dashboard-57576c4679-vpcjb              1/1     Running   1          8h

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   3d1h
service/kubernetes-dashboard   NodePort    10.109.158.56   <none>        80:31000/TCP             8h

NAME                                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-flannel-ds-amd64     3         3         3       3            3           <none>                   3d1h
daemonset.apps/kube-flannel-ds-arm       0         0         0       0            0           <none>                   3d1h
daemonset.apps/kube-flannel-ds-arm64     0         0         0       0            0           <none>                   3d1h
daemonset.apps/kube-flannel-ds-ppc64le   0         0         0       0            0           <none>                   3d1h
daemonset.apps/kube-flannel-ds-s390x     0         0         0       0            0           <none>                   3d1h
daemonset.apps/kube-proxy                3         3         3       3            3           kubernetes.io/os=linux   3d1h

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns                2/2     2            2           3d1h
deployment.apps/kubernetes-dashboard   1/1     1            1           8h

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-66bff467f8                2         2         2       3d1h
replicaset.apps/kubernetes-dashboard-57576c4679   1         1         1       8h

```

## Creating a New Namespace

1. Create a new YAML file called `my-namespace.yaml` with the contents:

    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: <insert-namespace-name-here>
    ```
    Then run:
   
    ```
    kubectl create -f ./my-namespace.yaml
    ```

2. Alternatively, you can create namespace using below command:

    ```
    kubectl create namespace <insert-namespace-name-here>

    kubectl create namespace testing
    kubectl get ns
    ```
> **The outputis as follows.**
```
NAME              STATUS   AGE
default           Active   4h36m
kube-node-lease   Active   4h36m
kube-public       Active   4h36m
kube-system       Active   4h36m
testing           Active   3s

```
We can see that the new Namespace testing was created.

## Deleting a namespace

Delete a namespace with

```shell
kubectl delete namespaces <insert-some-namespace-name>
```
> NOTE: Deleting Namespace will `deletes _everything_` under the namespace!


We will continue using the --namespace argument to operate within the newly created Namespace. 
However, writing --namespace with every command is tedious. Instead, we’ll create a new context.
```
kubectl config set-context testing --namespace testing --cluster kubernetes --user kubernetes-admin
```
We created a new context called testing. It is the same as the kubernetes context, except that it uses the testing Namespace.

You can use the config view command, to view the config file
```
kubectl config view
```
> **The outputis as follows.**

```json
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.31.106.125:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
- context:
    cluster: kubernetes
    namespace: testing
    user: kubernetes-admin
  name: testing
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```
We can see that there are two contexts. Both are set to use the same kubernetes cluster with the same kubernetes-admin user. The only difference is that one does not have the Namespace set, meaning that it will use the default. The other has it set to testing.

### Context Switching
Now that we have two contexts, we can switch to testing.

```bash
kubectl config use-context testing
```
  We switched to the testing context that uses the Namespace of the same name. From now on, all the kubectl commands will be executed within the context of the testing Namespace. That is, until we change the context again, or use the --namespace argument.

### Verification

```
kubectl get all
```
The output shows that no resources were found.

### Creating a Pod

We’ll create an alpine-based Pod that we’ll use to demonstrate communication between Namespaces.
```
kubectl config use-context kubernetes-admin@kubernetes

kubectl run test --image=alpine --generator "run-pod/v1" sleep 1000

kubectl get pod test
```
> **The outputis as follows.**
```
NAME READY STATUS  RESTARTS AGE
test 1/1   Running 0        10m
```

### Establishing the Communication 

Before we proceed, we’ll install curl inside the container in the test Pod.
```
kubectl exec -it test -- apk add -U curl
```
We’ll start by deploying the [go-demo-2.yml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/lab/namespaces/go-demo-2.yml) application and use it to explore Namespaces.

Since the test Pod is running in the default Namespace, we can, for example, reach the go-demo-2-api Service by using the Service name as a DNS name.

```
kubectl exec -it test -- curl "http://go-demo-2-api:8080/demo/hello"
```
> **The outputis as follows.**
```
hello, release 1.0
```



## Quick Quiz!

Let's test our understanding of Namespaces.

1. We can have multiple running Kubernetes cluster without any resource and operational overhead.
A)
True
B)
False

2. Which of the following command we can use for altering the Kubernetes definitions?
A)
alter
B)
sed
C)
cha


3. In Kubernetes virtual clusters inside the clusters are created using?
A)
Volumes
B)
Sevices
C)
Namespaces

4. If we do not specify otherwise, against the objects of which Namespace all the kubectl commands will operate?
A)
kube-public
B)
kube-system
C)
default
D)
user


5. The objects created inside the kube-public Namespace are visible?
A)
Throughout the cluster
B)
To specified Namespaces only
C)
To other clusters as well

6. The purpose of the existance of DNSes with Namespaces is to reach the services in other Namespaces.
A)
True
B)
False


7. What happens when we delete a Namespace?
A)
Everything inside it gets deleted
B)
Only partial objects inside it get deleted
C)
Nothing happens to the objects inside it