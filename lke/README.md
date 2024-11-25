# Install linode-cli

This is already done in the reqquirements.txt of the virutal environment

# Provision a minimal LKE cluster

Check available versions
```
linode lke versions-list
┌──────┐
│ id   │
├──────┤
│ 1.31 │
├──────┤
│ 1.30 │
├──────┤
│ 1.29 │
└──────┘
```

Check availabe node types
```
❯ linode linodes types --label "Linode 4GB" --json --pretty
[
  {
    "addons": {
      "backups": {
        "price": {
          "hourly": 0.008,
          "monthly": 5.0
        },
        "region_prices": [
          {
            "hourly": 0.009,
            "id": "id-cgk",
            "monthly": 6.0
          },
          {
            "hourly": 0.01,
            "id": "br-gru",
            "monthly": 7.0
          }
        ]
      }
    },
    "class": "standard",
    "disk": 81920,
    "gpus": 0,
    "id": "g6-standard-2",
    "label": "Linode 4GB",
    "memory": 4096,
    "network_out": 4000,
    "price": {
      "hourly": 0.036,
      "monthly": 24.0
    },
    "region_prices": [
      {
        "hourly": 0.043,
        "id": "id-cgk",
        "monthly": 28.8
      },
      {
        "hourly": 0.05,
        "id": "br-gru",
        "monthly": 33.6
      }
    ],
    "successor": null,
    "transfer": 4000,
    "vcpus": 2
  }
]
```

Let's provision a cluster with the following configuration:

```
linode lke cluster-create \
  --label eks-to-lke \
  --k8s_version 1.31 \
  --region ca-central \
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
┌────────┬────────────┬────────────┬─────────────┬─────────────────────────────────┬──────┐
│ id     │ label      │ region     │ k8s_version │ control_plane.high_availability │ tier │
├────────┼────────────┼────────────┼─────────────┼─────────────────────────────────┼──────┤
│ 274239 │ eks-to-lke │ ca-central │ 1.31        │ False                           │      │
└────────┴────────────┴────────────┴─────────────┴─────────────────────────────────┴──────┘ 
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
NAME                            STATUS   ROLES    AGE     VERSION
lke274239-446323-3442419d0000   Ready    <none>   2m34s   v1.31.0
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
❯ kubectl get svc --kubeconfig ~/.kube/lke-config
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
go-quote-service   LoadBalancer   10.128.173.62   172.105.1.151   80:30204/TCP   4m4s
kubernetes         ClusterIP      10.128.0.1      <none>          443/TCP        9m32s
```

Great. We have an external IP. Let's check if we can access the service:
```
❯ http -b http://172.105.1.151/quotes quote='test quote 1'
❯ http -b http://172.105.1.151/quotes quote='test quote 2'
❯ http -b http://172.105.1.151/quotes
[
    "test quote 1",
    "test quote 2"
]
```
