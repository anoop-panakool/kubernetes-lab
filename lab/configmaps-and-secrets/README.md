# Configuration
Keeping your web application's configuration data separate from your source code is a sensible best practice for running production web services. It enables you to change configuration settings without redeployment, and also prevents leaking secret keys to the wrong environments or into source control.
Kubernetes provides two mechanisms for this—ConfigMaps and Secrets. ConfigMaps and Secrets
are virtually identical, except that Secrets are naturally more suited to storing sensitive data like API keys.

## ConfigMaps

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

- A ConfigMap is a key-value dictionary whose data is injected into your container's environment when it runs.
- The first step to using a ConfigMapis to create the ConfigMap resource inside your Kubernetes cluster.
- The next step is for a pod to consume it by mounting it as a volume or getting
its data injected through environment variables.

## Creating a ConfigMap
ConfigMaps are very flexible in how they are created and consumed. You can,

 - create a ConfigMap directly in YAML — this is our preferred solution
 - create a ConfigMap from a directory of files
 - create a ConfigMap from a single file
 - create a ConfigMap from literal values on the command line

1.  Create a ConfigMap [configmap.yaml](/lab/configmaps-and-secrets/configmap.yaml) like any other Kubernetes resource

 ```yaml
 kind: ConfigMap
apiVersion: v1
metadata:
  name: example-configmap
data:
  # Configuration values can be set as key-value properties
  database: mongodb
  database_uri: mongodb://localhost:27017
# Or set as complete file contents (even JSON!)
  keys: |
     image.public.key=771
     rsa.public.key=42
 ```

2. Apply the ConfigMap
```yaml
kubectl apply -f configmap.yaml
```

- Using a ConfigMap through a volume

> Once created, add the ConfigMap as a volume in your pod's specification. Then, mount that volume into the container's filesystem. Each property name in the ConfigMap will become a new file in the mounted directory, and the contents of each file will be the value specified in the ConfigMap.

3. Create the pod [pod-using-configmap.yaml](/lab/configmaps-and-secrets/pod-using-configmap.yaml), start an interactive shell inside it, and inspect the contents of the ConfigMap that was mounted as a volume. Each entry in the ConfigMap was mounted as a separate file in the /etc/config directory, with file contents corresponding to what we specified in the YAML.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pod-using-configmap
spec:
  # Add the ConfigMap as a volume to the Pod
  volumes:
  # `name` here must match the name
  # specified in the volume mount
    - name: example-configmap-volume
      # Populate the volume with config map data
      configMap:
      # `name` here must match the name
      # specified in the ConfigMap's YAML
        name: example-configmap
  containers:
    - name: container-configmap
      image: nginx:1.7.9
      # Mount the volume that contains the configuration data
      # into your container filesystem
      volumeMounts:
         # `name` here must match the name
         # from the volumes section of this pod
        - name: example-configmap-volume
          mountPath: /etc/config
```

```yaml
kubectl apply -f pod-using-configmap.yaml
```

```yaml
kubectl exec -it pod-using-configmap sh

# ls /etc/config

database
database_uri
keys

# cat /etc/config/*

mongodbmongodb://localhost:27017image.public.key=771
```
4. The Output is as follow

```yaml
mongodb                     <- contents of `database`
http://localhost:27017      <- contents of `database_uri`
image.public.key=771        <- contents of `keys`
rsa.public.key=42            -
```

- Using a ConfigMap through environment variables

Another useful way of consuming a ConfigMap you have created in your Kubernetes cluster is by injecting its data as environment variables into your container.

5. Create a POD [pod-env-var.yaml](/lab/configmaps-and-secrets/pod-env-var.yaml) and inject environment variables using ConfigMap
We can access and view the injected environment variables once the pod is running.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pod-env-var
spec:
  containers:
  - name: env-var-configmap
    image: nginx:1.7.9
    envFrom:
    - configMapRef:
        name: example-configmap
```
```yaml
kubectl apply -f pod-env-var.yaml

kubectl exec -it pod-env-var sh
# env
```

6. The Output is as follow

```yaml
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
database=mongodb
HOSTNAME=pod-env-var
database_uri=mongodb://localhost:27017
HOME=/root
keys=image.public.key=771
rsa.public.key=42
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_VERSION=1.7.9-1~wheezy
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```

# Secrets

If you understand creating and consuming ConfigMaps, you also understand how to use Secrets. The primary difference between the two is that Secrets are designed to hold sensitive data—things like keys, tokens, and passwords.

Kubernetes will avoid writing Secrets to disk—instead using tmpfs volumes on each node, so they can't be left behind on a node. However, etcd, Kubernetes’ configuration key-value store, stores secrets in plaintext

## Creating a Secret

As we did with a ConfigMap, let's create a YAML file to configure secrets for an API access key and token.

When adding Secrets via YAML, they must be encoded in base64. base64 is not an encryption method and does not provide any security for what is encoded—it's simply a way of
presenting a string in a different format. Do not commit your base64- encoded secrets to source control or share them publicly.

Encode strings in base64 using the base64 command in your shell, or with an online tool like base64encode.org

1. Let's base64 encode our access key and API token.

```yaml
echo "OUR_API_ACCESS_KEY" | base64
```
The Output as follow

```yaml
T1VSX0FQSV9BQ0NFU1NfS0VZCg==
```
```yaml
echo "SECRET_7t4836378erwdser34" | base64
```
The Output as follow
```yaml
U0VDUkVUXzd0NDgzNjM3OGVyd2RzZXIzNAo=
```

2. Now, add these encoded strings to the YAML file [api-authentication-secret.yaml](/lab/configmaps-and-secrets/api-authentication-secret.yaml) for our Secret.

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: api-authentication-secret
type: Opaque
data:
  key: T1VSX0FQSV9BQ0NFU1NfS0VZCg==
  token: U0VDUkVUXzd0NDgzNjM3OGVyd2RzZXIzNAo=
```
3. Create the Secret
```yaml
kubectl apply -f api-authentication-secret.yaml
```
4. Useg this Secret in the POD file [pod-using-secret.yaml](/lab/configmaps-and-secrets/pod-using-secret.yaml)

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pod-using-secret
spec:
  # Add the Secret as a volume to the Pod
  volumes:
    # `name` here must match the name
    # specified in the volume mount
    - name: api-secret-volume
      # Populate the volume with config map data
      secret:
        # `secretName` here must match the name
        # specified in the secret's YAML
        secretName: api-authentication-secret
  containers:
  - name: container-secret
    image: nginx:1.7.9
    volumeMounts:
      # `name` here must match the name
      # from the volumes section of this pod
    - name: api-secret-volume
      mountPath: /etc/secret
```
4. Create the POD after injecting secret insite it,Secret will be mounted inside the`/etc/secret` directory in our container, and we can access those files from our container.

```yaml
kubectl apply -f pod-using-secret.yaml

kubectl exec -it pod-using-secret sh

# cat /etc/secret/key
OUR_API_ACCESS_KEY

# cat /etc/secret/token
SECRET_7t4836378erwdser34
```