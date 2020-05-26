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
The output is as follows.
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

![Image of Multi-Container-Pod]()

### Anatomy of a Pod

- Pod is a group of containers.

- It is the smallest unit that can be scheduled to be deployed through K8s.

- All the containers that form a Pod are running on the same machine. A Pod cannot be split across multiple nodes.

- All the processes `(containers)` inside a Pod share the same set of resources, and they can communicate with each other through `localhost`. One of those shared resources is storage.

- A volume (think of it as a directory with shareable data) defined in a Pod can be accessed by all the containers thus allowing them all to share the same data.

### Example- go-demo-2.yml specification
```
cat go-demo-2.yml
```
The output is as follows.
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
   *db* and *api*
- The service inside the vfarcic/go-demo-2 image uses environment variable DB to know where the database is.
- The value is localhost since all the containers in the same Pod are reachable through it