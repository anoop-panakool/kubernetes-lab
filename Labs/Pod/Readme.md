# A Lazy and Dirty Way to Run Pods

In this lab we will create a Pod with Mongo on Kubernetes cluster.

## Prerequisites

- Cluster is in Ready state

## Instructions

1. Creating a Pod with Mongo
`
kubectl allows us to create Pods with a single command. For example, if we would like to create a Pod with a Mongo database, the command is as follows.
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

Looking into a Pod’s Definition #
`
Let’s take a look at a simple Pod definition by accessing the db.yml click [here](kubernetes-lab/Labs/Lab06-pod/db.yml)
`
```
cat db.yml
````
The Ouptput is as follows.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    type: db
    vendor: MongoLabs
spec:
  containers:
  - name: db
    image: mongo:3.3
    command: ["mongod"]
    args: ["--rest", "--httpinterface"]
```

Let’s analyze the various sections in the output definition of a Pod.

Line 1-2: We’re using v1 of Kubernetes Pods API. Both apiVersion and kind are mandatory. That way, Kubernetes knows what we want to do (create a Pod) and which API version to use.

Line 3-7: The next section is metadata. It provides information that does not influence how the Pod behaves. We used metadata to define the name of the Pod (db) and a few labels. Later on, when we move into Controllers, labels will have a practical purpose. For now, they are purely informational.

Line 8: The last section is the spec in which we defined a single container. As you might have guessed, we can have multiple containers defined as a Pod. Otherwise, the section would be written in singular (container without s). We’ll explore multi-container Pods later.

Line 12: In our case, the container is defined with the name (db), the image (mongo), the command that should be executed when the container starts (mongod)

Line 13: Finally, the set of arguments. The arguments are defined as an array with, in this case, two elements (--rest and --httpinterface).
