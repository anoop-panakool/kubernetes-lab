### Deploying a Kubernetes Cluster

####  A proper Kubernetes installation spans multiple physical or virtual machines and requires proper network setup to allow all containers in the cluster to communicate with each other. You can install Kubernetes on your laptop computer, on your organization’s infrastructure, or on virtual machines provided by cloud providers (Google Compute Engine, Amazon EC2,Microsoft Azure, and so on). Most cloud providers now offer managed Kubernetes services, saving you from the hassle of installation and management. Few of the largest cloud providers offer:

- Google offers GKE - [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine)
- Amazon has EKS - [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/)
- Microsoft has AKS – [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/)
- IBM has [IBM Cloud Kubernetes Service](https://www.ibm.com/nl-en/cloud/container-service)
- Alibaba provides the [Alibaba Cloud Container Service](https://www.alibabacloud.com/product/container-service)

Refer to the [kubernetes.io](https://kubernetes.io) website to learn more.

## Using the built-in Kubernetes cluster in Docker Desktop

    If you use macOS or Windows, you can unstall Docker as Desktop to run kubernetes
    It contains a single-node Kubernetes cluster that you canenable via its Settings dialog box. This may be the easiest way for you to start your Kubernetes journey

## Enable Kubernetes In Docker Desktop

    Once you install Docker Desktop installed on your computer, you can start the Kubernetescluster by:
    Clicking the whale icon in the system tray and opening the Settings dialog box.
    Clickthe Kubernetes tab and make sure the Enable Kubernetes checkbox is selected.
    The components that make up the Control Plane run as Docker containers, but they aren’t displayed in the list of running containers when you invoke the docker ps command.To display them: 
    Select the Show system containers checkbox.

![docker-desktop.jpg](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/docker-desktop.jpg)


 ####   Remember the Reset Kubernetes Cluster button if you ever want to reset the cluster toremove all the objects you’ve deployed in it.

 ### Visualize how variuos components in Kubernetes cluster run in Docker Desktop?

![k8s-running-in-docker-desktop.jpg](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/k8s-running-in-docker-desktop.jpg)


## Run a local cluster using Minikube

    You can create a Kubernetes cluster using Minikube, a tool maintained by theKubernetes community.
    The cluster consists of a single node and is suitable for both testing Kubernetes and developing applications locally.
    It normally runs Kubernetes in a Linux VM, but if your computer is Linux-based, it can also deploy Kubernetes directly in your host OS via Docker.
 
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
    1.  Signing up for a Google account if you don’t have one already.
    2. Creating a project in the Google Cloud Platform Console.
    3. Enabling billing. This does require your credit card info, but Google provides a 12-month free trial with a free $300 credit. And they don’t start charging automaticallyafter the free trial is over.
    4. Downloading and installing the Google Cloud SDK, which includes the gcloud tool.
    5. Creating the cluster using the gcloud command-line tool.

### Create a GKE Kubernetes Cluster with 3 Nodes

##### First decide in which geographical region and zone it should be created . Refer to Refer to https://cloud.google.com/compute/docs/regions-zones . 
##### In my case, I use the europe-west3 region based in Frankfurt, Germany. It has three different zones - I’ll use the zone europe-west3-c.
##### The default zone for all gcloud operations can be set with the following command:

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