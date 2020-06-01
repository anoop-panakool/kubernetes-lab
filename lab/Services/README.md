

## Sequential Breakdown of the Process

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