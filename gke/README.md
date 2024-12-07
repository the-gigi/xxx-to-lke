# Create an a Google account and go to the Google cloud console

https://console.cloud.google.com/

# Install and configure gcloud

Follow the instructions here:
https://cloud.google.com/sdk/docs/install

```bash
gcloud --version
Google Cloud SDK 502.0.0
bq 2.1.9
core 2024.11.15
gcloud-crc32c 1.0.0
gsutil 5.31
```

# Login and check your GCP project

```
gcloud auth login
```

```
gcloud config get project

playground-161404
```

# Provision an Auto-pilot GKE cluster with gcloud

## Enable the GKE API if needed

```
gcloud services  list --enabled | rg container
container.googleapis.com            Kubernetes Engine API
containerfilesystem.googleapis.com  Container File System API
containerregistry.googleapis.com    Container Registry API
```
You can enable API services like so: 
```
gcloud services enable container.googleapis.com
```

## Choose a region

See:
https://cloud.google.com/compute/docs/regions-zones
https://cloud.google.com/compute/docs/regions-zones/viewing-regions-zones
https://cloud.google.com/about/locations

Let's see what regions are available in the US west:

```
❯ gcloud compute regions list | rg us-west
us-west1                 0/24  0/10240   0/23       0/8                 UP
us-west2                 0/24  0/10240   0/23       0/8                 UP
us-west3                 0/24  0/10240   0/23       0/8                 UP
us-west4                 0/24  0/10240   0/23       0/8                 UP
```

Let's pick us-west3 (Salt Lake City). 

## Create the cluster

```
gcloud container clusters create-auto test-cluster --region us-west3
Note: The Kubelet readonly port (10255) is now deprecated. Please update your workloads to use the recommended alternatives. See https://cloud.google.com/kubernetes-engine/docs/how-to/disable-kubelet-readonly-port for ways to check usage and for migration instructions.
Creating cluster test-cluster in us-west3... Cluster is being health-checked (Kubernetes Control Plane is healthy)...done.
Created [https://container.googleapis.com/v1/projects/playground-161404/zones/us-west3/clusters/test-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west3/test-cluster?project=playground-161404
CRITICAL: ACTION REQUIRED: gke-gcloud-auth-plugin, which is needed for continued use of kubectl, was not found or is not executable. Install gke-gcloud-auth-plugin for use with kubectl by following https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl#install_plugin
kubeconfig entry generated for test-cluster.
NAME          LOCATION  MASTER_VERSION      MASTER_IP       MACHINE_TYPE  NODE_VERSION        NUM_NODES  STATUS
test-cluster  us-west3  1.30.5-gke.1443001  34.106.204.240  e2-small      1.30.5-gke.1443001  3          RUNNING
```

Let's follow the instructions and install the gke-gcloud-auth-plugin:

```
gcloud components install gke-gcloud-auth-plugin
```


Create kubeconfig entry for the cluster 

```
gcloud container clusters get-credentials test-cluster --region us-west3
```

```
kubectl config current-context 

gke_playground-161404_us-west3_test-cluster
```

This is Autopilot cluster, so nodes are created only when needed. Initially there are no nodes:

```
kubectl get no

No resources found
```

Not bad. about 2 cores and almost 7 GiB of memory.

# Deploy the go-quote-service

```
kubectl apply -f ../k8s/manifests.yaml
deployment.apps/go-quote created
service/go-quote-service created
horizontalpodautoscaler.autoscaling/go-quote-hpa created
```
 This will trigger scale up, which will provision a node. The pods will be pending until that 
 happens:

```
❯ kubectl get po
NAME                        READY   STATUS    RESTARTS   AGE
go-quote-7b747d5f8f-kvjmf   0/1     Pending   0          13s
```

Eventually, a node will be provisioned let's check it out

```
kubectl get no
NAME                                    STATUS   ROLES    AGE     VERSION
gk3-test-cluster-pool-2-674e1f90-rczt   Ready    <none>   2m54s   v1.30.5-gke.1443001
```

OK. The node is ready. Let's check the available CPU and memory:

```
❯ kubectl get node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') -o yaml | yq '.status.allocatable | {"cpu": .cpu, "memory": .memory}' | awk -F': ' '/cpu/ {cpu=$2} /memory/ {mem=$2} END {printf "cpu: %s\nmemory: %.2f Gi\n", cpu, mem / 1024 / 1024}'

cpu: 1930m
memory: 5.82 Gi
```

Let's verify everything looks fine. Here is the deployment.

```
❯ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
go-quote   1/1     1            1           3h20m

```

Here is the service.
```
❯ kubectl get svc
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
go-quote-service   LoadBalancer   34.118.230.115   34.106.216.147   80:30378/TCP   3h20m
kubernetes         ClusterIP      34.118.224.1     <none>           443/TCP        3h53m
```

Let's confirm we can access the go-quote srvice using the external IP.

```
❯ http -b http://a95fe017ca2c04783af8fb15c8145420-434346127.us-west-2.elb.amazonaws.com/quotes
[
    "test quote 1",
    "test quote 2"
]
```

# Shutdown the cluster

```
❯ eksctl  get cluster
NAME					REGION		EKSCTL CREATED
extravagant-sculpture-1732490260	us-west-2	True
```

```
```