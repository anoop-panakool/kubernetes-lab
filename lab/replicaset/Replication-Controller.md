### There are multiple types of Controllers in Kubernetes

# Replication Controller

## Follow below commnads and observe how pods are getting created 

    kubectl create -f kubia-rc.yaml
    kubectl get pods
    kubectl delete pod kubia-53thy
    kubectl get pods
    kubectl get rc

 ### Observe RC

    kubectl describe rc kubia



![delete-rc-pod.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/delete-rc-pod.png)

 ### Remove One Node(Use kubectl drain to remove a node) from cluster and check the status of POD

    kubectl get nodes --show-labels
    kubectl drain <node name> --ignore-daemonsets
    kubectl uncordon <node name>

 ### Modify a pod's label 
 
     kubectl get pods --show-labels
     kubectl label pod <podname> app=foo --overwrite
 
### observe the pods with label column [new pod being created]
  
    kubectl get pods -L app
    kubectl get pods --show-labels
  

  ![remove-rc-pod.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/remove-rc-pod.png)


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
