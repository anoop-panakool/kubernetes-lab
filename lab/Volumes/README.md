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







## Pods in Kubernetes are ephemeral, which makes the local container filesytem unusable, as you can never ensure the pod will remain. To decouple your storage from your pods, you will be creating a persistent volume to mount for use by your pods. You will be deploying a redis image. You will first create the persistent volume, then create the pod YAML for deploying the pod to mount the volume. You will then delete the pod and create a new pod, which will access that same volume.

![storage4](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/storage4.png)