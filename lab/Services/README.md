

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
```
kubectl get pods | grep go-demo-2

go-demo-2-88rsw   2/2     Running   0          3m57s
go-demo-2-qj65n   2/2     Running   0          3m57s
```

### Exposing a Resource Using `NodePort Service type`

We can use the kubectl expose command to expose a resource as a new `Kubernetes Service`. That resource can be a Deployment,a ReplicaSet, a ReplicationController, or a Pod. We’ll expose the ReplicaSet since it is already running in the cluster.

1. Run this command to expose the resources `kubectl expose <Resource Type>  <Resource Name> --name <Service Name> --target-port=<port-number> --type=<NodePort>

```bash
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
```
kubectl get services

NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
go-demo-2-svc   NodePort   10.111.97.246   <none>        28017:30358/TCP   13s
```
4. Describe the service
```
kubectl describe services go-demo-2-svc
```
5. Access the PODS from outsite world
```
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
![svc-05.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-05.png)




![svc-06.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-06.png)