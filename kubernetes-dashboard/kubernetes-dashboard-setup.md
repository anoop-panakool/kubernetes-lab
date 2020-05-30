
For reference -
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

cd dashboard/

Launch the required  deployment, service sa, etc using -

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml

    kubectl apply -f https://raw.githubusercontent.com/shivamjhalabfiles/kubernetes-lab/master/kubernetes-dashboard/dashboard-sa.yml?token=APURC7TS5XOTWEXG76YN5DK62LEMS

    kubectl apply -f https://raw.githubusercontent.com/shivamjhalabfiles/kubernetes-lab/master/kubernetes-dashboard/dashboard-role-binding.yaml?token=APURC7WZNVZWNMXDS5B3MB262LEK2

    kubectl get services -n kubernetes-dashboard

Edit the kubernetes-dashboard service type from ClusterIp to NodePort

            kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
            <Change type: ClusterIp to type: NodePort>
            save the file and exit
Observe the Node Port assigned to kubernetes-dashboard service

Open Firefox browser (It does not work in chrome due to security check )

     https://<public-ip-of-master-node>:<nodeport>
      eg.
     https://63.32.55.212:30085/#/login

 - Proceed by accepting the warning

 - It will ask for a token

 - Generate the token using below command on Master:


    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

Copy the token from the output.
Paste it on the browser window under token field


![k8s-dasboard.png](https://github.com/shivamjhalabfiles/kubernetes-lab/blob/master/images/k8s-dasboard.png)


