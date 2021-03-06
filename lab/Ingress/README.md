## Why Services Are Not the Best Fit for External Access?

It is a bad practice to publish fixed ports through Services. That method is likely to result in conflicts or, at the very least, create the additional burden of carefully keeping track of which port belongs to which Service. 

1. let’s create the Deployments and the Services using this YAML file [go-demo-2-deploy.yml](/lab/Ingress/go-demo-2-deploy.yml)

```yaml
kubectl create -f go-demo-2-deploy.yml

kubectl get -f go-demo-2-deploy.yml
```

2. The output of the get command is as follows.
```yaml
kubectl get -f go-demo-2-deploy.yml

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-demo-2-db   1/1     1            1           58s

NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
service/go-demo-2-db   ClusterIP   10.98.76.36   <none>        27017/TCP   58s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-demo-2-api   3/3     3            3           58s

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/go-demo-2-api   NodePort   10.109.198.99   <none>        8080:32013/TCP   58s
```

3. List the PODS
```yaml
kubectl get pods | grep go-demo-2
````

4. The output is as follows.
```yaml
go-demo-2-api-7b67f95d58-7d78j   1/1     Running   0          3m39s
go-demo-2-api-7b67f95d58-nmklg   1/1     Running   0          3m39s
go-demo-2-api-7b67f95d58-pt7lq   1/1     Running   0          3m39s
go-demo-2-db-7c77d7bc64-9j6cx    1/1     Running   0          3m39s
```

4. Access Through Services

```yaml
http://ExternalIP:<NodePort-Port Number>

Example: http://34.89.204.43:32013/demo/hello
```
5. You will get page as below

![svc-11.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc11.png)

The application responded with the status code 200 thus confirming that the Service indeed forwards the requests.

While publishing a random, or even a hard-coded port of a single application might not be so bad, if we’d apply the same principle to more applications, the user experience would be horrible. To make the point a bit clearer, we’ll deploy another application.

6. Let's deploy another application, Create [devops-toolkit-dep.yml](/lab/Ingress/devops-toolkit-dep.yml)

```yaml
kubectl create -f devops-toolkit-dep.yml --record --save-config

kubectl get -f devops-toolkit-dep.yml

kubectl get pods | grep devops-toolkit
```

7. The Output is as follows

```yaml

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/devops-toolkit   3/3     3            3           31s

NAME                     TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/devops-toolkit   NodePort   10.108.222.194   <none>        80:32674/TCP   31s

kubectl get pods | grep devops-toolkit
devops-toolkit-597c5c9496-fjwd7   1/1     Running   0          5m5s
devops-toolkit-597c5c9496-ncxdd   1/1     Running   0          5m5s
devops-toolkit-597c5c9496-tcqgp   1/1     Running   0          5m5s
```
This application follows similar logic to the first. From the latter command, we can see that it contains a Deployment and a Service. 

## Understanding the Process

- Let’s check whether the new application is indeed reachable

```yaml
http://$IP:$PORT

