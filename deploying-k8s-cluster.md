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

[pod-containers-workernode-relation] (https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/rpod-containers-workernode-relation.jpg)









## Examples: Common operations

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