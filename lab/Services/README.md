

## How to Start the Communication ?

### Lets discuss about the problem with PODS?

- Pods are the smallest unit in Kubernetes and have a relatively short life-span. They are born, and they are destroyed. They are never healed. The system heals itself by creating new Pods (cells) and by terminating those that are unhealthy. The system is long-living, Pods are not.

- Controllers, together with other components like the scheduler, are making sure that the Pods are doing the right thing.

- ReplicaSet is in charge of making sure that the desired number of Pods is always running. If there’s too few of them, new ones will be created. If there’s too many of them, some will be destroyed. Pods that become unhealthy are terminated as well. All that is controlled by ReplicaSet.

- The problem with our current setup is that there are no communication paths. Our Pods cannot speak with each other. So far, only containers inside a Pod can talk with each other through localhost. That led us to the design where both the API and the database needed to be inside the same Pod. That is not a right solution for quite a few reasons.

- The main problem is that we cannot scale one without the other. We could not design the setup in a way that there are, for example, three replicas of the API and one replica of the database. The primary obstacle was communication.

- Truth is, each Pod does get its own address. We could have split the API and the database into different Pods and configure the API Pods to communicate with the database through the address of the Pod it lives in.

- Since Pods are unreliable, short-lived, and volatile, we cannot assume that the database would always be accessible through the IP of a Pod. When that Pod gets destroyed (or fails), the ReplicaSet would create a new one and assign it a new address.

- We need a stable, never-to-be-changed address that will forward requests to whichever Pod is currently running.

### What is the Solution ?

Kubernetes *`Services`* provide addresses through which Pods can be accessed.

Types of services:

1. ClusterIP
  - ClusterIP – The default value. The service is only accessible from within the Kubernetes cluster – you can’t make requests to your Pods from outside the cluster!

2. NodePort
  - NodePort - This makes the service accessible on a static port on each Node in the cluster. This means that the service can handle requests that originate from outside the cluster.

3. LoadBalancer
   - LoadBalancer -  The service becomes accessible externally through a cloud provider's load balancer functionality. GCP, AWS, Azure, and OpenStack offer this functionality. The cloud provider will create a load balancer, which then automatically routes requests to your Kubernetes Service

### Creating ReplicaSets

Before we see how the services created and works, we will create a ReplicaSet. It’ll provide the Pods we can use to demonstrate how Services work.

1.  Let’s take a quick look at the ReplicaSet definition [go-demo-2-rs.yml](/lab/Services/go-demo-2-rs.yml)
```
cat go-demo-2-rs.yml
```
The only significant difference is the db container definition. It is as follows.
```yaml
...
- name: db
  image: mongo:3.3
  command: ["mongod"]
  args: ["--rest", "--httpinterface"]
  ports:
  - containerPort: 28017
    protocol: TCP
...
```
We customized the command and the arguments so that MongoDB exposes the REST interface. We also defined the containerPort. Those additions are needed so that we can test that the database is accessible through the Service.

2. Let’s create the ReplicaSet.
```
kubectl create -f go-demo-2-rs.yml
```
3. Let’s view the created ReplicaSet.
```
kubectl get -f go-demo-2-rs.yml
```
We created the ReplicaSet and retrieved its state from Kubernetes. The output is as follows.
```
NAME        DESIRED   CURRENT  READY   AGE
go-demo-2   2         2        2       1m
```
4. Verify the number of pods created by this ReplicaSet.
```yaml
kubectl get pods | grep go-demo-2

go-demo-2-88rsw   2/2     Running   0          3m57s
go-demo-2-qj65n   2/2     Running   0          3m57s
```

### Exposing a Resource Using `NodePort Service type`

We can use the kubectl expose command to expose a resource as a new `Kubernetes Service`. That resource can be a Deployment,a ReplicaSet, a ReplicationController, or a Pod. We’ll expose the ReplicaSet since it is already running in the cluster.

