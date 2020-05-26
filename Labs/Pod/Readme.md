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

Line 1-2: We’re using v1 of Kubernetes Pods API. Both apiVersion and kind are mandatory. That way, Kubernetes knows what we want to do (create a Pod) and which API version to use.

Line 3-7: The next section is metadata. It provides information that does not influence how the Pod behaves. We used metadata to define the name of the Pod (db) and a few labels. Later on, when we move into Controllers, labels will have a practical purpose. For now, they are purely informational.

Line 8: The last section is the spec in which we defined a single container. As you might have guessed, we can have multiple containers defined as a Pod. Otherwise, the section would be written in singular (container without s). We’ll explore multi-container Pods later.

Line 12: In our case, the container is defined with the name (db), the image (mongo), the command that should be executed when the container starts (mongod)

Line 13: Finally, the set of arguments. The arguments are defined as an array with, in this case, two elements (--rest and --httpinterface).

1. Let’s create the Pod defined in the db.yml file.
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

2. Retrieve more information about the POD by specifying wide output
```
kubectl get pods -o wide
```
The output is as follows.
```
NAME READY STATUS  RESTARTS AGE IP              NODE                     NOMINATED NODE  READINESS GATES
db   1/1   Running 0        1m  192.168.221.134 shivam2c.labserver.com   <none>          <none>
```
we got two additional columns; the IP and the NODE.

3. If you’d like to parse the output, using json format.
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
                            "k:{\"ip\":\"192.168.221.134\"}": {
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
        "dnsPolicy": "ClusterFirst",
        "enableServiceLinks": true,
        "nodeName": "shivam2c.mylabserver.com",
        "priority": 0,
        "restartPolicy": "Always",
        "schedulerName": "default-scheduler",
        "securityContext": {},
        "serviceAccount": "default",
        "serviceAccountName": "default",
        "terminationGracePeriodSeconds": 30,
        "tolerations": [
            {
                "effect": "NoExecute",
                "key": "node.kubernetes.io/not-ready",
                "operator": "Exists",
                "tolerationSeconds": 300
            },
            {
                "effect": "NoExecute",
                "key": "node.kubernetes.io/unreachable",
                "operator": "Exists",
                "tolerationSeconds": 300
            }
        ],
        "volumes": [
            {
                "name": "default-token-mvzfj",
                "secret": {
                    "defaultMode": 420,
                    "secretName": "default-token-mvzfj"
                }
            }
        ]
    },
    "status": {
        "conditions": [
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-05-26T11:23:09Z",
                "status": "True",
                "type": "Initialized"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-05-26T11:23:21Z",
                "status": "True",
                "type": "Ready"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-05-26T11:23:21Z",
                "status": "True",
                "type": "ContainersReady"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-05-26T11:23:09Z",
                "status": "True",
                "type": "PodScheduled"
            }
        ],
        "containerStatuses": [
            {
                "containerID": "docker://4f5b9dea758efdb126bac288ffd0fa4ae6038c69f7d6c4940394666c96ce5dcb",
                "image": "mongo:latest",
                "imageID": "docker-pullable://mongo@sha256:c880f6b56f443bb4d01baa759883228cd84fa8d78fa1a36001d1c0a0712b5a07",
                "lastState": {},
                "name": "db",
                "ready": true,
                "restartCount": 0,
                "started": true,
                "state": {
                    "running": {
                        "startedAt": "2020-05-26T11:23:21Z"
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

4. If you’d like to parse the output, using YAML format.
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

### 1. API Server 

### 2. Scheduler

### 3. Kubelet 
