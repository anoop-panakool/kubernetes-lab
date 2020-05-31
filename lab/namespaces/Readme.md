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
We’ll start by deploying the [go-demo-2.yml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/lab/namespaces/go-demo-2.yml) application and use it to explore Namespaces.

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

    kubectl create ns testing
    kubectl get ns

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






The `--selector` option can be abbreviated to `-l`, so to select pods that are
labelled with `env=development`, do:

```bash
$ kubectl get pods -l env=development
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

Oftentimes, Kubernetes objects also support set-based selectors.
Let's launch POD named [labelexother.yaml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/Labs/Labels-and-Selectors/labelexother.yaml)
that has two labels (`env=production` and `owner=shivam`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: labelexother
  labels:
    env: production
    owner: shivam
spec:
  containers:
  - name: simpleservice
    image: quay.io/openshiftlabs/simpleservice:0.5.0
    ports:
    - containerPort: 9876
```


```bash
$ kubectl apply -f labelexother.yaml
```

Now, let's list all pods that are either labelled with `env=development` or with
`env=production`:

```bash
$ kubectl get pods -l 'env in (production, development)'
NAME           READY     STATUS    RESTARTS   AGE
labelex        1/1       Running   0          43m
labelexother   1/1       Running   0          3m
```

Other verbs also support label selection, for example, you could
remove both of these pods with:

```bash
$ kubectl delete pods -l 'env in (production, development)'
```

Beware that this will destroy any pods with those labels.

You can also delete them directly, via their names, with:

```bash
$ kubectl delete pods labelex

$ kubectl delete pods labelexother
```

Note that labels are not restricted to pods. In fact you can apply them to
all sorts of objects, such as nodes or services.

###  Create a new pod with two labels.
Let's create a POD named [kubia-manual-with-labels.yaml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/Labs/Labels-and-Selectors/kubia-manual-with-labels.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual
    env: prod
    app: kubia
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
```
```
kubectl create -f kubia-manual-with-labels.yaml
```
The kubectl get pods command doesn’t list any labels by default, but you can see them by using the --show-labels
```
kubectl get po --show-labels
```
Instead of listing all labels, we are interested in certain labels, you can specify them with the -L  and have  List pods again
```
kubectl get po -L creation_method,env
```
#### Modify labels of existing pods
> Labels can also be added to and modified on existing pods.

Add the `creation_method=manual` label to `kubia-manual` pod
```
kubectl label po kubia-manual creation_method=manual
```
Change the `env=prod` label to `env=debug` on the `kubia-manual-v2` pod
> NOTE: You need to use the `--overwrite` option when changing existing labels.
```
kubectl label po kubia-manual-v2 env=debug --overwrite
```
List the pods again to see the updated labels:
```
kubectl get po -L creation_method,env
```

#### Listing pods using a label selector
To see all pods you created manually (you labeled them with creation_method=manual), do the following
```
kubectl get po -l creation_method=manual
```
List all pods that include the env label, whatever its value is
```
kubectl get po -l env
```
List all the pods that don’t have the env label
```
kubectl get po -l '!env'
```
> NOTE Make sure to use single quotes around !env, so the bash shell doesn’t evaluate the exclamation mark.

Select the pods with the creation_method label with any value other than manual
```
kubectl get po -l creation_method!=manual
```
Select pods with the env label set to either prod or devel
```
kubectl get pods -l 'env in (prod, devel)'
```
Select pods with the env label set to any value other than prod or devel
```
kubectl get pods -l 'env notin (prod, devel)'
```
Select all pods those have `app=pc`& `rel=betalabel` by using `selector`
```
kubectl get pods --selector app=pc,rel=beta
```
## Using labels and selectors to constrain pod scheduling

Using a label selector to schedule a pod named []to a specific node: kubia-gpu.yaml
> You’ve added a `nodeSelector` field under the `spec` section. When you create the pod, the scheduler will only choose among the nodes that contain the `gpu=true` label (which is only a single node in your case).

Step One : Attach label to the node
> Run `kubectl get nodes` to get the names of your cluster’s nodes.
```
kubectl get nodes
```
> `kubectl label nodes <node-name> <label-key>=<label-value>` to add a label to the node you’ve chosen. For example, if my node name is ‘shivamjha-k8s-3cudh’ and my desired label is ‘disktype=ssd’, then I can run 
```
kubectl label nodes shivamjha-k8s-3cudh disktype=ssd
```
> Now verify that it worked by re-running `kubectl get nodes --show-labels` and check that the node now has a label.
```
kubectl get nodes --show-labels
```
Step Two: Add a `nodeSelector` field to your pod `pod-nginx.yaml` configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
```
> Then add a `nodeSelector` under spec sction in `pod-nginx.yaml` file like as below: 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```
> Then run `kubectl apply -f pod-nginx.yaml` Pod will get scheduled on the node that you attached the label to.    You can verify that it worked by running `kubectl get pods -o wide` and looking at the “NODE” that the Pod was assigned to.
```
kubectl get pods -o wide
```
### Add the label `gpu=true` to one of your worker nodes (`kubectl get nodes`)
```
kubectl label node [Node-Name] gpu=true
```
List only nodes that include the label `gpu=true`
```
kubectl get nodes -l gpu=true
```
Create the Pod named kubia-gpu from file [kubia-gpu.yaml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/Labs/Labels-and-Selectors/kubia-gpu.yaml)

```
kubectl create -f kubia-gpu.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-gpu
spec:
  nodeSelector:
    gpu: "true"
  containers:
  - image: luksa/kubia
    name: kubia
```
![Label-microservice](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/Label-microservice.png)

## Scheduling the POD to one specific node


### Add a label to a node
    ```shell
    kubectl get nodes --show-labels
    ```
Chose one of your nodes, and add a label to it:

    ```shell
    kubectl label nodes <your-node-name> disktype=ssd
    ```
 where `<your-node-name>` is the name of your chosen node.

Verify that your chosen node has a `disktype=ssd` label:

```shell
    kubectl get nodes --show-labels
```

 The output is similar to this:

```shell
    NAME      STATUS    ROLES    AGE     VERSION        LABELS
    worker0   Ready     <none>   1d      v1.16.0        ...,disktype=ssd,kubernetes.io/hostname=worker0
    worker1   Ready     <none>   1d      v1.16.0        ...,kubernetes.io/hostname=worker1
    worker2   Ready     <none>   1d      v1.16.0        ...,kubernetes.io/hostname=worker2
 ```

In this output, you can see that the `worker0` node has a `disktype=ssd` label.

### Create a pod that gets scheduled to your chosen node

This pod configuration file describes a pod that has a node selector, `disktype: ssd`. This means that the pod will get scheduled on a node that has a `disktype=ssd` label.

Create the Pod named `nginx`from file [pod-nginx.yaml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/Labs/Labels-and-Selectors/pod-nginx.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

```shell
    kubectl apply -f pod-nginx.yaml
```

Verify that the pod is running on your chosen node:
```shell
    kubectl get pods --output=wide
             or
    kubectl get pods -o wide   
```
The output is similar to this:
    
```shell
    NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
    nginx    1/1       Running   0          13s    10.200.0.4   worker0
```
### Create a pod that gets scheduled to specific node
> You can also schedule a pod to one specific node via setting `nodeName`

Create the Pod named `nginx`from file [pod-nginx-specific-node.yaml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/Labs/Labels-and-Selectors/pod-nginx-specific-node.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: shivamjha-k8s-3cudh # Change the node name as per your cluster setup and schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```
```
kubectl create -f pod-nginx-specific-node.yaml
```
```
kubectl get pods nginx -o wide
```