1. Run this command to expose the resources `kubectl expose <Resource Type>  <Resource Name> --name <Service Name> --target-port=<port-number> --type=<NodePort>

```yaml
kubectl expose rs go-demo-2 \
    --name=go-demo-2-svc \
    --target-port=28017 \
    --type=NodePort
```
![svc-07.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc07.png)


- Line 1: We specified that we want to expose a ReplicaSet (rs).

- Line 2: The name of the new Service should be go-demo-2-svc.

- Line 3: The port that should be exposed is 28017 (the port MongoDB interface is listening to).

- Line 4: we specified that the type of the Service should be NodePort.

As a result, the target port will be exposed on every node of the cluster to the outside world, and it will be routed to one of the Pods controlled by the ReplicaSet.

2. Get the services
```
kubectl get services
```
3. Output will be as below
```yaml
kubectl get services

NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
go-demo-2-svc   NodePort   10.111.97.246   <none>        28017:30358/TCP   13s
```
4. Describe the service
```
kubectl describe services go-demo-2-svc
```
5. Access the PODS from outsite world
```yaml
http://ExternalIP:<NodePort-Port Number>

Example: http://ExternalIP:<30358>
```

6. Result will be as below

![svc-08](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/sv08.png)

## Other Types of Services 

There are other Service types we could have used to establish communication

### ClusterIP

ClusterIP (the default type) exposes the port only inside the cluster. Such a port would not be accessible from anywhere outside. ClusterIP is useful when we want to enable communication between Pods and still prevent any external access.

> NOTE: If NodePort is used, ClusterIP will be created automatically.

### LoadBalancer

The LoadBalancer type is only useful when combined with cloud provider’s load balancer.

### ExternalName

ExternalName maps a service to an external address (e.g., kubernetes.io).

## Sequential Breakdown of the Service creation

![svc-01.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-01.png)

### The Sequence

- The processes that were initiated with the creation of the Service are as follows:

  - Kubernetes client (kubectl) sent a request to the API server requesting the creation of the Service based on Pods created through the go-demo-2 ReplicaSet.

  - Endpoint controller is watching the API server for new service events. It detected that there is a new Service object.

  - Endpoint controller created endpoint objects with the same name as the Service, and it used Service selector to identify endpoints (in this case the IP and the port of go-demo-2 Pods).

  - kube-proxy is watching for service and endpoint objects. It detected that there is a new Service and a new endpoint object.

  - kube-proxy added iptables rules which capture traffic to the Service port and redirect it to endpoints. For each endpoint object, it adds iptables rule which selects a Pod.

  - he kube-dns add-on is watching for Service. It detected that there is a new service.

  - The kube-dns added db's record to the dns server (skydns).

The sequence we described is useful when we want to understand everything that happened in the cluster from the moment we requested the creation of a new Service. 

However, it might be too confusing so we’ll try to explain the same process through a diagram that more closely represents the cluster.

### The Kubernetes components view when requesting creation of a Service

![svc-02png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-02.png)

1. Let’s take a look at our new Service, describe the new service
```
kubectl describe svc go-demo-2-svc
```
2. The output is as follows

![svc-03.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc03.png)

- Line 1-2: We can see the name and the namespace. We did not yet explore namespaces (coming up later) and, since we didn’t specify any, it is set to default.

- Line 3-6: Since the Service is associated with the Pods created through the ReplicaSet, it inherited all their labels. The selector matches the one from the ReplicaSet. The Service is not directly associated with the ReplicaSet (or any other controller) but with Pods through matching labels.

- Line 9-13: Next is the NodePort type which exposes ports to all the nodes. Since NodePort automatically created ClusterIP type as well, all the Pods in the cluster can access the TargetPort. The Port is set to 28017. That is the port that the Pods can use to access the Service. Since we did not specify it explicitly when we executed the command, its value is the same as the value of the TargetPort, which is the port of the associated Pod that will receive all the requests. NodePort was generated automatically since we did not set it explicitly. It is the port which we can use to access the Service and, therefore, the Pods from outside the cluster. In most cases, it should be randomly generated, that way we avoid any clashes.


![svc-04.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-04.png)


As we know, creating Kubernetes objects using imperative commands is not a good idea unless we’re trying some quick hack.

The same applies to Services. Even though kubectl expose did the work, we should try to use a documented approach through YAML files. So, we’ll destroy the service we created and start creating using YAML.

```
kubectl delete svc go-demo-2-svc
```

## Creating Services through Declarative Syntax

### Looking into the Syntax

1. We can accomplish a similar result as the one using `kubectl expose` through the [go-demo-2-svc.yml](/lab/Services/go-demo-2-svc.yml) specification.

```
cat go-demo-2-svc.yml
```
2. The output is as follows.

![svc-09.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc09.png)


- Line 1-4: You should be familiar with the meaning of apiVersion, kind, and metadata, so we’ll jump straight into the spec section.

- Line 5: Since we already explored some of the options through the kubectl expose command, the spec should be relatively easy to grasp.

- Line 6: The type of the Service is set to NodePort meaning that the ports will be available both within the cluster as well as from outside by sending requests to any of the nodes.

- Line 7-10: The ports section specifies that the requests should be forwarded to the Pods on port 28017. The nodePort is new. Instead of letting the service expose a random port, we set it to the explicit value of 30001. Even though, in most cases, that is not a good practice, I thought it might be a good idea to demonstrate that option as well. The protocol is set to TCP. The only other alternative would be to use UDP. We could have skipped the protocol altogether since TCP is the default value but, sometimes, it is a good idea to leave things as a reminder of an option.

- Line 11-13: The selector is used by the Service to know which Pods should receive requests. It works in the same way as ReplicaSet selectors. In this case, we defined that the service should forward requests to Pods with labels type set to backend and service set to go-demo. Those two labels are set in the Pods spec of the ReplicaSet.

3. Creating the Service
```
kubectl create -f svc/go-demo-2-svc.yml
```
4. Get the service created
```
kubectl get -f svc/go-demo-2-svc.yml
```
5. We created the Service and retrieved its information from the API server. The output of this command is as follows.
```yaml
kubectl get -f svc/go-demo-2-svc.yml

NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
go-demo-2   NodePort   10.104.113.10   <none>        28017:30001/TCP   56s
```
6. Now that the Service is running (again), we can double-check that it is working as expected by trying to access MongoDB UI.

```yaml
http://ExternalIP:<NodePort-Port Number>

Example: http://ExternalIP:<30001>
```
Since we fixed the nodePort to 30001, we did not have to retrieve the Port from the API server. Instead, we used the External IP of the node and the hard-coded port 30001 to open the UI.


![svc-05.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-05.png)


7. Let’s take a look at the endpoint. It holds the list of Pods that should receive requests.
```
kubectl get ep go-demo-2 -o yaml
```
8. The output is as follows.

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    endpoints.kubernetes.io/last-change-trigger-time: "2020-06-01T21:31:20Z"
  creationTimestamp: "2020-06-01T21:31:20Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:endpoints.kubernetes.io/last-change-trigger-time: {}
      f:subsets: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-06-01T21:31:20Z"
  name: go-demo-2
  namespace: k8sdemo
  resourceVersion: "213608"
  selfLink: /api/v1/namespaces/k8sdemo/endpoints/go-demo-2
  uid: d733c825-fbc1-40c5-a472-c808bd138cd5
subsets:
- addresses:
  - ip: 10.244.1.17
    nodeName: worker07-01
    targetRef:
      kind: Pod
      name: go-demo-2-88rsw
      namespace: k8sdemo
      resourceVersion: "198698"
      uid: dfe2e566-b63a-455e-a117-6f0c7b25c1fa
  - ip: 10.244.2.15
    nodeName: worker07-02
    targetRef:
      kind: Pod
      name: go-demo-2-qj65n
      namespace: k8sdemo
      resourceVersion: "198696"
      uid: a84a12bb-9287-4f54-a834-c87487b3aec4
  ports:
  - port: 28017
    protocol: TCP
```
We can see that there are two subsets, corresponding to the two Pods that contain the same labels as the Service selector.

