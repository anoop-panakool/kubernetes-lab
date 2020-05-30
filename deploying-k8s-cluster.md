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

 ### How variuos components in Kubernetes cluster run in Docker Desktop?


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

  ![minikube-start.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/minikube-start.png)


### Scale out (Horizontally scaling pods)

    kubectl scale rc kubia --replicas=10

             OR Declarative way

    kubectl edit rc kubia2
    kubectl get rc

### Scale down to 3

    kubectl scale rc kubia --replicas=3

             OR Declarative way

    kubectl edit rc kubia2
    kubectl get rc

### Delete RC without deleting pods 

    kubectl delete rc kubia --cascade=false

### Change the POD template

    kubectl edit rc kubia
   
 
# Daemon set
 
 ### This daemon set is designed to run on all the disks that has ssd disk (disk=ssd )
      kubectl create -f ssd-monitor-daemonset.yaml
      kubectl get ds
      kubectl get po
      kubectl get node
      kubectl label node <nodename> disk=ssd
 
 ### check it now 
     kubectl get po
  
# job
      kubectl get jobs
      kubectl get po

### After the two minutes have passed, see the status "completed"
     kubectl get po -a
     kubectl logs <jobpodname>

### Sequential completion and parallelism

    kubectl create -f multi-completion-batch-job.yaml
    kubectl create -f multi-completion-parallel-batch-job.yaml
### You can even change a Job’s parallelism property while the Job is running
    kubectl scale job multi-completion-batch-job --replicas 3
