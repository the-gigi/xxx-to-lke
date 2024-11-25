# Install and configure aws-cli

Follow the instructions here:
https://docs.aws.amazon.com/cli/latest/userguide

```bash
aws --version
aws-cli/2.18.16 Python/3.12.7 Darwin/24.0.0 source/arm64
```


# Install and configure eksctl

Follow the instructions here:
https://eksctl.io/installation/

```bash
eksctl version
0.194.0
```

# Provision EKS cluster with eksctl

```
eksctl create cluster
2024-11-24 15:17:40 [ℹ]  eksctl version 0.194.0
2024-11-24 15:17:40 [ℹ]  using region us-west-2
2024-11-24 15:17:40 [ℹ]  setting availability zones to [us-west-2b us-west-2a us-west-2d]
2024-11-24 15:17:40 [ℹ]  subnets for us-west-2b - public:192.168.0.0/19 private:192.168.96.0/19
2024-11-24 15:17:40 [ℹ]  subnets for us-west-2a - public:192.168.32.0/19 private:192.168.128.0/19
2024-11-24 15:17:40 [ℹ]  subnets for us-west-2d - public:192.168.64.0/19 private:192.168.160.0/19
2024-11-24 15:17:40 [ℹ]  nodegroup "ng-69c8c0cb" will use "" [AmazonLinux2/1.30]
2024-11-24 15:17:40 [ℹ]  using Kubernetes version 1.30
2024-11-24 15:17:40 [ℹ]  creating EKS cluster "extravagant-sculpture-1732490260" in "us-west-2" region with managed nodes
2024-11-24 15:17:41 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
2024-11-24 15:17:41 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-west-2 --cluster=extravagant-sculpture-1732490260'
2024-11-24 15:17:41 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "extravagant-sculpture-1732490260" in "us-west-2"
2024-11-24 15:17:41 [ℹ]  CloudWatch logging will not be enabled for cluster "extravagant-sculpture-1732490260" in "us-west-2"
2024-11-24 15:17:41 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-west-2 --cluster=extravagant-sculpture-1732490260'
2024-11-24 15:17:41 [ℹ]  default addons coredns, vpc-cni, kube-proxy were not specified, will install them as EKS addons
2024-11-24 15:17:41 [ℹ]
2 sequential tasks: { create cluster control plane "extravagant-sculpture-1732490260",
    2 sequential sub-tasks: {
        2 sequential sub-tasks: {
            1 task: { create addons },
            wait for control plane to become ready,
        },
        create managed nodegroup "ng-69c8c0cb",
    }
}
2024-11-24 15:17:41 [ℹ]  building cluster stack "eksctl-extravagant-sculpture-1732490260-cluster"
2024-11-24 15:17:41 [ℹ]  deploying stack "eksctl-extravagant-sculpture-1732490260-cluster"
2024-11-24 15:25:44 [ℹ]  waiting for CloudFormation stack "eksctl-extravagant-sculpture-1732490260-cluster"
2024-11-24 15:25:45 [ℹ]  creating addon
2024-11-24 15:25:46 [ℹ]  successfully created addon
2024-11-24 15:25:46 [!]  recommended policies were found for "vpc-cni" addon, but since OIDC is disabled on the cluster, eksctl cannot configure the requested permissions; the recommended way to provide IAM permissions for "vpc-cni" addon is via pod identity associations; after addon creation is completed, add all recommended policies to the config file, under `addon.PodIdentityAssociations`, and run `eksctl update addon`
2024-11-24 15:25:46 [ℹ]  creating addon
2024-11-24 15:25:47 [ℹ]  successfully created addon
2024-11-24 15:25:47 [ℹ]  creating addon
2024-11-24 15:25:47 [ℹ]  successfully created addon
2024-11-24 15:27:49 [ℹ]  building managed nodegroup stack "eksctl-extravagant-sculpture-1732490260-nodegroup-ng-69c8c0cb"
2024-11-24 15:27:49 [ℹ]  deploying stack "eksctl-extravagant-sculpture-1732490260-nodegroup-ng-69c8c0cb"
2024-11-24 15:27:49 [ℹ]  waiting for CloudFormation stack "eksctl-extravagant-sculpture-1732490260-nodegroup-ng-69c8c0cb"
2024-11-24 15:28:20 [ℹ]  waiting for CloudFormation stack "eksctl-extravagant-sculpture-1732490260-nodegroup-ng-69c8c0cb"
2024-11-24 15:29:11 [ℹ]  waiting for CloudFormation stack "eksctl-extravagant-sculpture-1732490260-nodegroup-ng-69c8c0cb"
2024-11-24 15:31:00 [ℹ]  waiting for CloudFormation stack "eksctl-extravagant-sculpture-1732490260-nodegroup-ng-69c8c0cb"
2024-11-24 15:31:00 [ℹ]  waiting for the control plane to become ready
2024-11-24 15:31:01 [✔]  saved kubeconfig as "/Users/gigi/.kube/config"
2024-11-24 15:31:01 [ℹ]  no tasks
2024-11-24 15:31:01 [✔]  all EKS cluster resources for "extravagant-sculpture-1732490260" have been created
2024-11-24 15:31:01 [✔]  created 0 nodegroup(s) in cluster "extravagant-sculpture-1732490260"
2024-11-24 15:31:01 [ℹ]  nodegroup "ng-69c8c0cb" has 2 node(s)
2024-11-24 15:31:01 [ℹ]  node "ip-192-168-0-42.us-west-2.compute.internal" is ready
2024-11-24 15:31:01 [ℹ]  node "ip-192-168-43-166.us-west-2.compute.internal" is ready
2024-11-24 15:31:01 [ℹ]  waiting for at least 2 node(s) to become ready in "ng-69c8c0cb"
2024-11-24 15:31:01 [ℹ]  nodegroup "ng-69c8c0cb" has 2 node(s)
2024-11-24 15:31:01 [ℹ]  node "ip-192-168-0-42.us-west-2.compute.internal" is ready
2024-11-24 15:31:01 [ℹ]  node "ip-192-168-43-166.us-west-2.compute.internal" is ready
2024-11-24 15:31:01 [✔]  created 1 managed nodegroup(s) in cluster "extravagant-sculpture-1732490260"
2024-11-24 15:31:02 [ℹ]  kubectl command should work with "/Users/gigi/.kube/config", try 'kubectl get nodes'
2024-11-24 15:31:02 [✔]  EKS cluster "extravagant-sculpture-1732490260" in "us-west-2" region is ready
```

