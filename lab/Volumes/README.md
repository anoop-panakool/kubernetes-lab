## Kubernetes storage

Storage is critical to most real-world production applications. Fortunately, Kubernetes has a mature and featurerich storage subsystem called the persistent volume subsystem.

- Kubernetes supports lots of types of storage from lots of different places. For example, iSCSI,SMB, NFS, and object storage blobs, all from a variety of external storage systems that can be in the cloud or in your on-premises data center. However, no matter what type of storage you have, or where it comes from, when it’s exposed on your Kubernetes cluster it’s called a volume. For example, Azure File resources surfaced in Kubernetes are called volumes, as are block devices from AWS Elastic Block Store. All storage on a Kubernetes cluster is called a volume.

![storag1](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/storag1.jpg)

- On the left, you’ve got storage providers. They can be your traditional enterprise storage arrays from vendors like EMC and NetApp, or they can be cloud storage services such as AWS Elastic Block Store (EBS) and GCE Persistent Disks (PD). All you need, is a plugin that allows their storage resources to be surfaced as volumes in Kubernetes.

- In the middle of the diagram is the plugin layer Container Storage Interface (CSI). In the simplest terms, this is the glue that connects external storage with Kubernetes.

- On the right of Figure is the Kubernetes persistent volume subsystem. This is a set of API objects that allow applications to consume storage. At a high-level, Persistent Volumes (PV) are how you map external storage onto the cluster, and Persistent Volume Claims (PVC) are like tickets that authorize applications (Pods) to use a PV.

### Let’s assume the quick example shown in Figure

A Kubernetes cluster is running on AWS and the AWS administrator has created a 25GB EBS volume called “ebs-vol”. The Kubernetes administrator creates a PV called “k8s-vol” that links back to the “ebs-vol” via the kubernetes.io/aws-ebs plugin. While that might sound complicated, it’s not. The PV is simply a way of representing the external storage on the Kubernetes cluster. Finally, the Pod uses a PVC to claim access to the PV and start using it.

![storage2](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/storage2.jpg)



### Storage Providers

Kubernetes can use storage from a wide range of external systems. These will often be native cloud services such as `AWSElasticBlockStore` or `AzureDisk`, but they can also be traditional on-premises storage arrays providing `iSCSI` or `NFS` volumes.

