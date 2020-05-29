# ReplicaSets

#### Most applications should be scalable and all must be fault tolerant. Pods do not provide those features, ReplicaSets do.

## Prerequisites

- Cluster should be in Ready state

## Instructions

Let’s take a look at a `ReplicaSet` based on the Pod defination file [go-demo-2.yml](/Labs/Replicaset/go-demo-2.yml)

![go-demo-2.yml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/go-demo-2.png)

> #### Note : The `apiVersion`, `kind`, and `metadata` fields are mandatory with all Kubernetes objects. `ReplicaSet` is no exception, i.e., it is also a Kubernetes object.

### Create the ReplicaSet 
```
kubectl create -f go-demo-2.yml
```
> We got the response that the replicaset "go-demo-2" was created.

Confirm that by listing all the ReplicaSets in the cluster.
```bash
kubectl get rs
```
> The **output** is as follows.

```bash
NAME        DESIRED   CURRENT   READY   AGE
go-demo-2   2         2         0       12s
```
Instead of retrieving all the replicas in the cluster, we can retrieve those specified in defination file `go-demo-2.yml`

```bash
kubectl get -f go-demo-2.yml
```

Describle the `Replicaset` with kubectl describe command.
```
kubectl describe -f go-demo-2.ym
```
> The **output** is as follows.
```yaml
root@master:~/# kubectl describe -f go-demo-2.yml 
Name:         go-demo-2
Namespace:    default
Selector:     service=go-demo-2,type=backend
Labels:       <none>
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  db=mongo
           language=go
           service=go-demo-2
           type=backend
  Containers:
   db:
    Image:        mongo:3.3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
   api:
    Image:      vfarcic/go-demo-2
    Port:       <none>
    Host Port:  <none>
    Liveness:   http-get http://:8080/demo/hello delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:
      DB:    localhost
    Mounts:  <none>
  Volumes:   <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  60s   replicaset-controller  Created pod: go-demo-2-f4x2h
  Normal  SuccessfulCreate  60s   replicaset-controller  Created pod: go-demo-2-87djm

```
> We can see that ReplicaSet created two Pods while trying to match the desired state with the actual state.

 If you want to sure that ReplicaSet created the Pods, run below command in the cluster and confirm
 ```
 kubectl get pods --show-labels
 ```
> The **output** is as follows.
```
root@master:~/# kubectl get pods --show-labels

NAME              READY   STATUS    RESTARTS   AGE     LABELS
go-demo-2-87djm   2/2     Running   0          22m     db=mongo,language=go,service=go-demo-2,type=backend
go-demo-2-f4x2h   2/2     Running   0          22m     db=mongo,language=go,service=go-demo-2,type=backend
```
![db1-yaml.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/db1-yaml.png)

## **Sequential breakdown of the Process followed to create the ReplicaSet**

The sequence of events that happens when you run `kubectl create -f go-demo-2.yml` command is as follows.

1. Kubernetes client (kubectl) sent a request to the API server requesting the creation of a ReplicaSet defined in the `go-demo-2.yml` file.

2. The controller is watching the API server for new events, and it detected that there is a new ReplicaSet object.

3. The controller creates two new pod definitions because we have configured replica value as 2 in go-demo-2.yml file.

4. Since the scheduler is watching the API server for new events,it detected that there are two unassigned Pods.

5. The scheduler decided which node to assign the Pod and sent that information to the API server.

6. Kubelet is also watching the API server.It detected that the two Pods are assigned to the node it is running on.

7. Kubelet sent requests to Docker requesting the creation of the containers that form the Pod. In our case, the Pod defines two containers based on the mongo and api image. So in total four containers are created.

8. Finally, Kubelet sent a request to the API server notifying it that the Pods were created successfully.

![sequence-replicaset](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/sequence-replicaset.png)

### The sequence described above says what happened in the cluster from the moment we requested the creation of a new ReplicaSet. 
## **We will see the same process through a diagram that more closely represents the cluster.**

![replicaset-events](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/replicaset-events.png)

## **Operating ReplicaSets & self-healing feature**

> *Deleting ReplicaSets - What would happen if we delete the ReplicaSet?*