## Request Forwarding

Each Pod has a unique IP that is included in the algorithm used when forwarding requests. Actually, it’s not much of an algorithm. Requests will be sent to those Pods randomly. That randomness results in something similar to round-robin load balancing. If the number of Pods does not change, each will receive an approximately equal number of requests.

## Now We Can Split

So far, we have repeated a few times that our current Pod design is flawed. We have two containers (an API and a database) packaged together. This prevents us from scaling one without the other. Now that we learned how to use Services, we can redesign our Pod solution.

## Destroying Everything

```yaml
kubectl delete -f go-demo-2-svc.yml
kubectl delete -f go-demo-2-rs.yml
```
## Splitting the Pod and Establishing Communication through Services

1. Let’s take a look at a ReplicaSet definition [go-demo-2-db-rs.yml](/lab/Services/go-demo-2-db-rs.yml) for a Pod with only the database.

```yaml
cat go-demo-2-db-rs.yml
```
2. The output is as follows.
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: go-demo-2-db
spec:
  selector:
    matchLabels:
      type: db
      service: go-demo-2
  template:
    metadata:
      labels:
        type: db
        service: go-demo-2
        vendor: MongoLabs
    spec:
      containers:
      - name: db
        image: mongo:3.3
        ports:
        - containerPort: 28017
```

Since this ReplicaSet defines only the database, we reduced the number of replicas to 1. For now, we’ll pretend that one replica of a database is enough.

Since selector labels need to be unique, we changed them slightly. The service is still go-demo-2, but the type was changed to db.

The rest of the definition is the same except that the containers now contain only mongo. We’ll define the API in a separate ReplicaSet.

3. Creating the ReplicaSet, Let’s create the ReplicaSet before we move to the Service that will reference its Pod

```yaml
kubectl create -f go-demo-2-db-rs.yml
```

4. Output will be as follow

```yaml
kubectl get rs

NAME           DESIRED   CURRENT   READY   AGE
go-demo-2-db   1         1         1       2m4s
```
5. Get the PODS detail

```yaml
kubectl get po

NAME                 READY   STATUS    RESTARTS   AGE
go-demo-2-db-j7zpt   1/1     Running   0          3m41s
```
6. Creating the Service for the Pod we just created through the ReplicaSet using this Service defination file [go-demo-2-db-svc.yml](/lab/Services/go-demo-2-db-svc.yml)

```yaml
cat go-demo-2-db-svc.yml
```

7. The output is as follows.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-demo-2-db
spec:
  ports:
  - port: 27017
  selector:
    type: db
    service: go-demo-2
```

- This Service definition does not contain anything new.

  - There is no type, so it’ll default to ClusterIP.

  - Since there is no reason for anyone outside the cluster to communicate with the database, there’s no need to expose it using the NodePort type.

  - We also skipped specifying the NodePort, since only internal communication within the cluster is allowed.

  - The same is true for the protocol. TCP is all we need, and it happens to be the default one.

  - Finally, the selector labels are the same as the labels that define the Pod.

8. Let’s create the Service.

```yaml
kubectl create -f go-demo-2-db-svc.yml
```
We are finished with the database. The ReplicaSet will make sure that the Pod is (almost) always up-and-running and the Service will allow other Pods to communicate with it through a fixed DNS.

## Creating the Split API Pods

1. we will create API Pods using ReplicaSet [go-demo-2-api-rs.yml](/lab/Services/go-demo-2-api-rs.yml) and establish communication by creating Service.

```yaml
cat go-demo-2-api-rs.yml
```
2. The output is as follows.

