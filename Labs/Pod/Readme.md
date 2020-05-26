# A Lazy and Dirty Way to Run Pods

In this lab we will create a Pod with Mongo on Kubernetes cluster.

## Prerequisites

- Cluster is in Ready state

## Instructions

1. Creating a Pod with Mongo
`
   Just as we can execute docker run to create containers, kubectl allows us to create Pods with a single command. For example, if we’d like to create a Pod with a Mongo database, the command is as follows.
`
```bash
kubectl run db --image mongo \
    --generator "run-pod/v1"
```
You’ll notice that the output says that pod/db was created. We created Pod. We can confirm that by listing all the Pods in the cluster.

2. Verification
`
We have created a Pod. We can confirm that by listing all the Pods in the cluster.
`
```bash
kubectl get pod
```
The output is as follows.
```bash
NAME   READY   STATUS              RESTARTS  AGE
db     0/1     ContainerCreating   0         1
```
```
In the output we can see:

The name of the Pod
Its readiness
The status
The number of times it restarted
For how long it has existed (its age)
If you were fast enough, or your network is slow, none of the pods might be ready. We expect to have one Pod, but there’s zero running at the moment.

Since the mongo image is relatively big, it might take a while until it is pulled from Docker Hub. After a while, we can retrieve the Pods one more time to confirm that the Pod with the Mongo database is running.
```
Since the mongo image is relatively big, it might take a while until it is pulled from Docker Hub. After a while, we can retrieve the Pods one more time to confirm that the Pod with the Mongo database is running.
```bash
kubectl get pod
```
The output is as follows.
```bash
NAME   READY   STATUS    RESTARTS  AGE
db     1/1     Running   0         6

We can see that, this time, the Pod is ready and we can start using the Mongo database.
```
3. That was not the best way to run Pods so we’ll delete it.
```
kubectl delete pod db
```
The output is as follows.
```
pod "db" deleted
```

# Defining Pods through Declarative Syntax
`
Even though a Pod can contain any number of containers, the most common use case is to use the single-container-in-a-Pod model.
In such a case, a Pod is a wrapper around one container. From Kubernetes’ perspective, a Pod is the smallest unit.
We cannot tell Kubernetes to run a container. Instead, we ask it to create a Pod that wraps around a container
`