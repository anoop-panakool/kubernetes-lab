

                               LAB Setup On Ubuntu

We will install and configure a Kubernetes cluster consisting of 1 master and 2 nodes. Once the installation and configuration are complete, we will have a 3-node Kubernetes cluster that uses Calico as the network overlay.

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
 
                  Note: Complete the following section on both MASTER & Worker Node !
                              
## Step#1 Add the iptables rule to sysctl.conf ,As a requirement for your Linux Node’s iptables to correctly see bridged traffic, you should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config

      cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      EOF
          
      setenforce 0

## Step#2 Enable iptables immediately
      
      sysctl --system
      sysctl -p

## Step#3 Install Docker runtime, To run containers in Pods, Kubernetes uses a container runtime.
       
      # (Install Docker CE)
      ## Set up the repository:
      ### Install packages to allow apt to use a repository over HTTPS

      apt-get update && apt-get install -y \
        apt-transport-https ca-certificates curl software-properties-common gnupg2

      # Add Docker’s official GPG key

      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
      
      # Add the Docker apt repository

      add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"


      # Install Docker CE

      apt-get update && apt-get install -y \
        containerd.io=1.2.13-1 \
        docker-ce=5:19.03.8~3-0~ubuntu-$(lsb_release -cs) \
        docker-ce-cli=5:19.03.8~3-0~ubuntu-$(lsb_release -cs)

      # Set up the Docker daemon

      cat > /etc/docker/daemon.json <<EOF
      {
        "exec-opts": ["native.cgroupdriver=systemd"],
        "log-driver": "json-file",
        "log-opts": {
        "max-size": "100m"
       },
       "storage-driver": "overlay2"
      }
      EOF

      mkdir -p /etc/systemd/system/docker.service.d

      # Restart Docker

      systemctl daemon-reload
      systemctl restart docker

## Step#4 Installing kubeadm, kubelet and kubectl

    Install these packages on all of your machines:

    kubeadm: the command to bootstrap the cluster.
    kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
    kubectl: the command line util to talk to your cluster.

      apt-get update && sudo apt-get install -y apt-transport-https curl

      ## Get the Kubernetes gpg key

      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

      ## Add the Kubernetes repository

      cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
      deb https://apt.kubernetes.io/ kubernetes-xenial main
      EOF

      ## downloads the package lists from the repositories and Update your packages & dependecies to the newest versions

      apt-get update

      ## Install Docker, kubelet, kubeadm, and kubectl

      apt-get install -y kubelet kubeadm kubectl

      ## Hold them at the current version
      apt-mark hold kubelet kubeadm kubectl


                Note: Complete the following section on the MASTER Node ONLY!

## Step#5 Initialize the Kubernetes cluster.In the master node, run below command to initialize the cluster using kubeadm


       sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=NumCPU
       sudo kubeadm init --pod-network-cidr=192.168.0.0/16 #Do this only if proper CPU cores are available

## Step#6 Set up local kubeconfig ,To start using your cluster, you need to run the following as a regular user
       mkdir -p $HOME/.kube
       sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
       sudo chown $(id -u):$(id -g) $HOME/.kube/config


                Note: Complete the following steps on the WORKER NODES ONLY!
         
## Step#7 Join the worker nodes to the cluster, Copy kubeadm join command from output of "kubeadm init on master node" on each WORKER NODE
        
        ## Example - kubeadm join 172.31.24.221:6443 --token pexa5a.4zk3o0xs7e0bq4ip \
                       --discovery-token-ca-cert-hash sha256:d4d3276b15704711ad682c76a195ceca754304ffc16328c869de9448821fa59a

        <kubeadm join command copies from master node>


                Note: Complete the following section on the MASTER Node ONLY!

## Step#8 Apply Calico CNI network overlay , On Master Node only

    apt install git -y
    git clone https://github.com/shivamjhalabfiles/kubernetes-lab/tree/master/calico
    cd kubernetes-lab/calico/
    
    kubectl apply -f .


## Step#9 Verify the worker nodes have joined the cluster successfully

    kubectl get nodes

    root@K8s-master:~# kubectl get nodes
    NAME                    STATUS   ROLES    AGE   VERSION
    master.labserver.com    Ready    master   75m   v1.18.3
    worker1.labserver.com   Ready    <none>   73m   v1.18.3
    worker2.labserver.com   Ready    <none>   73m   v1.18.3

    root@K8s-master:~# kubectl get nodes -o wide
    NAME                    STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
    master.labserver.com    Ready    master   76m   v1.18.3   172.31.24.221   <none>        Ubuntu 18.04.4 LTS   5.3.0-1017-aws   docker://19.3.8
    worker1.labserver.com   Ready    <none>   74m   v1.18.3   172.31.31.18    <none>        Ubuntu 18.04.4 LTS   5.3.0-1017-aws   docker://19.3.8
    worker2.labserver.com   Ready    <none>   73m   v1.18.3   172.31.16.84    <none>        Ubuntu 18.04.4 LTS   5.3.0-1017-aws   docker://19.3.8
 

    kubectl get pods -n kube-system

    root@K8s-master:~# kubectl get pods -n kube-system
    NAME                                               READY   STATUS    RESTARTS   AGE
    calico-etcd-4wvpn                                  1/1     Running   0          74m
    calico-kube-controllers-7997dc8d7b-886d8           1/1     Running   1          75m
    calico-node-gf4zk                                  2/2     Running   2          75m
    calico-node-jjcwn                                  2/2     Running   2          75m
    calico-node-v85fh                                  2/2     Running   2          75m
    coredns-66bff467f8-4c9px                           1/1     Running   0          80m
    coredns-66bff467f8-th4bd                           1/1     Running   0          80m
    etcd-master.labserver.com                          1/1     Running   0          81m
    kube-apiserver-master.labserver.com                1/1     Running   0          81m
    kube-controller-manager-master.labserver.com       1/1     Running   0          81m
    kube-proxy-c5dfq                                   1/1     Running   0          78m
    kube-proxy-tdzsb                                   1/1     Running   0          79m
    kube-proxy-tlgn8                                   1/1     Running   0          80m
    kube-scheduler-master.labserver.com                1/1     Running   0          81m


## Step#10 On all the worker nodes do

    mkdir -p $HOME/.kube
    export KUBECONFIG=/etc/kubernetes/kubelet.conf
