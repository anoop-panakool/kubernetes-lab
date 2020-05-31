## Managing Deployment, Scaling and Rolling updates

1. Create a deployment [kubeserve-deployment.yaml](/lab/deployment/kubeserve-deployment.yaml)  with following contents:

- Create a deployment with a record (for rollbacks):
```
kubectl create -f kubeserve-deployment.yaml --record
```

- Check the status of the rollout:
```
kubectl rollout status deployments kubeserve
```

- View the ReplicaSets in your cluster:
```
kubectl get replicasets
```

- Scale up your deployment by adding more replicas:
```
kubectl scale deployment kubeserve --replicas=5
```

- Expose the deployment and provide it a service:
```
kubectl expose deployment kubeserve --port 80 --target-port 80 --type NodePort
```

- Set the minReadySeconds attribute to your deployment:
```
kubectl patch deployment kubeserve -p '{"spec": {"minReadySeconds": 10}}'
```

- Use kubectl apply to update a deployment:
```
kubectl apply -f kubeserve-deployment.yaml
```

- Use kubectl replace to replace an existing deployment:
```
kubectl replace -f kubeserve-deployment.yaml
```

- Run this curl look while the update happens:
```
while true; do curl http://10.105.31.119; done
```

- Perform the rolling update:
```
kubectl set image deployments/kubeserve app=linuxacademycontent/kubeserve:v2 --v 6
```

- Describe a certain ReplicaSet:
```
kubectl describe replicasets kubeserve-[hash]
```

- Apply the rolling update to version 3 (buggy):
```
kubectl set image deployment kubeserve app=linuxacademycontent/kubeserve:v3
```

- Undo the rollout and roll back to the previous version:
```
kubectl rollout undo deployments kubeserve
```

- Look at the rollout history:
```
kubectl rollout history deployment kubeserve
```

- Roll back to a certain revision:
```
kubectl rollout undo deployment kubeserve --to-revision=2
```

- Pause the rollout in the middle of a rolling update (canary release):
```
kubectl rollout pause deployment kubeserve
```

- Resume the rollout after the rolling update looks good:
```
kubectl rollout resume deployment kubeserve
```

2.  Create a deployment `deploy-1.yaml` with following contents:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: deploy1
    spec:
        replicas: 3
        revisionHistoryLimit: 10
        strategy:
            rollingUpdate:
            maxSurge: 1
            maxUnavailable: 1
        minReadySeconds: 3
        selector:
            matchLabels:
                app: web1
        template:
            metadata:
                labels:
                    app: web1
            spec:
                containers:
                -   name: web
                    image: nginx:1.7.9
                    ports:
                -   containerPort: 80
    ```

- Now, run following commands to deploy.

    ```bash
    # Deploy
    $ kubectl apply -f deploy-1.yaml
    # Verify the deployment
    $ kubectl get deploy deploy1
    $ kubectl get rs -l app=web1
    $ kubectl get pods -l app=web1
    ```

- Try scaling deployment through command line

    ```bash
    $ kubectl scale deploy deploy1 --replicas=5  
    $ kubectl get deploy
    $ kubectl get rs -l app=web1
    ```

- Now, lets try rolling update. Try changing yaml file and update image version from 1.7.9 to 1.11.0 using CLI.

    ```bash
    # Update container image for 'web' container for deployment 'deploy1'
    $ kubectl set  image deploy/deploy1 web=nginx:1.11.0
    # View the rollout status, pod status and replica-sets
    $ kubectl rollout  status deploy/deploy1  
    $ kubectl get pods -l app=web1
    $ kubectl get rs -l app=web1
    ```

- Undo the last change (revert to last version) using following command:

    ```bash
    $ kubectl rollout undo deploy/deploy1    
    $ kubectl rollout status deploy/deploy1
    # View all replica sets along with image tags
    $ kubectl get rs -l app=web1 -o wide
    ```

- View the rollout history

    ```bash
    $ kubectl rollout history
    ```

-Performing rolling update using declarative option (editing deploy-1.yaml)
    Open `deploy-1.yaml` and replace `nginx:1.7.9` to `nginx:1.13`
    Use following command to apply changes and view rollout history.

    ```bash
    $ kubectl apply -f deploy-1.yaml --record
    $ kubectl rollout history
    $ kubectl get rs -o wide -l app=web1
    ```
    
> the extra option `--record` records the current command and use it as `change-cause` for rollout history.