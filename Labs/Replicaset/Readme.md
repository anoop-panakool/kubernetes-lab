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
![go-demo-2.yml](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/go-demo-2.png)