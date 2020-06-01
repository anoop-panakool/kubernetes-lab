## Kubernetes storage

Storage is critical to most real-world production applications. Fortunately, Kubernetes has a mature and featurerich storage subsystem called the persistent volume subsystem.

- Kubernetes supports lots of types of storage from lots of different places. For example, iSCSI,SMB, NFS, and object storage blobs, all from a variety of external storage systems that can be in the cloud or in your on-premises data center. However, no matter what type of storage you have, or where it comes from, when it’s exposed on your Kubernetes cluster it’s called a volume. For example, Azure File resources surfaced in Kubernetes are called volumes, as are block devices from AWS Elastic Block Store. All storage on a Kubernetes cluster is called a volume.

![storag1](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/storag1.jpg)