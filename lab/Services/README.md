

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

### Creating ReplicaSets

Before we see how the services created and works, we will create a ReplicaSet. It’ll provide the Pods we can use to demonstrate how Services work.

> Let’s take a quick look at the ReplicaSet definition [go-demo-2-rs.yml](/lab/Services/go-demo-2-rs.yml)
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

![svc-01.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-01.png)

## The Kubernetes components view when requesting creation of a Service
![svc-02png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-02.png)

## Let’s take a look at our new Service.

- Describe the new service

```bash
kubectl describe svc go-demo-2-svc
```
**The output is as follows.**


![svc-03.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc03.png)

1. Line 1-2: We can see the name and the namespace. We did not yet explore namespaces (coming up later) and, since we didn’t specify any, it is set to default.

2. Line 3-6: Since the Service is associated with the Pods created through the ReplicaSet, it inherited all their labels. The selector matches the one from the ReplicaSet. The Service is not directly associated with the ReplicaSet (or any other controller) but with Pods through matching labels.

3. Line 9-13: Next is the NodePort type which exposes ports to all the nodes. Since NodePort automatically created ClusterIP type as well, all the Pods in the cluster can access the TargetPort. The Port is set to 28017. That is the port that the Pods can use to access the Service. Since we did not specify it explicitly when we executed the command, its value is the same as the value of the TargetPort, which is the port of the associated Pod that will receive all the requests. NodePort was generated automatically since we did not set it explicitly. It is the port which we can use to access the Service and, therefore, the Pods from outside the cluster. In most cases, it should be randomly generated, that way we avoid any clashes.



![svc-04.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-04.png)




![svc-05.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-05.png)




![svc-06.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc-06.png)