Example : http://34.89.204.43:32674/
```
- We retrieved the port of the new Service and opened the application in a browser.

![svc-20.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc20.png)

- A simplified flow of requests is depicted in the below-given illustration.

  -  A user sends a request to one of the nodes of the cluster. That request is received by a Service and load balanced to one of the associated Pods.

![svc-21.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/svc21.png)


- We cannot expect our users to know specific ports behind each of those applications. Even with only two, that would not be very user-friendly. If that number would rise to tens or even hundreds of applications, our business would be very short-lived.

What we need is a way to make all services accessible through standard HTTP (80) or HTTPS (443) ports. Kubernetes Services alone cannot get us there. We need more.

## The Solution 

- What we need is to grant access to our services on predefined paths and domains. Our `go-demo-2 service` could be distinguished from others through the base path `/demo`. Similarly, the books application could be reachable through the devopstoolkitseries.com domain. If we could accomplish that, we could access them with the commands as follows.

- The request received the default backend - 404 response. There is no process listening on port 80, so this outcome is not a surprise. We could have changed one of the Services to publish the fixed port 80 instead assigning a random one. Still, that would provide access only to one of the two applications.

## Why Ingress Controllers are Required?

- We need a mechanism that will accept requests on pre-defined ports (e.g., 80 and 443) and forward them to Kubernetes Services.
- It should be able to distinguish requests based on paths and domains.
- Kubernetes itself does not have a ready-to-go solution for this. Unlike other types of   Controllers that are typically part of the kube-controller-manager binary, Ingress Controller needs to be installed separately. Instead of a Controller, kube-controller-manager offers Ingress resource that other third-party solutions can utilize to provide requests forwarding and SSL features. In other words, Kubernetes only provides an API, and we need to set up a Controller that will use it.

## What is an Ingress?

- In Kubernetes, an Ingress is an object that allows access to your Kubernetes services from outside the Kubernetes cluster. You configure access by creating a collection of rules that define which inbound connections reach which services.

- This lets you consolidate your routing rules into a single resource. For example, you might want to send requests to example.com/api/v1/ to an api-v1 service, and requests to example.com/api/v2/ to the api-v2 service. With an Ingress, you can easily set this up without creating a bunch of LoadBalancers or exposing each service on the Node.


### Kubernetes Ingress vs LoadBalancer vs NodePort

These options all do the same thing. They let you expose a service to external network requests. 
They let you send a request from outside the Kubernetes cluster to a service inside the cluster.



### NodePort 

![](https://raw.githubusercontent.com/collabnix/kubelabs/master/Ingress101/nodeport.png)

- NodePort is a configuration setting you declare in a service’s YAML. Set the service spec’s type to NodePort. Then, Kubernetes will allocate a specific port on each Node to that service, and any request to your cluster on that port gets forwarded to the service.

- This is cool and easy, it’s just not super robust. You don’t know what port your service is going to be allocated, and the port might get re-allocated at some point.

### LoadBalancer

![](https://raw.githubusercontent.com/collabnix/kubelabs/master/Ingress101/loadbalancer.png)

- You can set a service to be of type LoadBalancer the same way you’d set NodePort— specify the type property in the service’s YAML. There needs to be some external load balancer functionality in the cluster, typically implemented by a cloud provider.

- This is typically heavily dependent on the cloud provider—GKE creates a Network Load Balancer with an IP address that you can use to access your service.

- Every time you want to expose a service to the outside world, you have to create a new LoadBalancer and get an IP address.

## Ingress

![](https://raw.githubusercontent.com/collabnix/kubelabs/master/Ingress101/ingress.png)

- NodePort and LoadBalancer let you expose a service by specifying that value in the service’s type. Ingress, on the other hand, is a completely independent resource to your service. You declare, create and destroy it separately to your services.

- This makes it decoupled and isolated from the services you want to expose. It also helps you to consolidate routing rules into one place.

- The one downside is that you need to configure an Ingress Controller for your cluster. But that’s pretty easy—in this example, we’ll use the Nginx Ingress Controller.

## How to Use Nginx Ingress Controller

  - Start by creating the “mandatory” resources for Nginx Ingress in your cluster.
 
 ```
 kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
 ```
 
 - Then, enable the ingress add-on for Minikube
 
 ```
 minikube addons enable ingress
  ```
 
 - Or, if you’re using Docker for Mac to run Kubernetes instead of Minikube.
 
 ```
 kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/cloud-generic.yaml
  ```
 
 - Check that it’s all set up correctly.
 
 ```
 kubectl get pods --all-namespaces -l app=ingress-nginx
 ```
### Creating a Kubernetes Ingress

- First, let’s create two services to demonstrate how the Ingress routes our request. We’ll run two web applications that output a slightly different response.

```
git clone https://github.com/collabnix/kubelabs
cd ingress101
$ kubectl apply -f apple.yaml
$ kubectl apply -f banana.yaml
```

- Create the Ingress in the cluster

```
kubectl create -f ingress.yaml
```

Perfect! Let’s check that it’s working. If you’re using Minikube, you might need to replace localhost with minikube IP.

```
$ curl -kL http://localhost/apple
apple

$ curl -kL http://localhost/banana
banana

$ curl -kL http://localhost/notfound
default backend - 404
```

## Ingress Controllers and Ingress Resources

- Kubernetes supports a high level abstraction called Ingress, which allows simple host or URL based HTTP routing. An ingress is a core concept (in beta) of Kubernetes, but is always implemented by a third party proxy. These implementations are known as ingress controllers. An ingress controller is responsible for reading the Ingress Resource information and processing that data accordingly. Different ingress controllers have extended the specification in different ways to support additional use cases.

- Ingress is tightly integrated into Kubernetes, meaning that your existing workflows around kubectl will likely extend nicely to managing ingress. Note that an ingress controller typically doesn’t eliminate the need for an external load balancer — the ingress controller simply adds an additional layer of routing and control behind the load balancer.


# Contributors

[Sangam Biradar](https://twitter.com/BiradarSangam)

