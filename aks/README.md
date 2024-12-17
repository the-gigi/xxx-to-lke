# Create a Microsoft account and go to the Azure cloud console

https://azure.microsoft.com

# Install and configure az

Follow the instructions here:
https://learn.microsoft.com/en-us/cli/azure/install-azure-cli


# Login and check your Azure project

```
az login
```

```
az account list -o table
```

To see current subscription:

```
az account show

{
  "environmentName": "AzureCloud",
  "homeTenantId": "c033a3ee-1344-4c50-8fa1-90ee7448bd64",
  "id": "b2b32916-5e4d-46eb-95ae-495c93efb6e1",
  "isDefault": true,
  "managedByTenants": [],
  "name": "the-sub",
  "state": "Enabled",
  "tenantId": "c033a3ee-1344-4c50-8fa1-90ee7448bd64",
  "user": {
    "name": "the.gigi@gmail.com",
    "type": "user"
  }
}
``` 

# Provision an AKS Automatic cluster (preview)

## Install the aks-preview Azure CLI extension

```
az extension add --name aks-preview
```

## Register the Automatic feature flag

```
az feature register --namespace Microsoft.ContainerService --name AutomaticSKUPreview

Once the feature 'AutomaticSKUPreview' is registered, invoking 'az provider register -n Microsoft.ContainerService' is required to get the change propagated
{
  "id": "/subscriptions/b2b32916-5e4d-46eb-95ae-495c93efb6e1/providers/Microsoft.Features/providers/Microsoft.ContainerService/features/AutomaticSKUPreview",
  "name": "Microsoft.ContainerService/AutomaticSKUPreview",
  "properties": {
    "state": "Registering"
  },
  "type": "Microsoft.Features/providers/features"
}
```

Wait for the feature to be registered (this can take a while):

```
az feature show --namespace Microsoft.ContainerService --name AutomaticSKUPreview
{
  "id": "/subscriptions/b2b32916-5e4d-46eb-95ae-495c93efb6e1/providers/Microsoft.Features/providers/Microsoft.ContainerService/features/AutomaticSKUPreview",
  "name": "Microsoft.ContainerService/AutomaticSKUPreview",
  "properties": {
    "state": "Registered"
  },
  "type": "Microsoft.Features/providers/features"
}
```

## Register the container service provider

```
az provider register --namespace Microsoft.ContainerService
```

Then, wait until it's registered:
```
az provider show -n Microsoft.ContainerService | rg -i providers/Microsoft.ContainerService -A 4
  "id": "/subscriptions/b2b32916-5e4d-46eb-95ae-495c93efb6e1/providers/Microsoft.ContainerService",
  "namespace": "Microsoft.ContainerService",
  "providerAuthorizationConsentState": null,
  "registrationPolicy": "RegistrationRequired",
  "registrationState": "Registered",
```

## Choose a region  location

See https://datacenters.microsoft.com/globe/explore/

Let's see what regions are available in the US west:

```
❯ az account list-locations --output table | rg westus
West US 2                 westus2              (US) West US 2
West US 3                 westus3              (US) West US 3
West US (Stage)           westusstage          (US) West US (Stage)
West US 2 (Stage)         westus2stage         (US) West US 2 (Stage)
West US                   westus               (US) West US
```

Let's pick westus2 (Washington), which was opened in 2007 and supports availability zones.


## Create a resource group

```
az group create --name the-group --location westus2
{
  "id": "/subscriptions/b2b32916-5e4d-46eb-95ae-495c93efb6e1/resourceGroups/the-group",
  "location": "westus2",
  "managedBy": null,
  "name": "the-group",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create the cluster

See https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli

Type this command to create the cluster:

```
az aks create \
    --resource-group the-group \
    --name the-cluster \
    --sku automatic \
    --generate-ssh-keys
    
Argument '--sku' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
The behavior of this command has been altered by the following extension: aks-preview
SSH key files '/Users/gigi/.ssh/id_rsa' and '/Users/gigi/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage like Azure Cloud Shell without an attached file share, back up your keys to a safe location
The new node pool will enable SSH access, recommended to use '--ssh-access disabled' option to disable SSH access for the node pool to make it more secure.
Resource provider 'Microsoft.OperationalInsights' used by this operation is not registered. We are registering for you.    
```

Rename the SSH keys
```bash
cd ~/.ssh
mv id_rsa aks_id_rsa
mv id_rsa.pub aks_id_rsa.pub
```


Hmm... missing the insights provider. Let's register it:

```
az provider register --namespace Microsoft.Insights
```

Then, wait until it's registered (it can take a while:
```
az provider show -n Microsoft.Insights | rg -i providers/Microsoft.Insights -A 4
  "id": "/subscriptions/b2b32916-5e4d-46eb-95ae-495c93efb6e1/providers/microsoft.insights",
  "namespace": "microsoft.insights",
  "providerAuthorizationConsentState": null,
  "registrationPolicy": "RegistrationRequired",
  "registrationState": "Registered",
