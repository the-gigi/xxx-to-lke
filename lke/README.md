# Install linode-cli

This is already done in the requirements.txt of the virtual environment

# Provision a minimal LKE cluster

Check available versions
```
❯ lin lke versions-list
┌──────┐
│ id   │
├──────┤
│ 1.31 │
├──────┤
│ 1.30 │
└──────┘
```

Check available regions in the US

```
❯ lin regions ls | rg 'us-|label' | awk -F '│' '{print $2, $3}' | sed 's/^ *//;s/ *$//'
id             label
us-iad         Washington, DC
us-ord         Chicago, IL
us-sea         Seattle, WA
us-mia         Miami, FL
us-lax         Los Angeles, CA
us-central     Dallas, TX
us-west        Fremont, CA
us-southeast   Atlanta, GA
us-east        Newark, NJ
```

Check available node types

Let's search for node types with 2 CPUs and 4 GiB of memory
```
❯ lin linodes types --vcpus 2 --memory 4096 --json --pretty | jq '.[] | {class, id, vcpus, memory}'
{
  "class": "standard",
  "id": "g6-standard-2",
  "vcpus": 2,
  "memory": 4096
}
{
  "class": "dedicated",
  "id": "g6-dedicated-2",
  "vcpus": 2,
  "memory": 4096
}
{
  "class": "premium",
  "id": "g7-premium-2",
  "vcpus": 2,
  "memory": 4096
}
```

Let's provision a cluster with the following configuration:

```
linode lke cluster-create \
  --label eks-to-lke \
  --k8s_version 1.31 \
  --region us-sea \
  --node_pools '[{
    "type": "g6-standard-2",
    "count": 1,
    "autoscaler": {
      "enabled": true,
      "min": 1,
      "max": 3
    }
  }]'
  
Using default values: {}; use the --no-defaults flag to disable defaults
┌────────┬────────────┬────────┬─────────────┬─────────────────────────────────┬──────┐
│ id     │ label      │ region │ k8s_version │ control_plane.high_availability │ tier │
├────────┼────────────┼────────┼─────────────┼─────────────────────────────────┼──────┤
│ 278475 │ eks-to-lke │ us-sea │ 1.31        │ False                           │      │
└────────┴────────────┴────────┴─────────────┴─────────────────────────────────┴──────┘

```

# Access the cluster

Let's get the kube config for the cluster and store it in ~/.kube/lke-config

```
CLUSTER_ID=$(linode lke clusters-list --json | \
  jq -r '.[] | select(.label == "eks-to-lke") | .id')

linode lke kubeconfig-view --json "$CLUSTER_ID" | jq -r \
      '.[0].kubeconfig' | base64 --decode > ~/.kube/lke-config
```

Now, we can access the cluster using the kubeconfig file.

```
❯ kubectl get no --kubeconfig ~/.kube/lke-config
NAME                            STATUS   ROLES    AGE    VERSION
lke278475-459426-19cb173e0000   Ready    <none>   9m4s   v1.31.0
```

We have one node in the cluster as requested, and it is using version 1.31.0.

# Deploy the go-quote service

```
❯ kubectl apply -f ../k8s/manifests.yaml --kubeconfig ~/.kube/lke-config 
deployment.apps/go-quote created
service/go-quote-service created
horizontalpodautoscaler.autoscaling/go-quote-hpa created
```

Let's verify the deployment is ready and healthy:
```
❯ kubectl get deploy go-quote --kubeconfig ~/.kube/lke-config
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
go-quote   1/1     1            1           3m44s
```

Let's check the service:
```
❯ kubectl get svc
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
go-quote-service   LoadBalancer   34.118.230.115   34.106.216.147   80:30378/TCP   3h20m
kubernetes         ClusterIP      34.118.224.1     <none>           443/TCP        3h53m
```

Great. We have an external IP. Let's check if we can access the service:
```
❯ http -b 34.106.216.147/quotes quote='test quote 1'
❯ http -b 34.106.216.147/quotes quote='test quote 2'
❯ http -b 34.106.216.147/quotes                     
[
    "test quote 1",
    "test quote 2"
]
```
