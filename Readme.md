# Kubernetes Labs & Project Demo ! 

### kubernetes Lab & Project Files . 

All these Labs were tested on `On-Premises` & `Cloud(GCP,Amazon)`. And should work on other kubernetes environments as well. 

To Setup `Docker Desktop for Windows` on your system, make sure your system has following pre-requisites:

    1.  Virtualization enabled from BIOS / UEFI System.
    2.  Windows 10 PRO (1809 or Newer).
    3.  Window's Optional Feature: HyperV and Container.
    4.  RAM Should be 8 GB or More.
    5.  A Multi core CPU (Intel i5 or better). 

To install docker desktop [follow these steps](https://docs.docker.com/docker-for-windows/install/)

**Student LAB details**

Sr No | Username | Password  | Project Name | Email Id
------|-------------|--------| --------------- |--------|
1   | *Student01* | *June@2020* | *LFS458-Student01* | edi5k8student01@gmail.com
2   | *Student02* | *XXXXXX* | *LFS458-Student02* | edi5k8student01@gmail.com
3   | *Student03* | *JXXXXXX* |*LFS458-Student03* | edi5k8student01@gmail.com
4   | *Student04* | *XXXXXX* | *LFS458-Student04* | edi5k8student01@gmail.com
5   | *Student05* | *XXXXXX* | *LFS458-Student05* | edi5k8student01@gmail.com
6   | *Student06* | *XXXXXX* | *LFS458-Student06* | edi5k8strainer@gmail.com
7   | *Student07* | *XXXXXX* | *LFS458-Student07* | edi5k8strainer@gmail.com
8   | *Student08* | *XXXXXX* | *LFS458-Student08* | edi5k8strainer@gmail.com

> [Treaining Resources](https://training.linuxfoundation.org/cm/LFS258/)

> [Practise Files](https://github.com/shivamjhalabfiles/kubernetes-lab) 

> ### This LAB setup uses `kubeadm` to install and configure kubernetes cluster.

![K8s-cluster](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/K8s-cluster.png)

> Cluster is setup on `GCP Virtual Machines` `Ubuntu-1804-bionic` 1 master and 2 worker-nodes.
![GCP-VM-Instances](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/GCP-VM-Instances.png)

> Login to the Kubernetes Master node *`ssh -i path-to-private-key username@external-ip`*

### Important links:
1. Kuberenetes [Storage](https://docs.microsoft.com/en-us/azure/aks/concepts-storage) from Azure Docs 

2. Service [Types] (https://docs.microsoft.com/en-us/azure/aks/concepts-network from Azure Docs)

3. [RBAC](https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/) in Kubernetes from bitnami.

4.  Deploying Azure [Application Gateway](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/master/docs/setup/install-existing.md) in AKS Cluster. 

5.  [nginx ingress](https://www.nginx.com/products/nginx/kubernetes-ingress-controller) controller for kubernetes.
