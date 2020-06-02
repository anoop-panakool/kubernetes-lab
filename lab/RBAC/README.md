# RBAC - Kubernetes Access Control:  Authentication and Authorization

In  this lab you are going to,

  * Create users and groups and setup certs based authentication
  * Create service accounts for applications
  * Create Roles and ClusterRoles to define authorizations
  * Map Roles and ClusterRoles to subjects i.e. users, groups and service accounts using RoleBingings and ClusterRoleBindings.


## How one can access the Kubernetes API?

The Kubernetes API can be accessed by three ways.

  * Kubectl - A command line utility of Kubernetes
  * Client libraries - Go, Python, etc.,
  * REST requests

## Who can access the Kubernetes API?

Kubernetes API can be accessed by,

  * Human Users
  * Service Accounts  

Each of these topics will be discussed in detail in the later part of this chapter.

## Stages of a Request

When a request tries to contact the API , it goes through various stages as illustrated in the image given below.

  ![request-stages](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/rback01.png)
  <sub>[source: official kubernetes site](https://kubernetes.io/docs/home/)</sub>



## api groups and resources

  | apiGroup     | Resources     |
  | :------------- | :------------- |
  | apps      |   daemonsets, deployments, deployments/rollback, deployments/scale, replicasets, replicasets/scale, statefulsets, statefulsets/scale     |
  |core|configmaps, endpoints, persistentvolumeclaims, replicationcontrollers, replicationcontrollers/scale, secrets, serviceaccounts, services,services/proxy
  |autoscaling|horizontalpodautoscalers
  | batch|cronjobs, jobs
  |policy| poddisruptionbudgets
  |networking.k8s.io|networkpolicies
  |authorization.k8s.io|localsubjectaccessreviews
  |rbac.authorization.k8s.io|rolebindings,roles
  |extensions | deprecated (read notes) |


##### Notes

  In addition to the above apiGroups, you may see **extensions** being used in some example code snippets. Please note that **extensions** was initially created as a experiement and is been deprecated, by moving most of the matured apis to one of the groups mentioned above.  [You could read this comment and the thread](https://github.com/kubernetes/kubernetes/issues/43214#issuecomment-287143011) to get clarity on this.  


## Role Based Access Control (RBAC)

| Group     | User     |  Namespaces     |  Resources     |  Access Type (verbs) |
| :------------- | :------------- |  :------------- | :------------- |    :------------- |
| ops       | maya       | all | all | get, list, watch, update, patch, create, delete, deletecollection |
| dev       | kim       | instavote | deployments, statefulsets, services, pods, configmaps, secrets, replicasets, ingresses, endpoints, cronjobs, jobs, persistentvolumeclaims  | get, list , watch, update, patch, create |
| interns       |  yono  | instavote | readonly | get, list, watch |



| Service Accounts     |  Namespace     |  Resources     |  Access Type (verbs) |
|  :------------- |  :------------- | :------------- |    :------------- |
| monitoring              | all | all | readonly |





### Creating Kubernetes Users and Groups

Generate the user's private key
```
mkdir -p  ~/.kube/users
cd ~/.kube/users


openssl genrsa -out kim.key 2048

```

[sample Output]

```
openssl genrsa -out kim.key 2048
Generating RSA private key, 2048 bit long modulus
.............................................................+++
.........................+++
e is 65537 (0x10001)

```

Lets now create a **Certification Signing Request (CSR)** for each of the users. When you generate the csr make sure you also provide

  * CN: This will be set as username
  * O: Org name. This is actually used as a **group** by kubernetes while authenticating/authorizing users.  You could add as many as you need


e.g.

```
openssl req -new -key kim.key -out kim.csr -subj "/CN=kim/O=dev/O=example.org"

```
If you get error `Can't load /root/.rnd into RNG` during Creating CSR , then run below command first the generate CSR
```
openssl rand -out /home/ubuntu/.rnd -hex 256
```
In order to be deemed authentic, these CSRs need to be signed by the **Certification Authority (CA)** which in this case is Kubernetes Master.   You need access to the folllwing files on  kubernetes master.

  * Certificate : ca.crt (kubeadm) or ca.key (kubespray)
  * Pricate Key : ca.key (kubeadm) or ca-key.pem  (kubespray)

You would typically find it  the following paths

  * /etc/kubernetes/pki

To verify which one is your cert and which one is key, use the following command,

```
$ file /etc/kubernetes/pki/ca.crt
ca.pem: PEM certificate


$ file /etc/kubernetes/pki/ca.key
ca-key.pem: PEM RSA private key
```


Once signed, .csr files with added signatures become the certificates that could be used to authenticate.

You could either

   * move the crt files to k8s master, sign and download  
   * copy over the CA certs and keys to your management node and use it to sign. Make sure to keep your CA related files secure.


In the example here, I have already downloaded **ca.pem** and **ca-key.pem** to my management workstation, which are used to sign the CSRs.  

Assuming all the files are in the same directory, sign the CSR as,

```
openssl x509 -req -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 730 -in kim.csr -out kim.crt

```


### Setting up User configs with kubectl


In order to configure the users that you created above, following steps need to be performed with kubectl

  * Add credentials in the configurations
  * Set context to login as a user to a cluster
  * Switch context in order to assume the user's identity while working with the cluster


to add credentials,

```
kubectl config set-credentials kim --client-certificate=/root/.kube/users/kim.crt --client-key=/root/.kube/users/kim.key
```

where,

  * /root/.kube/users/kim.crt : Absolute path to the users' certificate
  * /root/.kube/users/kim.key: Absolute path to the users' key



And proceed to set/create  contexts (user@cluster). If you are not sure whats the cluster name, use the following command to find,

```
kubectl config get-contexts

```
[sample output]

```
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   instavote
```

where,  **kubernetes** is the  cluster name.

To set context for **kubernetes** cluster,

```
kubectl config set-context kim-kubernetes --cluster=kubernetes  --user=kim --namespace=instavote

```

Where,

  * kim-kubernetes : name of the context  
  * kubernetes  : name of the  cluster you set while creating it  
  * kim : user you created and configured above to connect to the cluster  


You could verify the configs with


```
kubectl config get-contexts

CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          kim-kubernetes                kubernetes   kim                instavote
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   instavote
```


and


```
kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://178.128.109.8:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: instavote
    user: kim
  name: kim-kubernetes
- context:
    cluster: kubernetes
    namespace: instavote
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kim
  user:
    client-certificate: users/kim.crt
    client-key: users/kim.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

Where, you should see the configurations for the new user you created have been added.


You could assume the identity of user **kim** and connect  to the **kubernetes** cluster as,

```
kubectl config use-context kim-kubernetes

kubectl config get-contexts

CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kim-kubernetes                kubernetes   kim                instavote
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   instavote
```

This time * appears on the line which lists context **kim-kubernetes** that you just created.


And then try running any command as,

```
kubectl get pods
```


Alternately, if you are a admin user, you could impersonate a user and run a command with that literally using  --as option

```
kubectl config use-context admin-prod
kubectl get pods --as yono
```

[Sample Output]
```
No resources found.
Error from server (Forbidden): pods is forbidden: User "yono" cannot list pods in the namespace "instavote"

```



Either ways, since there are authorization rules set, the user can not make any api calls. Thats when you would create some roles and bind it to the users in the next section.


## Define authorisation rules with Roles and ClusterRoles

Whats the difference between Roles and ClusterRoles ??

  * Role is  limited to a namespace (Projects/Orgs/Env)
  * ClusterRole is Global

Lets say you want to provide read only access to **instavote**, a project specific namespace to all users in the **example.org**

```
kubectl config use-context kubernetes-admin@kubernetes
cd projects/instavote/dev
```

`file: readonly-role.yaml`

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  namespace: instavote
  name: readonly
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

In order to map it to all users in **example.org**, create a RoleBinding as


`file: readonly-rolebinding.yml`

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: readonly
  namespace: instavote
subjects:
- kind: Group
  name: dev
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: readonly
  apiGroup: rbac.authorization.k8s.io
```


```
kubectl apply -f readonly-role.yaml
kubectl apply -f readonly-rolebinding.yml
```

To get information about the objects created above,

```
kubectl get roles,rolebindings -n instavote

kubectl describe role readonly
kubectl describe rolebinding readonly

```

To validate the access,
```
kubectl config get-contexts
kubectl config use-context kim-kubernetes
kubectl get pods

```

To switch back to admin,

```
kubectl config use-context kubernetes-admin@kubernetes

```

## Securing Applications and Clusters
In a complex system such as Kubernetes, authorization mechanisms are used to set who is allowed to make what changes to the cluster resources and manipulate them. Role-based access control (RBAC) is a mechanism that's highly integrated into Kubernetes that grants users and applications granular access to Kubernetes APIs.

As good practice, you should use the Node and RBAC authorizers together with the NodeRestriction admission plugin. 

In this section, we will cover getting RBAC enabled and creating Roles and RoleBindings to grant applications and users access to the cluster resources.

Getting ready
Make sure you have an RBAC-enabled Kubernetes cluster ready (since Kubernetes 1.6, RBAC is enabled by default) and that kubectl and helm have been configured so that you can manage the cluster resources. Creating private keys will also require that you have the openssl tool before you attempt to create keys for users.



### How to do it? 

This section is further divided into the following subsections to make this process easier:

- Viewing the default Roles
- Creating user accounts
- Creating Roles and RoleBindings
- Testing the RBAC rules

### Viewing the default Roles

RBAC is a core component of the Kubernetes cluster that allows us to create and grant roles to objects and control access to resources within the cluster. This example will help you understand the content of roles and role bindings.

Let's perform the following steps to view the default roles and role bindings in our cluster:

1. View the default cluster roles using the following command. You will see a long mixed list of system:, system:controller:, and a few other prefixed roles. system:* roles are used by the infrastructure, system:controller  roles are used by a Kubernetes controller manager, which is a control loop that watches the shared state of the cluster. In general, they are both good to know about when you need to troubleshoot permission issues, but they're not something we will be using very often:

```
kubectl get clusterroles

kubectl get clusterrolebindings
```

2. View one of the system roles owned by Kubernetes to understand their purpose and limits. In the following example, we're looking at system:node, which defines the permission for kubelets. In the output in Rules, apiGroups: indicates the core API group, resources indicates the Kubernetes resource type, and verbs indicates the API actions allowed on the role:

```
kubectl get clusterroles system:node -oyaml
```

3. Let's view the default user-facing roles since they are the ones we are more interested in. The roles that don't have the system: prefix are intended to be user-facing roles. The following command will only list the non-system: prefix roles. The main roles that are intended to be granted within a specific namespace using RoleBindings are the admin, edit, and view roles:

```
kubectl get clusterroles | grep -v '^system'
NAME                 AGE
admin                8d      #gives read-write access to all resources
cluster-admin        8d      #super-user, gives read-write accessto all resources
edit                 8d      #allows create/update/delete on resources except RBAC permissions
kube-dns-autoscaler  8d
view                 8d      #read-only access to resources
```

4. Now, review the default cluster binding, that is, cluster-admin, using the following command. You will see that this binding gives the system:masters group cluster-wide superuser permissions with the cluster-admin role:
```yaml

kubectl get clusterrolebindings/cluster-admin -o yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
...
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

### Creating user accounts
- In the following recipes, you will learn how to create new Roles and RoleBindings and grant accounts the permissions that they need.

As explained in the Kubernetes docs, Kubernetes doesn't have objects to represent normal user accounts. Therefore, they need to be managed externally (check the Kubernetes Authentication documentation in the See also section for more details). This recipe will show you how to create and manage user accounts using private keys.

- Let's perform the following steps to create a user account:

1. Create a private key for the example user. In our example, the key file is user3445.key:

```yaml
openssl genrsa -out user3445.key 2048
```

2. Create a certificate sign request (CSR) called user3445.csr using the private key we created in Step 1. Set the username (/CN) and group name (/O) in the -subj parameter. In the following example, the username is john.geek, while the group is development:

```yaml
openssl req -new -key user3445.key -out user3445.csr -subj "/CN=john.geek/O=development"
```

3. To use the built-in signer, you need to locate the cluster-signing certificates for your cluster. By default, the `ca.crt` and `ca.key` files should be in the `/etc/kubernetes/pki/` directory.Once you've located the keys, change the CERT_LOCATION mentioned in the following code to the current location of the files and generate the final signed certificate:

```yaml
$ openssl x509 -req -in user3445.csr \
-CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial -out user3445.crt \
-days 500
```

4. If all the files have been located, the command in Step 3 should return an output similar to the following:
```yaml
Signature ok
subject=CN = john.geek, O = development
Getting CA Private Key
```

5. Create a new context using the new user credentials:
```yaml
kubectl config set-credentials user3445 --client-certificate=user3445.crt --client-key=user3445.key


kubectl config set-context user3445-context --cluster=local --namespace=secureapp --user=user3445
```
6. List the existing context using the following comment. You will see that the new user3445-context has been created:
```yaml
kubectl config get-contexts
CURRENT NAME                    CLUSTER AUTHINFO NAMESPACE
*   Service-account-context     local   kubecfg
    user3445-context            local   user3445 secureapp
```

7. Now, try to list the pods using the new user context. You will get an access denied error since the new user doesn't have any roles and new users don't come with any roles assigned to them by default:
```yaml
kubectl --context=user3445-context get pods
Error from server (Forbidden): pods is forbidden: User "john.geek" cannot list resource "pods" in API group "" in the namespace "secureapps"
```
8. Optionally, you can base64 encode all three files (user3445.crt, user3445.csr, and user3445.key) using the openssl base64 -in <infile> -out <outfile> command and distribute the populated config-user3445.yml file to your developers. An example file can be found in this book's GitHub repository in the src/chapter9/rbac directory. There are many ways to distribute user credentials. Review the example using your text editor:

```yaml
$ cat config-user3445.yaml
```
With that, you've learned how to create new users. Next, you will create roles and assign them to the user.

### Creating Roles and RoleBindings

Roles and RolesBindings are always used in a defined namespace, meaning that the permissions can only be granted for the resources that are in the same namespace as the Roles and the RoleBindings themselves compared to the ClusterRoles and ClusterRoleBindings that are used to grant permissions to cluster-wide resources such as nodes. 

- Let's perform the following steps to create an example Role and RoleBinding in our cluster:

1. First, create a namespace where we will create the Role and RoleBinding. In our example, the namespace is secureapp:

```yaml
kubectl create ns secureapp
```

2. Create a role using the following rules. This role basically allows all operations to be performed on deployments, replica sets, and pods for the deployer role in the secureapp namespace we created in Step 1. Note that any permissions that are granted are only additive and there are no deny rules:
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: secureapp
  name: deployer
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

3. Create a RoleBinding using the deployer role and for the username john.geek in the secureapp namespace. We're doing this since a RoleBinding can only reference a Role that exists in the same namespace:

```yaml
cat <<EOF | kubectl apply -f -
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: deployer-binding
  namespace: secureapp
subjects:
- kind: User
  name: john.geek
  apiGroup: ""
roleRef:
  kind: Role
  name: deployer
  apiGroup: ""
EOF
```
With that, you've learned how to create a new Role and grant permissions to a user using RoleBindings.

###  Testing the RBAC rules

- Let's perform the following steps to test the Role and RoleBinding we created earlier:

1. Deploy a test pod in the secureapp namespace where the user has access:
```yaml
cat <<EOF | kubectl --context=user3445-context apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: secureapp
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
EOF
```

2. List the pods in the new user's context. The same command that failed in the Creating user accounts recipe in Step 7 should now execute successfully:

```yaml
kubectl --context=user3445-context get pods 
NAME     READY  STATUS   RESTARTS AGE
busybox  1/1    Running  1        2m
```

If you try to create the same pod in a different namespace, you will see that the command will fail to execute.

### How it works...

- This recipe showed you how to create new users in Kubernetes and quickly create Roles and RoleBindings to grant permission to user accounts on Kubernetes.

- Kubernetes clusters have two types of users:

  - User accounts: User accounts are normal users that are managed externally.

  -  Service accounts: Service accounts are the users who are associated with the Kubernetes services and are managed by the Kubernetes API with its own resources.


- In the Creating Roles and RoleBindings recipe, in Step 1, we created a Role named deployer.

- Then, in Step 2, we granted the rules associated with the deployer Role to the user account john.geek.

- RBAC uses the rbac.authorization.k8s.io API to make authorization decisions. This allows admins to dynamically configure policies using the Kubernetes APIs. If you wanted to use the existing Roles and give someone cluster-wide superuser permission, you could use the cluster-admin ClusterRole with a ClusterRoleBinding instead. ClusterRoles don't have namespace limits and can execute commands in any namespace with the granted permissions. Overall, you should be careful while assigning the cluster-admin ClusterRole to users. ClusterRoles can be also limited to namespaces, similar to Roles if they are used with RoleBindings to grant permissions instead. 

### Refrences 

 [RBAC Authorization in Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)

 [Managing Service Accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)