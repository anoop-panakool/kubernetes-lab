### Labels & Selectors 

Labels are the mechanism you use to organize Kubernetes objects. A label is a key-value
pair with certain [restrictions](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)
concerning length and allowed values but without any pre-defined meaning.
So you're free to choose labels as you see fit, for example, to express
environments such as 'this pod is running in production' or ownership,
like 'department X owns that pod'.

Let's create a POD named [labelex.yaml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/Labs/Labels-and-Selectors/labelex.yaml) that initially has one label (`env=development`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: labelex
  labels:
    env: development
spec:
  containers:
  - name: simpleservice
    image: quay.io/openshiftlabs/simpleservice:0.5.0
    ports:
    - containerPort: 9876
```

```bash
$ kubectl apply -f labelex.yaml

$ kubectl get pods --show-labels
NAME       READY     STATUS    RESTARTS   AGE    LABELS
labelex    1/1       Running   0          10m    env=development
```
In above `get pods` command note the `--show-labels` option that output the
labels of an object in an additional column.

You can add a label to the pod as:

```bash
$ kubectl label pods labelex owner=michael

$ kubectl get pods --show-labels
NAME        READY     STATUS    RESTARTS   AGE    LABELS
labelex     1/1       Running   0          16m    env=development,owner=shivam
```

To use a label for filtering, for example to list only pods that have an
`owner` that equals `shivam`, use the `--selector` option:

```bash
$ kubectl get pods --selector owner=shivam
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

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

Add the label `gpu=true` to one of your worker nodes (`kubectl get nodes`)
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
> You can also schedule a pod to one specific node via setting nodeName

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