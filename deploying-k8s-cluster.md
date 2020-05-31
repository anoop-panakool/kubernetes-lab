### Deploying a Kubernetes Cluster

A proper Kubernetes installation spans multiple physical or virtual machines and requires proper network setup to allow all containers in the cluster to communicate with each other. You can install Kubernetes on your laptop computer, on your organization’s infrastructure, or on virtual machines provided by cloud providers (Google Compute Engine, Amazon EC2,Microsoft Azure, and so on). Most cloud providers now offer managed Kubernetes services, saving you from the hassle of installation and management. Few of the largest cloud providers offer:

- Google offers GKE - [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine)
- Amazon has EKS - [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/)
- Microsoft has AKS – [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/)
- IBM has [IBM Cloud Kubernetes Service](https://www.ibm.com/nl-en/cloud/container-service)
- Alibaba provides the [Alibaba Cloud Container Service](https://www.alibabacloud.com/product/container-service)

Refer to the [kubernetes.io](https://kubernetes.io) website to learn more.

## Using the built-in Kubernetes cluster in Docker Desktop

If you use macOS or Windows, you can unstall Docker as Desktop to run kubernetes. It contains a single-node Kubernetes cluster that you canenable via its Settings dialog box. This may be the easiest way for you to start your Kubernetes journey.

## Enable Kubernetes In Docker Desktop

Once you install Docker Desktop installed on your computer, you can start the Kubernetescluster by:
- Clicking the whale icon in the system tray and opening the Settings dialog box.
- Clickthe Kubernetes tab and make sure the Enable Kubernetes checkbox is selected.
- The components that make up the Control Plane run as Docker containers, but they aren’t displayed in the list of running containers when you invoke the docker ps command.To display them:
- Select the Show system containers checkbox.

![docker-desktop.jpg](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/docker-desktop.jpg)


 Remember the Reset Kubernetes Cluster button if you ever want to reset the cluster toremove all the objects you’ve deployed in it.

 ### Visualize how variuos components in Kubernetes cluster run in Docker Desktop?

![k8s-running-in-docker-desktop.jpg](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/k8s-running-in-docker-desktop.jpg)


## Run a local cluster using Minikube

You can create a Kubernetes cluster using Minikube, a tool maintained by theKubernetes community. The cluster consists of a single node and is suitable for both testing Kubernetes and developing applications locally.It normally runs Kubernetes in a Linux VM, but if your computer is Linux-based, it can also deploy Kubernetes directly in your host OS via Docker.
 
### Install Minikube
  
Minikube supports macOS, Linux, and Windows.
It has a single binary executable file, whichy ou’ll find in the Minikube repository on GitHub (http://github.com/kubernetes/minikube).
Follow the current installation instructions published there.
- On macOS you can install it using the Brew Package Manager,
- On Windows there is an installer that you can download, and
- On Linux you can either download a .deb or .rpmpackage or simply download the binary file and make it executable with the following command:

    ```
    $ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube
      [CA] -linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/mini
      [CA] kube
    ```
### Start a Kubernetes Cluster with Minikube
    
    $ minikube start
>   The process may take several minutes, because the VM image and the container images of the Kubernetes components must be downloaded.

### Check the Minikube status

    $ minikube status

  ![minikube-start.jpg](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/minikube-start.png)

### Visualize how a single-node Kubernetes cluster using Minikube is running?
   
    The architecture of the system, in this figure, is practically identical to Docker Desktop.

![Kubernetes-cluster-using-Minikube.jpg](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/Kubernetes-cluster-using-Minikube.jpg)


## Create a managed cluster with Google Kubernetes Engine

    If you want to use a full-fledged multi-node Kubernetes cluster instead of a local one, you canuse a managed cluster, such as the one provided by Google Kubernetes Engine (GKE). In GKE you don’t have to manually set up all the cluster nodes and networking.

### Set-up Google Cloud and Installing the GCloud client binary

First set up your GKE environment

Below are the procedure:  
1. Signing up for a Google account if you don’t have one already.
2. Creating a project in the Google Cloud Platform Console.
3. Enabling billing. This does require your credit card info, but Google provides a 12-month free trial with a free $300 credit. And they don’t start charging automaticallyafter the free trial is over.
4. Downloading and installing the Google Cloud SDK, which includes the gcloud tool.
5. Creating the cluster using the gcloud command-line tool.

### Create a GKE Kubernetes Cluster with 3 Nodes

First decide in which geographical region and zone it should be created . Refer to Refer to https://cloud.google.com/compute/docs/regions-zones .In my case, I use the europe-west3 region based in Frankfurt, Germany. It has three different zones - I’ll use the zone europe-west3-c.The default zone for all gcloud operations can be set with the following command:

    $ gcloud config set compute/zone europe-west3-c

 #### Create the Kubernetes cluster using the command `gcloud container clusters create`. You can choose any name other than shivam. 
    
    
    $ gcloud container clusters create shivam --num-nodes 3

    Creating cluster shivam in europe-west3-c...
    ...
    kubeconfig entry generated for shivam.
    NAME LOCAT. MASTER_VER MASTER_IP MACH_TYPE ... NODES STATUS
    shivam eu-w3-c 1.13.11... 5.24.21.22 n1-standard-1 ... 3 RUNNING

#### NOTE- I have all three worker nodes in the same zone, but you can also spread them across all zones in the region by setting the compute/zone config value to an entire region instead of a single zone. If you do, note that --num-nodes indicates the number of nodes per zone. If the region contains three zones and you only want three nodes, you must set --num-nodes to 1.
    
### List the GCE virtual machines

    $ gcloud compute instances list

    NAME ZONE MACHINE_TYPE INTERNAL_IP EXTERNAL_IP STATUS
    ...-ctlk eu-west3-c n1-standard-1 10.156.0.16 34.89.238.55 RUNNING
    ...-gj1f eu-west3-c n1-standard-1 10.156.0.14 35.242.223.97 RUNNING
    ...-r01z eu-west3-c n1-standard-1 10.156.0.15 35.198.191.189 RUNNING

#### NOTE-  Each VM incurs costs. To reduce the cost of your cluster, you can reduce the number of nodes to one, or even to zero while not using it. See next section for details.

![GoogleKubernetesEngine.jpg](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/GoogleKubernetesEngine.jpg)


### How to scale the number of nodes in GKE?

##### We can easily increase or decrease the number of nodes in your cluster

##### To scale the cluster to zero, use the below command:

    $ gcloud container clusters resize kubia --size 0


##  Running your first application on Kubernetes

Once Kubernetes Cluster is up and running then deploy something to your cluster.To deploy an application you have to prepare a JSON or YAML file describing all the components that your application consists of and apply that file to your cluster. This would be the declarative approach.
Since this may be our first time deploying an application to Kubernetes, let’s choose an easier way to do this. We’ll use simple, one-line imperative commands to deploy our application.

#### Deploying application using imperative way, use the *`kubectl create deployment`* command
By default, the image is pulled from Docker Hub, but you can also specify the image registry in the image name (for example, quay.io/).
### 
Make sure that the image is stored in a public registry and can be pulled without access authorization.

#### Create Deployment
    
    $ kubectl create deployment shivam --image=luksa/kubia
    deployment.apps/shivam created

    $ kubectl get deployments

    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    shivam   0/1     1            0           88s

    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    shivam   1/1     1            1           88s
     
    $ kubectl get containers
    error: the server doesn't have a resource type "containers"

##  Introduction of PODS

In Kubernetes, instead of deploying individual containers, you deploy groups of co-located containers – so-called pods.
A pod is a group of one or more closely related containers that run together on the same worker node and need to share certain Linux namespaces, so that they can interact more closely than with other pods.

In this picture we can see, the relationship between containers, pods, and worker nodes

![pod-containers-workernode-relation](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/pod-containers-workernode-relation.jpg)

#### List the PODS

    $ kubectl get pods
    NAME                      READY   STATUS    RESTARTS   AGE
    shivam-7878fc7498-wccd9   1/1     Running   0          20m

#### To see more information about the pod, use the `kubectl describe pod` command,

```yaml
root@master:~# kubectl describe pod shivam-7878fc7498-wccd9
Name:         shivam-7878fc7498-wccd9
Namespace:    default
Priority:     0
Node:         shivam2c.mylabserver.com/172.31.109.218
Start Time:   Sat, 30 May 2020 22:01:23 +0000
Labels:       app=shivam
              pod-template-hash=7878fc7498
Annotations:  <none>
Status:       Running
IP:           10.244.1.52
IPs:
  IP:           10.244.1.52
Controlled By:  ReplicaSet/shivam-7878fc7498
Containers:
  kubia:
    Container ID:   docker://af0e32ff383beec7b3a65206655d374edb6d39d7f5e7271cf4dfc30fe2663527
    Image:          luksa/kubia
    Image ID:       docker-pullable://luksa/kubia@sha256:3f28e304dc0f63dc30f273a4202096f0fa0d08510bd2ee7e1032ce600616de24
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 30 May 2020 22:01:25 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4dfq6 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-4dfq6:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-4dfq6
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                               Message
  ----    ------     ----  ----                               -------
  Normal  Scheduled  23m   default-scheduler                  Successfully assigned default/shivam-7878fc7498-wccd9 to shivam2.labserver.com
  Normal  Pulling    23m   kubelet, shivam2.labserver.com  Pulling image "luksa/kubia"
  Normal  Pulled     23m   kubelet, shivam2.labserver.com  Successfully pulled image "luksa/kubia"
  Normal  Created    23m   kubelet, shivam2.labserver.com  Created container kubia
  Normal  Started    23m   kubelet, shivam2.labserver.com  Started container kubia
```

#### Visualize what happened when you created the Deployment

How creating a Deployment object results in a running application container ?

![visualize-deployment](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/visualize-deployment.jpg)

1. When you ran the kubectl create command, it created a new Deployment object in the cluster by sending an HTTP request to the Kubernetes API server.

2. Kubernetes then created a new Pod object, which was then assigned or scheduled to one of the worker nodes.

3. The Kubernetes agent on the worker node (the Kubelet) became aware of the newly created Pod object, saw that it was scheduled to its node, and instructed Docker to pull the specified image from the registry, create a container from the image, and execute it.

### Expose application to the world

Since our application is now running,but how to access it ?

To make the pod accessible externally, you have to expose it but how ?

#### Creating a Service

    $ kubectl expose deployment shivam --type=LoadBalancer --port 8080
    service/shivam exposed

#### List the Services

    $ kubectl get services
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
    kubernetes   ClusterIP      10.96.0.1     <none>        443/TCP          2d2h
    shivam       LoadBalancer   10.99.22.40   <pending>     8080:31984/TCP   2m16s

NOTE:  Notice the use of the abbreviation `svc` instead of `services`. Most resource types have a short name that you can use instead of the full object type (for example, `po` is short for `pods`, `no` for `nodes` and `deploy` for `deployments`).



![loadBalancer-works](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/loadBalancer-works.jpg)

    $ kubectl get services
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP       PORT(S)          AGE
    kubernetes   ClusterIP      10.96.0.1     <none>            443/TCP          2d2h
    shivam       LoadBalancer   10.245.79.24   64.225.80.243     8080:32125/TCP   2m16s

#### Accessing the application through Load Balancer
    Go to the broser and enter <loadbalacer-external-ip:8080>
Result will be as below
    You've hit shivam-7d448449b4-f9hds

#### Accessing the application when a Load Balancer is not available
If Using Minikube

    $ minikube service shivam --url
    http://192.168.99.102:30838

#### Access the application via the node ports.

    $ curl <worker-node-ip>:30838
    http://worker-node-ip:30838
![service-nodeport](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/service-nodeport.jpg)

#### Horizontally scaling the application

Increase the Number of running Application instances
    $ kubectl scale deployment shivam --replicas=3
    deployment.apps/shivam scaled

#### See the result of the Scale-Out

    $ kubectl get deploy
    NAME      READY   UP-TO-DATE   AVAILABLE   AGE
    shivam    3/3     3            3            18m

#### List the PODS

    $ kubectl get pods
    NAME                     READY   STATUS    RESTARTS   AGE
    shivam-9d785b578-58vhc   1/1     Running   0          17s
    shivam-9d785b578-jmnj8   1/1     Running   0          17s
    shivam-9d785b578-p449x   1/1     Running   0          18m

#### Display the PODS host node when listing PODS

    $ kubectl get pods -o wide
    NAME                     READY   STATUS    RESTARTS   AGE    IP             NODE
    shivam-9d785b578-58vhc   1/1     Running   0          17s    10.16.2.5      shivam2.labserver.com
    shivam-9d785b578-jmnj8   1/1     Running   0          17s    10.16.2.4      shivam3.labserver.com
    shivam-9d785b578-p449x   1/1     Running   0          18m    10.16.2.3      shivam2.labserver.com

#### Observer the requests hitting all three pods when using the service

    $ curl 35.246.179.22:8080
    You've hit shivam-7878fc7498-tds4z
    $ curl 35.246.179.22:8080
    You've hit shivam-7878fc7498-wccd9
    $ curl 35.246.179.22:8080
    You've hit shivam-7878fc7498-tds4z
    $ curl 35.246.179.22:8080
    You've hit shivam-7878fc7498-p449x

![verify-lb-request](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/verify-lb-request.jpg)

### Understanding the deployed application

The logical view consists of the objects you’ve created in the Kubernetes API – either directly or indirectly. This figure shows how the objects relate to each other.

Our deployed application consists of a Deployment, several Pods, and a Service.

The objects are as follows:
- the Deployment object you created,
- the Pod objects that were automatically created based on the Deployment, and
- the Service object you created manually.

![logical-view-of-application-deployed](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/logical-view-of-application-deployed.jpg)



## Examples: Common operations

## Create a Deployment

A Kubernetes [*Pod*](/docs/concepts/workloads/pods/pod/) is a group of one or more Containers,
tied together for the purposes of administration and networking. The Pod in this
tutorial has only one Container. A Kubernetes
[*Deployment*](/docs/concepts/workloads/controllers/deployment/) checks on the health of your
Pod and restarts the Pod's Container if it terminates. Deployments are the
recommended way to manage the creation and scaling of Pods.

1. Use the `kubectl create` command to create a Deployment that manages a Pod. The
Pod runs a Container based on the provided Docker image.

    ```shell
    kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
    ```

2. View the Deployment:

    ```shell
    kubectl get deployments
    ```

    The output is similar to:

    ```
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    hello-node   1/1     1            1           1m
    ```

3. View the Pod:

    ```shell
    kubectl get pods
    ```

    The output is similar to:

    ```
    NAME                          READY     STATUS    RESTARTS   AGE
    hello-node-5f76cf6ccf-br9b5   1/1       Running   0          1m
    ```

4. View cluster events:

    ```shell
    kubectl get events
    ```

5. View the `kubectl` configuration:

    ```shell
    kubectl config view
    ```

#### NOTE
For more information about `kubectl`commands, see the [kubectl overview](/docs/user-guide/kubectl-overview/).

## Create a Service

By default, the Pod is only accessible by its internal IP address within the
Kubernetes cluster. To make the `hello-node` Container accessible from outside the
Kubernetes virtual network, you have to expose the Pod as a
Kubernetes [*Service*](/docs/concepts/services-networking/service/).

1. Expose the Pod to the public internet using the `kubectl expose` command:

    ```shell
    kubectl expose deployment hello-node --type=LoadBalancer --port=8080
    ```

    The `--type=LoadBalancer` flag indicates that you want to expose your Service
    outside of the cluster.

2. View the Service you just created:

    ```shell
    kubectl get services
    ```

    The output is similar to:

    ```
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
    kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
    ```

    On cloud providers that support load balancers,
    an external IP address would be provisioned to access the Service. On Minikube,
    the `LoadBalancer` type makes the Service accessible through the `minikube service`
    command.

3. View the Pod and Service you just created:

    ```shell
    kubectl get pod,svc -n kube-system
    ```
## Clean up

Now you can clean up the resources you created in your cluster:

    ```shell
    kubectl delete service hello-node
    kubectl delete deployment hello-node
    ```

Optionally, stop the Minikube virtual machine (VM):

    ```shell
    minikube stop
    ```

Optionally, delete the Minikube VM:

    ```shell
    minikube delete
    ```
## Running end-to-end tests ensures your application will run efficiently without having to worry about cluster health problems.

    Run a simple nginx deployment

```
kubectl run nginx --image=nginx
```
    View the deployments in your cluster
```
kubectl get deployments
```

    View the pods in the cluster
```
kubectl get pods
```

    Use port forwarding to access a pod directly
```
kubectl port-forward $pod_name 8081:80
```

    Get a response from the nginx pod directly
```
curl --head http://127.0.0.1:8081
```

    View the logs from a pod:
```
kubectl logs $pod_name
```

    Run a command directly from the container
```
kubectl exec -it $pod_name -- nginx -v
```

    Create a service by exposing port 80 of the nginx deployment
```
kubectl expose deployment nginx --port 80 --type NodePort
```

    List the services in your cluster
```
kubectl get services
```

    Get the list of Nodes
```
kubectl get nodes
```

    View detailed information about the nodes
```
kubectl describe nodes
```

    View detailed information about the pods
```
kubectl describe pods
```

### You can display a list of all supported types by running kubectl api-resources. The list also shows the short name for each type and some other information you need to define objects in JSON/YAML files

```shell
# List all Kubernetes object types
kubectl api-resources
```
Use the following set of examples to help you familiarize yourself with running the commonly used `kubectl` operations:

`kubectl apply` - Apply or Update a resource from a file or stdin.

```shell
# Create a service using the definition in example-service.yaml.
kubectl apply -f example-service.yaml

# Create a replication controller using the definition in example-controller.yaml.
kubectl apply -f example-controller.yaml

# Create the objects that are defined in any .yaml, .yml, or .json file within the <directory> directory.
kubectl apply -f <directory>
```

`kubectl get` - List one or more resources.

```shell
# List all pods in plain-text output format.
kubectl get pods

# List all pods in plain-text output format and include additional information (such as node name).
kubectl get pods -o wide

# List the replication controller with the specified name in plain-text output format. Tip: You can shorten and replace the 'replicationcontroller' resource type with the alias 'rc'.
kubectl get replicationcontroller <rc-name>

# List all replication controllers and services together in plain-text output format.
kubectl get rc,services

# List all daemon sets in plain-text output format.
kubectl get ds

# List all pods running on node server01
kubectl get pods --field-selector=spec.nodeName=server01
```

`kubectl describe` - Display detailed state of one or more resources, including the uninitialized ones by default.

```shell
# Display the details of the node with name <node-name>.
kubectl describe nodes <node-name>

# Display the details of the pod with name <pod-name>.
kubectl describe pods/<pod-name>

# Display the details of all the pods that are managed by the replication controller named <rc-name>.
# Remember: Any pods that are created by the replication controller get prefixed with the name of the replication controller.
kubectl describe pods <rc-name>

# Describe all pods
kubectl describe pods
```
### Note
The `kubectl get` command is usually used for retrieving one or more resources of the same resource type. It features a rich set of flags that allows you to customize the output format using the `-o` or `--output` flag, for example.
You can specify the `-w` or `--watch` flag to start watching updates to a particular object. The `kubectl describe` command is more focused on describing the many related aspects of a specified resource. It may invoke several API calls to the API server to build a view for the user. For example, the `kubectl describe node`
command retrieves not only the information about the node, but also a summary of the pods running on it, the events generated for the node etc.
`kubectl delete` - Delete resources either from a file, stdin, or specifying label selectors, names, resource selectors, or resources.

```shell
# Delete a pod using the type and name specified in the pod.yaml file.
kubectl delete -f pod.yaml

# Delete all the pods and services that have the label '<label-key>=<label-value>'.
kubectl delete pods,services -l <label-key>=<label-value>

# Delete all pods, including uninitialized ones.
kubectl delete pods --all
```

`kubectl exec` - Execute a command against a container in a pod.

```shell
# Get output from running 'date' from pod <pod-name>. By default, output is from the first container.
kubectl exec <pod-name> -- date

# Get output from running 'date' in container <container-name> of pod <pod-name>.
kubectl exec <pod-name> -c <container-name> -- date

# Get an interactive TTY and run /bin/bash from pod <pod-name>. By default, output is from the first container.
kubectl exec -ti <pod-name> -- /bin/bash
```

`kubectl logs` - Print the logs for a container in a pod.

```shell
# Return a snapshot of the logs from pod <pod-name>.
kubectl logs <pod-name>

# Start streaming the logs from pod <pod-name>. This is similar to the 'tail -f' Linux command.
kubectl logs -f <pod-name>
```

`kubectl diff` - View a diff of the proposed updates to a cluster.

```shell
# Diff resources included in "pod.json".
kubectl diff -f pod.json

# Diff file read from stdin.
cat service.yaml | kubectl diff -f -
```