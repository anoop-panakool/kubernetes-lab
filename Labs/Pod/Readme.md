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
Let’s take a look at a simple Pod definition by accessing the db.yml
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
 > click [here](/Labs/Pod/Lab06-pod/db.yml)
Let’s analyze the various sections in the output definition of a Pod.
```
Line 1-2: We’re using v1 of Kubernetes Pods API. Both apiVersion and kind are mandatory. That way, Kubernetes knows what we want to do (create a Pod) and which API version to use.

Line 3-7: The next section is metadata. It provides information that does not influence how the Pod behaves. We used metadata to define the name of the Pod (db) and a few labels. Later on, when we move into Controllers, labels will have a practical purpose. For now, they are purely informational.

Line 8: The last section is the spec in which we defined a single container. As you might have guessed, we can have multiple containers defined as a Pod. Otherwise, the section would be written in singular (container without s). We’ll explore multi-container Pods later.

Line 12: In our case, the container is defined with the name (db), the image (mongo), the command that should be executed when the container starts (mongod)

Line 13: Finally, the set of arguments. The arguments are defined as an array with, in this case, two elements (--rest and --httpinterface).
```
1. Let’s create the `Pod` defined in the `db.yml` file.
```
kubectl create -f db.yml
```
`
The command will create the kind of resource defined in the pod/db.yml
`
Let’s take a look at the Pods in the cluster.
`
kubectl get pod
`
The output is as follows.
```
NAME READY STATUS  RESTARTS AGE
db   1/1   Running 0        11s
```
Pod named db is up and running!

2. Retrieve more information about the `POD` by specifying wide output
```
kubectl get pods -o wide
```
The output is as follows.
```
NAME READY STATUS  RESTARTS AGE IP              NODE                     NOMINATED NODE  READINESS GATES
db   1/1   Running 0        1m  192.168.221.134 shivam2c.labserver.com   <none>          <none>
```
we got two additional columns; the IP and the NODE.

