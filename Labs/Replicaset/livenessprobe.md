### liveness probes & rediness probes

#### `Kubernetes can check if a container is still alive through liveness probes. You can specify a liveness probe for each container in the pod’s specification. Kubernetes will periodically execute the probe and restart the container if the probe fails.`

#### Kubernetes can probe a container using one of the three mechanisms:

- An HTTP GET probe performs an HTTP GET request on the container’s IP address, a port and path you specify. If the probe receives a response, and the response code doesn’t represent an error (in other words, if the HTTP response code is 2xx or 3xx), the probe is considered successful. If the server returns an error response code or if it doesn’t respond at all, the probe is considered a failure and the container will be restarted as a result.

- A TCP Socket probe tries to open a TCP connection to the specified port of the container. If the connection is established successfully, the probe is successful. Otherwise, the container is restarted.

- An Exec probe executes an arbitrary command inside the container and checks
the command’s exit status code. If the status code is 0, the probe is successful. All other codes are considered failures.

### Create an HTTP-based liveness probe

#### Follow below commnads and observe how pods are getting created 
```
    kubectl apply -f kubia-liveness-probe.yaml
```
 #### Observe the POD
```
    kubectl get po kubia-liveness
```
 #### Check the application log of a crashed container
```
      kubectl logs mypod --previous
```
### You can see why the container had to be restarted by looking at what kubectl describe
``` 
    kubectl describe po kubia-liveness
```
*`You can see that the container is currently running/Waiting, but it previously terminated because of an error.The exit code was 137, which has a special meaning—it denotes that the process was terminated by an external signal The number 137 is a sum of two numbers: 128+x, where x is the signal number sent to the process that caused it to terminate. In the example, x equals 9, which is the number of the SIGKILL signal, meaning the process was killed forcibly.`*

*`The events listed at the bottom show why the container was killed—Kubernetes detected the container was unhealthy, so it killed and re-created it.`*

*`NOTE When a container is killed, a completely new container is created—it’s not the same container being restarted again.`*

### Scale out
```
    kubectl scale rc kubia --replicas=10
```
### Delete without deleting pods 
    kubectl delete rc kubia --cascade=false
   
  # Replica set
   
 ### Replica Set creation 
    kubectl create -f kubia-replicaset.yaml

    kubectl get rs 
    kubectl describe rs
 
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