Refer this [volumes](https://kubernetes.io/docs/concepts/storage/volumes/) page to know more about storage type and provider.

### The Kubernetes persistent volume

From a day-to-day perspective, this is where you’ll spend most of your time configuring and interacting with Kubernetes storage.
You uses the resources provided by the persistent volume subsystem to leverage and use the storage in your apps.

![storage3](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/storage3.jpg)

- The three main resources in the persistent volume subsystem are:
  - Persistent Volumes (PV)
  - Persistent Volume Claims (PVC)
  - Storage Classes (SC)

## Let’s walk through a quick example.

Assume you have a Kubernetes cluster and an external storage system. The storage vendor provides a CSI plugin so that you can leverage its storage assets inside of your Kubernetes cluster. You provision 3 x 10GB volumes on the storage system and create 3 Kubernetes PV objects to make them available on your cluster. Each PV references one of the volumes on the storage array via the CSI plugin. At this point, the three volumes are visible and available for use on the Kubernetes cluster.

Now assume you’re about to deploy an application that requires 10GB of storage. That’s great, you already have three 10GB PVs. In order for the app to use one of them, it needs a PVC. As previously mentioned, a PVC is like  a ticket that lets a Pod (application) use a PV. Once the app has the PVC, it can mount the respective PV into its Pod as a volume. Refer back to Figure 8.2 if you need a visual representation.

That was a high-level example. Let’s do it.
 





### Exercise
Pods in Kubernetes are ephemeral, which makes the local container filesytem unusable, as you can never ensure the pod will remain. To decouple your storage from your pods, you will be creating a persistent volume to mount for use by your pods. You will be deploying a redis image. You will first create the persistent volume, then create the pod YAML for deploying the pod to mount the volume. You will then delete the pod and create a new pod, which will access that same volume.

![storage4](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/storage4.png)

- You have been given access to a three-node cluster. Your objective is to create persistent storage for the pod, and prove that the data resides on disk, even when you delete the pod. You must first create a PersistentVolume object in Kubernetes. Once the PersistentVolume has been created, you must create a PersistentVolumeClaim in order for you to claim that volume for the pod. Once you have your PersistentVolume and PersistentVolumeClaim, you are now ready to create the pod.

- Create the pod with the image redis and include the volume, mounted to the /data directory. Also, ensure that port 6379 is open on the container. Once you've created the pod, connect to it and write some data to the database using the redis-cli, then use the QUIT command to exit the redis-cli. Then, delete the pod and create a new pod that will mount that same volume. Connect to the new redis pod and retreive the data that you wrote to the database on the first pod. This will prove that the data persists beyond the life of the pod. Perform the following tasks in order to complete this hands-on lab:

  -  Create a PersistentVolume named redis-pv with 1Gi of storage and ReadWriteOnce access mode. Mount the PersistentVolume to the /mnt/data directory on the host
  - Create a PersistentVolumeClaim named redisdb-pvc with a request for 1Gi of storage. Make sure it has the same access mode as the PersistentVolume
  - Create a pod named redispod using the redis image and a container name of redisdb. The mount path on the container must be /data and you must open port 6379 on the container. Make sure you mount the same PVC that you created in the last step
  - Connect to the container by running the redis-cli command from within the container. Use the SET command to apply the key space server:name as "redis server". Use the GET command to verify that the server:name keyspace has been set. Use the QUIT command to exit the redis-cli
  - Delete the pod named redispod
  - Open the pod yaml and change the name of the pod to redispod2. Create the new pod with that YAML spec
  - Connect to the redispod2 container and run the redis-cli from within the container. Run the command GET server:name to retrieve the keyspace that we set with the previous pod. The result will be "redis server", which means the data persisted from redispod to redispod2

- Having persistent storage means that we can use that same volume with different pods in our Kubernetes cluster and access the same data.

 #### Creating Persistent Storage for Pods in Kubernetes,In this hands-on lab, to decouple our storage from our pods, we will create a persistent volume to mount for use by our pods. We will deploy a redis image. We will first create the persistent volume, then create the pod YAML for deploying the pod to mount the volume. We will then delete the pod and create a new pod, which will access that same volume.


- Create a `PersistentVolume`

1. Create the file, named [redis-pv.yaml](/lab/Volumes/redis-pv.yaml)
```yaml
vim redis-pv.yaml
```

2. Use the following YAML spec for the `PersistentVolume`
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv
spec:
  storageClassName: ""
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

3. Then, create the `PersistentVolume`
```yaml
kubectl apply -f redis-pv.yaml
```

- Create a `PersistentVolumeClaim`

4. Create the file, named [redis-pvc.yaml](/lab/Volumes/redis-pvc.yaml)

```yaml
vim redis-pvc.yaml
```

5. Use the following `YAM`L spec for the `PersistentVolumeClaim`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redisdb-pvc
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
 ```

6. Then, create the `PersistentVolumeClaim`

```yaml
kubectl apply -f redis-pvc.yaml
```

- Create a pod from the `redispod` image, with a mounted volume to mount path `/data`

7. Create the file, named [redispod.yaml](/lab/Volumes/redispod.yaml)
```
vim redispod.yaml
```

8. Use the following YAML spec for the pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redispod
spec:
  containers:
  - image: redis
    name: redisdb
    volumeMounts:
    - name: redis-data
      mountPath: /data
    ports:
    - containerPort: 6379
      protocol: TCP
  volumes:
  - name: redis-data
    persistentVolumeClaim:
      claimName: redisdb-pvc
```

9. Then, create the pod
```yaml
kubectl apply -f redispod.yaml
```

10. Verify the pod was created
```yaml
kubectl get pods
```

- Connect to the container and write some data.

11. Connect to the container and run the `redis-cli`
```yaml
kubectl exec -it redispod redis-cli
```

12. Set the key space `server:name` and value "redis server"
```yaml
SET server:name "redis server"
```

13. Run the `GET` command to verify the value was set:
```yaml
GET server:name
```

14. Exit the `redis-cli`
```yaml
QUIT
```
- Delete `redispod` and create a new pod named `redispod2`

15. Delete the existing `redispod`
```yaml
kubectl delete pod redispod
```

16. Open the file [redispod.yaml](/lab/Volumes/redispod.yaml) and change line 4 from `name: redispod` to `name: redispod2`

17. Create a new pod named `redispod2`
```yaml
kubectl apply -f redispod.yaml
```

- Verify the volume has persistent data.

19. Connect to the container and run `redis-cli`
```yaml
kubectl exec -it redispod2 redis-cli
```

20. Run the GET command to retrieve the data written previously:
```yaml
GET server:name
```

13. Exit the `redis-cli`
```yaml
QUIT
```