3. If you’d like to parse the output, using `json` format.
```
kubectl get pods -o json
```
```json
root@shivam1c:~/k8s-specs# kubectl get pods db -o json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "creationTimestamp": "2020-05-26T11:23:09Z",
        "labels": {
            "run": "db"
        },
        "managedFields": [
            {
                "apiVersion": "v1",
                "fieldsType": "FieldsV1",
                "fieldsV1": {
                    "f:metadata": {
                        "f:labels": {
                            ".": {},
                            "f:run": {}
                        }
                    },
                    "f:spec": {
                        "f:containers": {
                            "k:{\"name\":\"db\"}": {
                                ".": {},
                                "f:image": {},
                                "f:imagePullPolicy": {},
                                "f:name": {},
                                "f:resources": {},
                                "f:terminationMessagePath": {},
                                "f:terminationMessagePolicy": {}
                            }
                        },
                        "f:dnsPolicy": {},
                        "f:enableServiceLinks": {},
                        "f:restartPolicy": {},
                        "f:schedulerName": {},
                        "f:securityContext": {},
                        "f:terminationGracePeriodSeconds": {}
                    }
                },
                "manager": "kubectl",
                "operation": "Update",
                "time": "2020-05-26T11:23:09Z"
            },
            {
                "apiVersion": "v1",
                "fieldsType": "FieldsV1",
                "fieldsV1": {
                    "f:status": {
                        "f:conditions": {
                            "k:{\"type\":\"ContainersReady\"}": {
                                ".": {},
                                "f:lastProbeTime": {},
                                "f:lastTransitionTime": {},
                                "f:status": {},
                                "f:type": {}
                            },
                            "k:{\"type\":\"Initialized\"}": {
                                ".": {},
                                "f:lastProbeTime": {},
                                "f:lastTransitionTime": {},
                                "f:status": {},
                                "f:type": {}
                            },
                            "k:{\"type\":\"Ready\"}": {
                                ".": {},
                                "f:lastProbeTime": {},
                                "f:lastTransitionTime": {},
                                "f:status": {},
                                "f:type": {}
                            }
                        },
                        "f:containerStatuses": {},
                        "f:hostIP": {},
                        "f:phase": {},
                        "f:podIP": {},
                        "f:podIPs": {
                            ".": {},
                            "k:{\"ip\":\"192.168.221.134\"}": {`s
                                ".": {},
                                "f:ip": {}
                            }
                        },
                        "f:startTime": {}
                    }
                },
                "manager": "kubelet",
                "operation": "Update",
                "time": "2020-05-26T11:23:21Z"
            }
        ],
        "name": "db",
        "namespace": "default",
        "resourceVersion": "8657",
        "selfLink": "/api/v1/namespaces/default/pods/db",
        "uid": "a257f112-c2dc-43ad-bdb7-46c6d95fabd9"
    },
    "spec": {
        "containers": [
            {
                "image": "mongo",
                "imagePullPolicy": "Always",
                "name": "db",
                "resources": {},
                "terminationMessagePath": "/dev/termination-log",
                "terminationMessagePolicy": "File",
                "volumeMounts": [
                    {
                        "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                        "name": "default-token-mvzfj",
                        "readOnly": true
                    }
                ]
            }
        ],
        ---------
        ---------
        ---------
                    }
                }
            }
        ],
        "hostIP": "172.31.102.185",
        "phase": "Running",
        "podIP": "192.168.221.134",
        "podIPs": [
            {
                "ip": "192.168.221.134"
            }
        ],
        "qosClass": "BestEffort",
        "startTime": "2020-05-26T11:23:09Z"
    }
}
```
`The output is too big and not that important in currentl lab. One of the last line is as follows`

4. If you’d like to parse the output, using `YAML` format.
```
kubectl get pods -o yaml
```
```yaml
root@shivam1c:~/k8s-specs# kubectl get pods db -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-05-26T11:23:09Z"
  labels:
    run: db
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:run: {}
      f:spec:
        f:containers:
          k:{"name":"db"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:resources: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
    manager: kubectl
    operation: Update
    time: "2020-05-26T11:23:09Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"192.168.221.134"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2020-05-26T11:23:21Z"
  name: db
  namespace: default
  resourceVersion: "8657"
  selfLink: /api/v1/namespaces/default/pods/db
  uid: a257f112-c2dc-43ad-bdb7-46c6d95fabd9
spec:
  containers:
  - image: mongo
    imagePullPolicy: Always
    name: db
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-mvzfj
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: shivam2c.mylabserver.com
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-mvzfj
    secret:
      defaultMode: 420
      secretName: default-token-mvzfj
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-05-26T11:23:09Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2020-05-26T11:23:21Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2020-05-26T11:23:21Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2020-05-26T11:23:09Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://4f5b9dea758efdb126bac288ffd0fa4ae6038c69f7d6c4940394666c96ce5dcb
    image: mongo:latest
    imageID: docker-pullable://mongo@sha256:c880f6b56f443bb4d01baa759883228cd84fa8d78fa1a36001d1c0a0712b5a07
    lastState: {}
    name: db
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2020-05-26T11:23:21Z"
  hostIP: 172.31.102.185
  phase: Running
  podIP: 192.168.221.134
  podIPs:
  - ip: 192.168.221.134
  qosClass: BestEffort
  startTime: "2020-05-26T11:23:09Z"
  ```

5. If you’d like to see details of the specified resource, use the describe sub-command. 
`In this case, the resource is the Pod named db.`
```yaml
root@shivam1c:~/k8s-specs# kubectl describe pods db
Name:         db
Namespace:    default
Priority:     0
Node:         shivam2c.mylabserver.com/172.31.102.185
Start Time:   Tue, 26 May 2020 11:23:09 +0000
Labels:       run=db
Annotations:  <none>
Status:       Running
IP:           192.168.221.134
IPs:
  IP:  192.168.221.134
Containers:
  db:
    Container ID:   docker://4f5b9dea758efdb126bac288ffd0fa4ae6038c69f7d6c4940394666c96ce5dcb
    Image:          mongo
    Image ID:       docker-pullable://mongo@sha256:c880f6b56f443bb4d01baa759883228cd84fa8d78fa1a36001d1c0a0712b5a07
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 26 May 2020 11:23:21 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-mvzfj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-mvzfj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-mvzfj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
```
# Components Involved in a Pod’s Scheduling

## Three major components were involved in the process:

#### 1. API Server 
The API server is the central component of a Kubernetes cluster and it runs on the master node.

All other components interact with API server and keep watch for changes. Most of the coordination in Kubernetes consists of a component writing to the API Server resource that another component is watching. The second component will then react to changes almost immediately.

#### 2. Scheduler
The scheduler is also running on the master node. Its job is to watch for unassigned pods and assign them to a node which has available resources (CPU and memory) matching Pod requirements.

#### 3. Kubelet
Kubelet runs on each node. Its primary function is to make sure that assigned pods are running on the node. It watches for any new Pod assignments for the node. If a Pod is assigned to the node Kubelet is running on, it will pull the Pod definition and use it to create containers through Docker or any other supported container engine.

## Sequential Breakdown of Events

![Pod scheduling sequence](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/Pod-scheduling-sequence.png)
 

The sequence of events take place when we run `kubectl create -f db.yml` command is as below:

##### 1. Kubernetes client (kubectl) sent a request to the API server requesting creation of a Pod defined in the `db.yml` file.

##### 2. Since the scheduler is watching the API server for new events, it detected that there is an unassigned Pod.

##### 3. The scheduler decided which node to assign the Pod to and sent that information to the API server.

##### 4. Kubelet is also watching the API server. It detected that the Pod was assigned to the node it is running on.

##### 5. Kubelet sent a request to Docker requesting the creation of the containers that form the Pod. In our case, the Pod defines a single container based on the mongo image.

##### 6. Finally, Kubelet sent a request to the API server notifying it that the Pod was created successfully.

# Deep Dive in Running Pod & Running Single Container in a Single Pod


![Image of Single Container POD](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/single-container-pod.png)



### Describing the Resources
```
kubectl describe -f db.ym
```
### Executing a New Process
```
kubectl exec db ps aux
```
The output will be similar as follows.
```
USER PID %CPU %MEM    VSZ   RSS TTY STAT START TIME COMMAND
root   1  0.5  2.9 967452 59692 ?   Ssl  21:47 0:03 mongod --rest --httpinterface
root  31  0.0  0.0  17504  1980 ?   Rs   21:58 0:00 ps aux
```
### Executing the Process in detached mode, make the execution interactive with `-i` `(stdin)` and `-t` `(terminal)` arguments and run `shell` inside a container
```
kubectl exec -it db sh
```
###  Execute db.stats() to confirm that the database is running
```json
# echo 'db.stats()' | mongo localhost:27017/test
MongoDB shell version v4.2.6
connecting to: mongodb://localhost:27017/test?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("6976a5a3-1b54-451b-99c2-02c00cae8894") }
MongoDB server version: 4.2.6
{
        "db" : "test",
        "collections" : 0,
        "views" : 0,
        "objects" : 0,
        "avgObjSize" : 0,
        "dataSize" : 0,
        "storageSize" : 0,
        "numExtents" : 0,
        "indexes" : 0,
        "indexSize" : 0,
        "scaleFactor" : 1,
        "fileSize" : 0,
        "fsUsedSize" : 0,
        "fsTotalSize" : 0,
        "ok" : 1
}
bye
```
`We used mongo client to execute db.stats() for the database test running on localhost:27017`````s
### Exit out of the container
```
# exit
```

# Getting Logs
Usually, Logs should be shipped from containers to a central location but here we will exlore the logs of a container in a Pod.

### The command that outputs logs of the only container in the db Pod is as follows:
```
kubectl logs db
```
The output is too big and not that important currentl lab. One of the last line is as follows
```
2020-05-26T15:31:33.921+0000 I  SHARDING [LogicalSessionCacheReap] Marking collection config.transactions as collection version: <unsharded>
2020-05-26T15:37:34.187+0000 I  NETWORK  [listener] connection accepted from 127.0.0.1:54052 #1 (1 connection now open)
2020-05-26T15:37:34.337+0000 I  NETWORK  [conn1] received client metadata from 127.0.0.1:54052 conn1: { application: { name: "MongoDB Shell" }, driver: { name: "MongoDB Internal Client", version: "4.2.6" }, os: { type: "Linux", name: "Ubuntu", architecture: "x86_64", version: "18.04" } }
2020-05-26T15:37:34.363+0000 I  NETWORK  [conn1] end connection 127.0.0.1:54052 (0 connections now open)
```
# Troubleshoot the Failure of POD

### What happens when a container inside a Pod dies?

![Image of POD Killed](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/Pod-with-a-failed-container.png)
#### Let's Kill the container

```
kubectl exec -it db pkill mongod
kubectl get pods -w | grep db
```
> The **output** is as follows.
```
# kubectl get pods -w | grep db
db                   0/1     Completed          4          7h34m
db                   0/1     CrashLoopBackOff   4          7h34m
db                   1/1     Running            5          7h35m
```
### Deleting the Pod
we can delete a Pod if we don’t need it anymore.
```
# kubectl delete -f db.yml
pod "db" deleted

# kubectl get pods
```

# Running Multiple Containers in a Single Pod

![Image of multi-POD-container](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/Multi-Container-POD.png)

### Anatomy of a Pod

- Pod is a group of containers.

- It is the smallest unit that can be scheduled to be deployed through K8s.

- All the containers that form a Pod are running on the same machine. A Pod cannot be split across multiple nodes.

- All the processes `(containers)` inside a Pod share the same set of resources, and they can communicate with each other through `localhost`. One of those shared resources is storage.

- A volume (think of it as a directory with shareable data) defined in a Pod can be accessed by all the containers thus allowing them all to share the same data.

### Example- `go-demo-2.yml` specification
```
cat go-demo-2.yml
```
> The **output** is as follows.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: go-demo-2
  labels:
    type: stack
spec:
  containers:
  - name: db
    image: mongo:3.3
  - name: api
    image: vfarcic/go-demo-2
    env:
    - name: DB
      value: localhost
```
 > click [here](/Labs/Pod/Lab06-pod/go-demo-2.yml) to see the `go-demo-2.yml` file
 
- The YAML file defines a Pod with two containers named 
   -  db
   -  api
- The service inside the vfarcic/go-demo-2 image uses environment variable DB to know where the database is.
- The value is localhost since all the containers in the same Pod are reachable through it

> Create a new Pod defined in the go-demo-2.yml file and retrieve its information from Kubernetes Cluster. 
```
kubectl create -f go-demo-2.yml

kubectl get -f go-demo-2.yml
```
> The output of the command is as follows.
```
root@master:~/# kubectl create -f go-demo-2.yml
pod/go-demo-2 created

root@master:~/# kubectl get -f go-demo-2.yml
NAME        READY   STATUS    RESTARTS   AGE
go-demo-2   2/2     Running   0          17s
```
> We can see from the READY column that, this time, the Pod has two containers (2/2).

## Explore the Output

#### We want to retrieve the names of the containers in a Pod.

> The first thing have to do is get familiar with Kubernetes API. We can do that by going to [Pod v1 core](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.10/#Pod-v1-core) documentation. 

#### Inspect the Output of do-demo-e.yml in either YAML or json format

> The output is too big to be presented here, so captured only important part.

> We need to retrieve the names of the containers in the Pod. Therefore, the part of the output we’re looking for is in attached picture.

![Image of Output of multi-POD ](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/go-demo-2-container-output.png)

```
kubectl get -f pod/go-demo-2.yml \
    -o jsonpath="{.spec.containers[*].name}"
```
> The **output** is as follow
```
db api
```
## Executing Commands Inside the Pod

#### **How would we execute a command inside the Pod?**

> Display the processes inside the db container. Namely, the mongod process
```
kubectl exec -it -c db go-demo-2 ps aux
```
```
root@master:~/# kubectl exec -it -c db go-demo-2 ps aux

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mongodb      1  0.4  2.9 270696 59172 ?        Ssl  20:40   0:17 mongod
root        31  0.0  0.1  17496  2036 pts/0    Rs+  21:50   0:00 ps aux
```
```
```
kubectl exec -it -c api go-demo-2 ps aux
```
root@master:~/# kubectl exec -it -c api go-demo-2 ps aux

PID   USER     TIME   COMMAND
    1 root       0:00 go-demo
   13 root       0:00 ps aux
```
#### **How to see logs from a container?**
> **Can we excute the command** like this *kubectl logs go-demo-2* to check logs inside both the **Containers** 

- **NO** Since the Pod hosts multiple containers
- We need to be specific and name the container from which we want to see the logs.

#### Example as below
```
kubectl logs go-demo-2 -c db
```
> The **output** is as follow
```diff
root@master:~#  kubectl logs go-demo-2 -c db
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] MongoDB starting : pid=1 port=27017 dbpath=/data/db 64-bit host=go-demo-2
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] db version v3.3.15
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] git version: 520f5571d039b57cf9c319b49654909828971073
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.0.1t  3 May 2016
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] allocator: tcmalloc
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] modules: none
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] build environment:
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten]     distmod: debian81
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten]     distarch: x86_64
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten]     target_arch: x86_64
2020-05-26T20:40:21.588+0000 I CONTROL  [initandlisten] options: {}
2020-05-26T20:40:21.592+0000 I STORAGE  [initandlisten] 
2020-05-26T20:40:21.592+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
```
#### **How to scale the service so that there are two containers of the API and one container for the database??**

#### TRY NOW & See the result 
```
cat go-demo-2-scaled.yml
```
> The **output** is as follow

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: go-demo-2
  labels:
    type: stack
spec:
  containers:
  - name: db
    image: mongo:3.3
  - name: api-1
    image: vfarcic/go-demo-2
    env:
    - name: DB
      value: localhost
  - name: api-2
    image: vfarcic/go-demo-2
    env:
    - name: DB
      value: localhos
```
> We defined two containers for the API and named them api-1 and api-2.

# Single vs. Multi-Container Pods

#### **Why we need Single-container Pods**

- A Pod is a collection of containers that share the same resources.

- When we scale Pods with multi-container it will also scale all the container inside it .

- For example, we scale the Pod to three, we have three APIs and three DBs. Instead, we should have defined two Pods, one for each container (db and api).

> 📝 A Pod is a collection of containers. However, that does not mean that multi-container Pods are common. They are rare. Most Pods you’ll create will be single container units.

#### ***Does that mean that multi-container Pods are useless?***

- **NO,They’re not.**

- There are scenarios when having multiple containers in a Pod is a good idea.

- There are cases, where one container acts as the main service and the rest serving as side-cars(helper)

- Few use case is multi-container Pods used for:
  - Continuous integration (CI)
  - Continious Delivery (CD)
  - Continuous Deployment processes (CDP)

# Monitor the Health of the PODS

#### ***Why to Monitor Health?***

##### Example, a back-end API can be up and running but, due to a memory leak, serves requests much slower than expected. Such a situation might benefit from a health check that would verify whether the service responds within, for example, two seconds.

##### Many containers, especially web services, won't have an exit code that accurately tells Kubernetes whether the container was successful. Most web servers won't terminate at all! How do you tell Kubernetes about your container's health? 

##### You define container probes.

#### Container probes are small processes that run periodically. The result of this process determines Kubernetes' view of the container's state—the result of the probe is one of Success, Failed, or Unknown.

- We use container probes through Liveness and Readiness probes.
  - Liveness probes 
    - Liveness probes are responsible for determining if a container is running or when it needs to be restarted.
    - livenessProbe can be used to confirm whether a container should be running. If the probe fails, Kubernetes will kill the container and apply restart policy which defaults to Always.

  - Readiness probes
    - Readiness probes indicate that a container is ready to accept traffic.
    - Once all of its containers indicate they are ready to accept traffic, the pod containing them can accept requests.

- There are three ways to implement these probes.
  - One way is to use HTTP requests, which look for a successful status code in response to making a request to a defined endpoint.
  - Another method is to use TCP sockets, which returns a failed status if the TCP connection cannot be established.
  - The final, most flexible, way is to define a custom command, whose exit code determines whether the check is successful.

### Example- **Liveness probes** go-demo-2-health.yml specification
```
cat go-demo-2.yml
```
> The **output** is as follows.

![Liveness-Probe](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/livenessProbe01.png)

> click [here](/Labs/Pod/Lab06-pod/go-demo-2-health.yml) to see the `go-demo-2-health.yml` file

- Line 8-12: 
  - Two container are specified .

- Line 16-19: 
  - Definition is of the livenessProbe. We defined that the action should be httpGet followed with the path and the port of the service. 
  - Since /this/path/does/not/exist is true to itself, the probe will fail, thus showing us what happens when a container is unhealthy. The host is not specified since it defaults to the Pod IP.

- Line 20-23:
  - We declared that the first execution of the probe should be delayed by five seconds (initialDelaySeconds)
  - That requests should timeout after two seconds (timeoutSeconds),
  - That the process should be repeated every five seconds (periodSeconds), and
  - failureThreshold define how many attempts it must try before giving up .

##### Run this liveness Probe example
  ```
  kubectl create \
    -f pod/go-demo-2-health.yml
  ```
  ```
  kubectl describe \
    -f pod/go-demo-2-health.yml
  ```

  ##### Run this liveness & readiness Probe example
  ```
  kubectl apply -f liveness-readiness.yaml
  ```
  ![liveness-rediness-probe](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/liveness-rediness-probe.png)

  ```
  kubectl describe pod liveness-readiness-pod
  ```
  ![liveness-rediness-probe-result](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/liveness-rediness-probe-result.png)