under the covers `eksctl` uses CloudFormation to provision the resources.

Let's see how much allocatable CPU and memory we have on our nodes:

```
kubectl get node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') -o yaml | yq '.status.allocatable | {"cpu": .cpu, "memory": .memory}' | awk -F': ' '/cpu/ {cpu=$2} /memory/ {mem=$2} END {printf "cpu: %s\nmemory: %.2f Gi\n", cpu, mem / 1024 / 1024}'
cpu: 1930m
memory: 6.89 Gi
```

Not bad. about 2 cores and almost 7 GiB of memory.

# Deploy the go-quote-service

```
kubectl apply -f ../k8s/manifests.yaml
deployment.apps/go-quote created
service/go-quote-service created
horizontalpodautoscaler.autoscaling/go-quote-hpa created
```

Let's verify everything looks fine. Here is the deployment.

```
❯ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
go-quote   1/1     1            1           166m
```

Here is the service.
```
❯ kubectl get svc
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)        AGE
go-quote-service   LoadBalancer   10.100.246.249   a95fe017ca2c04783af8fb15c8145420-434346127.us-west-2.elb.amazonaws.com   80:31512/TCP   166m
kubernetes         ClusterIP      10.100.0.1       <none>                                                                   443/TCP        4h7m
```

Let's confirm we can access the go-quote srvice using the external IP.

```
❯ http -b http://a95fe017ca2c04783af8fb15c8145420-434346127.us-west-2.elb.amazonaws.com/quotes
[
    "test quote 1",
    "test quote 2"
]
```