```yaml
apiVersion:  apps/v1
kind: ReplicaSet
metadata:
  name: go-demo-2-api
spec:
  replicas: 3
  selector:
    matchLabels:
      type: api
      service: go-demo-2
  template:
    metadata:
      labels:
        type: api
        service: go-demo-2
        language: go
    spec:
      containers:
      - name: api
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: go-demo-2-db
        readinessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
          periodSeconds: 1
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
```
- Just as with the database, this ReplicaSet should be familiar since it’s very similar to the one we used before. We’ll talk only on the differences between `db` replicaset defination file and `api` replicaset defination file

  - Line 6: The number of replicas is set to 3. That solves one of the main problems we had with the previous ReplicaSets that defined Pods with both containers. Now the number of replicas can differ, and we have one Pod for the database, and three for the backend API.

  - Line 14: In the labels section, type label is set to api so that both the ReplicaSet and the (soon to come) Service can distinguish the Pods from those created for the database.

  - Line 22-23: We have the environment variable DB set to go-demo-2-db. The code behind the vfarcic/go-demo-2 image is written in a way that the connection to the database is established by reading that variable. In this case, we can say that it will try to connect to the database running on the DNS go-demo-2-db. If you go back to the database Service definition, you’ll notice that its name is go-demo-2-db as well. If everything works correctly, we should expect that the DNS was created with the Service and that it’ll forward requests to the database.

### The readinessProbe

The readinessProbe should be used as an indication that the service is ready to serve requests. When combined with Services construct, only containers with the readinessProbe state set to Success will receive requests.

The readinessProbe has the same fields as the livenessProbe. We used the same values for both, except for the periodSeconds, where instead of relying on the default value of 10, we set it to 1.

While livenessProbe is used to determine whether a Pod is alive or it should be replaced by a new one, the readinessProbe is used by the iptables. A Pod that does not pass the readinessProbe will be excluded and will not receive requests. In theory, requests might be still sent to a faulty Pod, between two iterations. Still, such requests will be small in number since the iptables will change as soon as the next probe responds with HTTP code less than 200, or equal or greater than 400.

3. Let’s create the ReplicaSet.

```
kubectl create -f svc/go-demo-2-api-rs.yml
```

4. The output is as follows.

```yaml
root@master-07:~/kubernetes-lab/lab# kubectl get rs
NAME            DESIRED   CURRENT   READY   AGE
go-demo-2-api   3         3         3       59s

```

5. Check the Service [go-demo-2-api-svc.yml](/lab/Services/go-demo-2-api-svc.yml) which will create now for `api` POD
```
cat go-demo-2-api-svc.yml
```

6. The output is as follow

```yaml
apiVersion: v1
kind: Service
metadata:
  name: go-demo-2-api
spec:
  type: NodePort
  ports:
  - port: 8080
  selector:
    type: api
    service: go-demo-2
```
There’s nothing new in this definition. The type is set to `NodePort` since the API should be accessible from outside the cluster. The selector label type is set to api so that it matches the labels defined for the Pods.

7. Let's create this service 

```yaml
kubectl create -f go-demo-2-api-svc.yml
```
8. Get and Describe the service 

```yaml
kubectl get  svc

kubectl describe svc go-demo-2-api
```
9. The Output as follow
```yaml
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
go-demo-2-api   NodePort    10.104.142.203   <none>        8080:30488/TCP    3m32s
```

```yaml
Name:                     go-demo-2-api
Namespace:                k8sdemo
Labels:                   <none>
Annotations:              <none>
Selector:                 service=go-demo-2,type=api
Type:                     NodePort
IP:                       10.104.142.203
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30488/TCP
Endpoints:                10.244.1.19:8080,10.244.2.16:8080,10.244.2.17:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

```

10. We’ll take a look at what we have in the cluster.

```yaml
kubectl get all
```

11. The Output is as follow.

```yaml
NAME                      READY   STATUS    RESTARTS   AGE
pod/go-demo-2-api-r55fs   1/1     Running   0          3m32s
pod/go-demo-2-api-sng48   1/1     Running   0          3m32s
pod/go-demo-2-api-vvcbp   1/1     Running   0          3m32s
pod/go-demo-2-db-bwvkb    1/1     Running   0          4m20s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/go-demo-2-api   NodePort    10.110.71.67    <none>        8080:31148/TCP   3m23s
service/go-demo-2-db    ClusterIP   10.104.40.176   <none>        27017/TCP        4m1s
service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP          16m

NAME                            DESIRED   CURRENT   READY   AGE
replicaset.apps/go-demo-2-api   3         3         3       3m33s
replicaset.apps/go-demo-2-db    1         1         1       4m20s
```