```

It needs the compute provider too:
```
az provider register --namespace Microsoft.Compute
```

Then, wait until it's registered (it can take a while):

```
az provider show -n Microsoft.Compute | rg -i providers/Microsoft.Compute -A 4
  "id": "/subscriptions/b2b32916-5e4d-46eb-95ae-495c93efb6e1/providers/Microsoft.Compute",
  "namespace": "Microsoft.Compute",
  "providerAuthorizationConsentState": null,
  "registrationPolicy": "RegistrationRequired",
  "registrationState": "Registered",
```

Hmmm... not enough quota
```
az aks create \
    --resource-group the-group \
    --name the-cluster \
    --sku automatic \
    --ssh-access disabled \


Argument '--sku' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Argument '--ssh-access' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
The behavior of this command has been altered by the following extension: aks-preview
(BadRequest) AKS Automatic could not find a suitable VM size. The subscription may not have the required quota of '16' vCPUs, may have restrictions, or location 'westus2' may not support three availability zones for the following VM sizes: 'standard_d4pds_v5,standard_d4lds_v5,standard_d4ads_v5,standard_d4ds_v5,standard_d4d_v5,standard_d4d_v4,standard_ds3_v2,standard_ds12_v2'. For more information, please visit https://aka.ms/aks-automatic-quota
Code: BadRequest
Message: AKS Automatic could not find a suitable VM size. The subscription may not have the required quota of '16' vCPUs, may have restrictions, or location 'westus2' may not support three availability zones for the following VM sizes: 'standard_d4pds_v5,standard_d4lds_v5,standard_d4ads_v5,standard_d4ds_v5,standard_d4d_v5,standard_d4d_v4,standard_ds3_v2,standard_ds12_v2'. For more information, please visit https://aka.ms/aks-automatic-quota
```

The quota is just 10 cpus per VM size.  Forget it... let's create a manual cluster

```
az aks create \
    --resource-group the-group \
    --name the-cluster \
    --node-count 1 \
    --generate-ssh-keys    
```

## Get the credentials

```
❯ az aks get-credentials --resource-group the-group --name the-cluster
The behavior of this command has been altered by the following extension: aks-preview
Merged "the-cluster" as current context in /Users/gigi/.kube/config
```

## Access the cluster

```
kubectl cluster-info
Kubernetes control plane is running at https://the-cluste-the-group-b2b329-6t3yu9ag.hcp.westus2.azmk8s.io:443
CoreDNS is running at https://the-cluste-the-group-b2b329-6t3yu9ag.hcp.westus2.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://the-cluste-the-group-b2b329-6t3yu9ag.hcp.westus2.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Let's check the version. It's 1.30

```
kubectl version
Client Version: v1.31.3
Kustomize Version: v5.4.2
Server Version: v1.30.6
```

Let's check out the nodes. We have one node with the same version as the control plane.

```
kubectl get no
NAME                                STATUS   ROLES    AGE   VERSION
aks-nodepool1-16089133-vmss000000   Ready    <none>   11h   v1.30.6
```


Let's check the available CPU and memory:

```
kubectl get node $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') -o yaml | yq '.status.allocatable | {"cpu": .cpu, "memory": .memory}' | awk -F': ' '/cpu/ {cpu=$2} /memory/ {mem=$2} END {printf "cpu: %s\nmemory: %.2f Gi\n", cpu, mem / 1024 / 1024}'

cpu: 1900m
memory: 4.92 Gi
```

Not bad. about 2 cores and almost 5 GiB of memory.

# Deploy the go-quote-service

```
kubectl apply -f ../k8s/manifests.yaml
deployment.apps/go-quote created
service/go-quote-service created
horizontalpodautoscaler.autoscaling/go-quote-hpa created
```



Let's verify everything looks fine. Here is the deployment.

```
kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
go-quote   1/1     1            1           49s
```

Here is the service.
```
kubectl get svc
NAME               TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE
go-quote-service   LoadBalancer   10.0.71.133   172.179.145.44   80:31543/TCP   4m36s
kubernetes         ClusterIP      10.0.0.1      <none>           443/TCP        11h
```

Let's confirm we can access the go-quote srvice using the external IP.

```
http -b http://172.179.145.44/quotes quote='test quote 1'
http -b http://172.179.145.44/quotes quote='test quote 2'
http -b http://172.179.145.44/quotes
[
    "test quote 1",
    "test quote 2"
]
```

# Shutdown the cluster

List the clusters
```
az aks list --output table

The behavior of this command has been altered by the following extension: aks-preview
Name         Location    ResourceGroup    KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
-----------  ----------  ---------------  -------------------  --------------------------  -------------------  ----------------------------------------------------------
the-cluster  westus2     the-group        1.30                 1.30.6                      Succeeded            the-cluste-the-group-b2b329-6t3yu9ag.hcp.westus2.azmk8s.io
```

Delete the cluster
```
az aks delete --name the-cluster --resource-group the-group --yes
```