Remove the ReplicaSet `go-demo-2` created
```
root@master:~/# kubectl delete -f go-demo-2.yml --cascade=false
replicaset.apps "go-demo-2" deleted
```
Let’s confirm that `replicaset` removed from the system.
```
root@master:~/# kubectl get rs
No resources found in default namespace.
```
If --cascade=false prevents Kubernetes from removing the dependents objects, the Pods should continue running in the cluster. Let’s confirm it.
```
kubectl get pods
```
> The **output** is as follows.
```
NAME              READY   STATUS    RESTARTS   AGE
go-demo-2-87djm   2/2     Running   0          146m
go-demo-2-f4x2h   2/2     Running   0          146m
```
The two Pods created by the ReplicaSet are still running in the cluster even though we removed the ReplicaSet.

> Note: *`ReplicaSet uses labels to decide whether the desired number of Pods is already running in the cluster,if we create the same ReplicaSet again, it should reuse the two Pods that are running in the cluster.`*

 Let’s confirm that,Create the ReplicaSet again
```
root@master:~/# kubectl create -f go-demo-2.yml --save-config
replicaset.apps/go-demo-2 created
```
> The **output** is as follows.
```
root@master:~/# kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
go-demo-2-87djm   2/2     Running   0          152m
go-demo-2-f4x2h   2/2     Running   0          152m
```
> *`If you compare the names of the Pods, you’ll see that they are the same as before we created the ReplicaSet. It looked for matching labels, deduced that there are two Pods that match them, and decided that there’s no need to create new ones. The matching Pods fulfill the desired number of replicas.`*

### Updating the Definition of ReplicaSet

Now we will use `ReplicaSet` defination file [go-demo-2-scaled.yml](/Labs/Replicaset/go-demo-2-scaled.yml) that differs only in the number of replicas set to 4.
```yaml
apiVersion:  apps/v1
kind: ReplicaSet
metadata:
  name: go-demo-2
spec:
  replicas: 4
  selector:
    matchLabels:
      type: backend
      service: go-demo-2
  template:
    metadata:
      labels:
        type: backend
        service: go-demo-2
        db: mongo
        language: go
    spec:
      containers:
      - name: db
        image: mongo:3.3
      - name: api
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: localhost
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
```
Let’s create the ReplicaSet with defination file *`go-demo-2-scaled.yml`*
```
root@master:~/# kubectl apply -f go-demo-2-scaled.yml 
replicaset.apps/go-demo-2 configured
```
This time, the output is slightly different. Instead of saying that the ReplicaSet was created, we can see that it was configured.

Let’s take a look at the Pods.
```
kubectl get pods
```
> The **output** is as follows.
```
root@master:~/# kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
go-demo-2-87djm   2/2     Running   0          173m
go-demo-2-f4x2h   2/2     Running   0          173m
go-demo-2-r4bjm   2/2     Running   0          119s
go-demo-2-vpls4   2/2     Running   0          119s
```
> As expected, now there are four Pods in the cluster. If you pay closer attention to the names of the Pods, you’ll notice that two of them are the same as before.

### Self-healing property of ReplicaSet

> Let’s test this property by making a chnages to our system.

Let’s destroy the Pod

```
root@master:~/# kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
go-demo-2-7c5fs   2/2     Running   0          17s
go-demo-2-87djm   2/2     Running   0          3h2m
go-demo-2-cfwln   2/2     Running   0          17s
go-demo-2-f4x2h   2/2     Running   0          3h2m
```
Retrive all the Pods ( used `-o name` to retrive only their names & the result was piped to tail -2 so that only two of the mames is output . Later command will use the variables to remove the pod)
```
root@master:~/# POD_NAME=$(kubectl get pods -o name | tail -2)
```
Delete the Pods captured in varibale `POD_NAME`
```
root@master:~/# kubectl delete $POD_NAME
pod "go-demo-2-cfwln" deleted
pod "go-demo-2-f4x2h" deleted
```
Let’s take another look at the Pods in the cluster
```
kubectl get pods
```
> The **output** is as follows.
```
root@master:~/# kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
go-demo-2-7c5fs   2/2     Running   0          31s
go-demo-2-87djm   2/2     Running   0          3h3m
go-demo-2-pgrr7   2/2     Running   0          6s
go-demo-2-rcf7d   2/2     Running   0          6s
```
> *`We can see that the Pod we deleted is created back. Since we have a ReplicaSet with replicas set to 4, as soon as it discovered that the number of Pods dropped to 3, it created the two new. This is what called as self-healing features.`*

> *`As long as there are enough available resources in the cluster, ReplicaSets will make sure that the specified number of Pod replicas are (almost) always up-and-running.`*