Both ReplicaSets for `db` and `api` are there, followed by the three replicas of the `go-demo-2-api` Pods and one replica of the `go-demo-2-db Pod`. Finally, the two Services are running as well, together with the one created by Kubernetes itself.

12. Accessing the API from outside world

```yaml
http://$ExternalIP:$PORT/demo/hello

Example: http://$ExternalIP:31148/demo/hello

```

![svc-11.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc11.png)


13. Destroying Services, we’ll delete the objects we created.

```yaml
kubectl delete -f go-demo-2-db-rs.yml
kubectl delete -f go-demo-2-db-svc.yml
kubectl delete -f go-demo-2-api-rs.yml
kubectl delete -f go-demo-2-api-svc.yml
```

## Defining Multiple Objects in the Same YAML file

The vfarcic/go-demo-2 and mongo images form the same stack or application. They work together and having four YAML definitions is confusing. It would get even more confusing later on since we are going to add more objects to the stack. Things would be much simpler and easier if we would move all the objects we created thus far into a single YAML definition. Fortunately, that is very easy to accomplish.

1. Let’s take a look at yet another YAML [go-demo-2.yml](/lab/Services/go-demo-2.yml) file 

```yaml
cat go-demo-2.yml
```

2. The output of `cat` [go-demo-2.yml](/lab/Services/go-demo-2.yml) is the same as the contents of the previous four YAML files combined. The only difference is that each object definition is separated by three dashes (---).

3. Let’s create and view the objects defined in that file.

```yaml
kubectl create -f go-demo-2.yml

kubectl get -f go-demo-2.yml
```

4. The output of the `kubectl get -f go-demo-2.yml` command is as follows.

```yaml

NAME                           DESIRED   CURRENT   READY   AGE
replicaset.apps/go-demo-2-db   1         1         1       2s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
service/go-demo-2-db   ClusterIP   10.100.12.196   <none>        27017/TCP   2s

NAME                            DESIRED   CURRENT   READY   AGE
replicaset.apps/go-demo-2-api   3         3         0       2s

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/go-demo-2-api   NodePort   10.111.76.226   <none>        8080:30746/TCP   2s
```

The two ReplicaSets and the two Services were created, and we can rejoice in replacing four files with one.

5. Finally, to be on the safe side, we’ll also double check that the stack API is up-and-running and accessible from outside world.

```yaml
http://$ExternalIP:$PORT/demo/hello

Example: http://$ExternalIP:31148/demo/hello

```

![svc-11.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc11.png)


## Discovering Services

- Services can be discovered through two principal modes:

 - Environment variables
 - DNS

- Every Pod gets environment variables for each of the active Services

1. Let’s take a look at the environment variables available in one of the Pods we’re running.

```yaml
POD_NAME=$(kubectl get pod \
    --no-headers \
    -o=custom-columns=NAME:.metadata.name \
    -l type=api,service=go-demo-2 \
    | tail -1)



kubectl exec $POD_NAME env
```

2. The The output is as follows.

```yaml

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=go-demo-2-api-rfc7j
DB=go-demo-2-db
GO_DEMO_2_DB_SERVICE_PORT=27017
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
GO_DEMO_2_API_PORT_8080_TCP=tcp://10.111.76.226:8080
GO_DEMO_2_API_PORT_8080_TCP_PORT=8080
KUBERNETES_SERVICE_PORT=443
GO_DEMO_2_DB_PORT_27017_TCP_PROTO=tcp
GO_DEMO_2_DB_SERVICE_HOST=10.100.12.196
GO_DEMO_2_DB_PORT=tcp://10.100.12.196:27017
GO_DEMO_2_DB_PORT_27017_TCP_ADDR=10.100.12.196
GO_DEMO_2_API_PORT=tcp://10.111.76.226:8080
GO_DEMO_2_API_PORT_8080_TCP_ADDR=10.111.76.226
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
GO_DEMO_2_DB_PORT_27017_TCP=tcp://10.100.12.196:27017
GO_DEMO_2_DB_PORT_27017_TCP_PORT=27017
GO_DEMO_2_API_SERVICE_HOST=10.111.76.226
GO_DEMO_2_API_SERVICE_PORT=8080
GO_DEMO_2_API_PORT_8080_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT_HTTPS=443
HOME=/root

``` 

