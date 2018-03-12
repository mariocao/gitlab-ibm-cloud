# Deploy Gitlab to IBM Cloud

* Scripts under the folder "kubernetes" were used in order to use NFS and avoid persistence related problems within the cluster in the IBM Cloud.
* In order to better track the Gitlab deployment, the yaml file have been organized as follows:
  * <service>-volume
  * <service>-claim
  * <service>

These yaml files have been applied by using the command:

```
kubectl apply -f <file.yaml>
```

## 1. Prerequisites

Have the IBM Cloud client (bx) and Kubernetes (kubectl) CLIs installed and working.

Have the NFS storage already deployed and configured.

## 2. Login to Bluemix & Configure environment

Login to Bluemix

```
bx login
```

Make sure you have the right API endpoint selected (as of the time of writing, we are only able to deploy on the cluster in the 'eu-gb' region)

Target the right organization and space

```
bx target --cf
```

## 3. (Only first time) Install the IBM Bluemix Container Service plug-in

```
bx plugin install container-service -r Bluemix
```

## 4. (Only first time) Check for existing clusters or create one

Test if you have any clusters set-up already

```
$ bx cs clusters
OK
Name        ID                                 State    Created       Workers   Datacenter   Version
mycluster   0c7cc955f9bf47caa335327a3fcd5421   normal   1 month ago   1         par01        1.7.4_1503  
```

(If not, use 'bx cs cluster-create' to create a new one or do it directly from the web-based UI).

## 5. Download and set the configuration for your cluster

```
bx cs cluster-config mycluster

OK
The configuration for mycluster was downloaded successfully. Export environment variables to start using Kubernetes.

export KUBECONFIG=/Users/<username>/.bluemix/plugins/container-service/clusters/mycluster/kube-config-lon02-mycluster.yml
```

Then, copy and paste the export statement to the command-line, setting your kubeconfig to the cluster of your choice.

If everything went well, and you have the kubectl CLI installed, you can use kubectl proxy to test your cluster, and have a look at it on http://127.0.0.1:8001/api/v1

```
kubectl proxy
```

## 6. Run the scripts in the repo

Execute the scripts in the following order:

```
$ kubectl create -f kubernetes/postgres
$ kubectl create -f kubernetes/redis
$ kubectl create -f kubernetes/gitlab
$ kubectl create -f kubernetes/ingress.yaml
```

After this, the GitLab container should be ready after 3 to 5 minutes.

## 7. Access GitLab

The Gitlab will be now accessible at mycluster.uk-south.containers.mybluemix.net. Happy coding!

## (OLD only for free/lite cluster) Retrieve external ip and port for GitLab

After few minutes run the following commands to get your public IP and NodePort number.

```
$ $ bx cs workers mycluster

OK
ID                                                 Public IP       Private IP     Machine Type   State    Status
kube-hou02-pa817264f1244245d38c4de72fffd527ca-w1   169.47.241.22   10.10.10.148   free           normal   Ready

$ kubectl get svc gitlab

NAME      CLUSTER-IP     EXTERNAL-IP   PORT(S)                     AGE
gitlab    10.10.10.148   <nodes>       80:30080/TCP,22:30022/TCP   2s
```

Congratulations! You can now find your newly deployed gitlab on http://<node_public_ip>:30080

Note: The administrator username is 'root'.