The environment variables are Kubernetes specific and follow the [SERVICE_NAME]_SERVICE_HOST and [SERVICE_NAME]_SERIVCE_PORT format (service name is upper-cased).

No matter which set of environment variables you choose to use (if any), they all serve the same purpose. They provide a reference we can use to connect to a Service and, therefore to the related Pods.

3. Things will become more evident when we describe the go-demo-2-db Service.

```
kubectl describe svc go-demo-2-db
```

4. The output is as follows.

```yaml

Name:              go-demo-2-db
Namespace:         k8sdemo
Labels:            <none>
Annotations:       <none>
Selector:          service=go-demo-2,type=db
Type:              ClusterIP
IP:                10.100.12.196
Port:              <unset>  27017/TCP
TargetPort:        27017/TCP
Endpoints:         10.244.2.18:27017
Session Affinity:  None
Events:            <none>

```
The key is in the IP field. That is the IP through which this service can be accessed and it matches the values of the environment variables GO_DEMO_2_DB_* and GO_DEMO_2_DB_SERVICE_HOST.

The code inside the containers that form the go-demo-2-api Pods could use any of those environment variables to construct a connection string towards the go-demo-2-db Pods. For example, we could have used GO_DEMO_2_DB_SERVICE_HOST to connect to the database. And, yet, we didn’t do that. The reason is simple. It is easier to use DNS instead.

5. Let’s take another look at the snippet from the go-demo-2-api-rs.yml ReplicaSet definition.

```yaml
cat go-demo-2-api-rs.yml
```

6. The output limited to the environment variable is as follows.

```yaml
...
env:
- name: DB
  value: go-demo-2-db
..
```

We declared an environment variable with the name of the Service (go-demo-2-db). That variable is used by the code as a connection string to the database.

Kubernetes converts Service names into DNSes and adds them to the DNS server.

## Sequential Breakdown of the Process

- Let’s go through the sequence of events related to service discovery and components involved.

  - When the api container go-demo-2 tries to connect with the go-demo-2-db Service, it looks at the nameserver configured in /etc/resolv.conf. kubelet configured the nameserver with the kube-dns Service IP (10.96.0.10) during the Pod scheduling process.

  - The container queries the DNS server listening to port 53. go-demo-2-db DNS gets resolved to the service IP 10.0.0.19. This DNS record was added by kube-dns during the service creation process.

  - The container uses the service IP which forwards requests through the iptables rules. They were added by kube-proxy during Service and Endpoint creation process.

  - Since we only have one replica of the go-demo-2-db Pod, iptables forwards requests to just one endpoint. If we had multiple replicas, iptables would act as a load balancer and forward requests randomly among Endpoints of the Service.
    
![svc-06.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-06.png)


Quick Quiz!

1. A Pod replacing the killed or failed Pod will have the same IP.

A)
True
B)
False


2. The port exposed by the ClusterIP type service is

A)
Accessible from anywhere outside of the Cluster
B)
Accessible only from the inside of the Cluster
C)
Accessible from both inside and outside of the Cluster


3. When NodePort is used for creating a service, ClusterIP will be created automatically.

A)
True
B)
False


4. Which of the following command can be used to expose a resource as a new Kubernetes Service.


A)
kubectl resource
B)
kubectl expose
C)
kubectl create


5. If the number of Pods doesn’t change, the number of requests forwarded to each Pod will be?

A)
Equal
B)
Unequal