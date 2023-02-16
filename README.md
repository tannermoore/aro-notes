### cloudshell config
```
cd ~
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz
mkdir openshift
tar -zxvf openshift-client-linux.tar.gz -C openshift
echo 'export PATH=$PATH:~/openshift' >> ~/.bashrc && source ~/.bashrc
```

### register resource providers
```
az provider register -n Microsoft.RedHatOpenShift --wait && \
az provider register -n Microsoft.Compute --wait && \
az provider register -n Microsoft.Storage --wait 
```

### env vars
```
LOCATION=southcentralus
RESOURCEGROUP=rg-aro-demo
BACKENDRESOURCEGROUP=rg-aro-demo-backend
CLUSTER=aro-demo-cluster
VNET=vnet-aro-demo
MASTERSUB=sub-aro-demo-master
WORKERSUB=sub-aro-demo-worker
TAGS="env=test" 
```

### create resource group
```
az group create --name $RESOURCEGROUP --location $LOCATION --output table 
```

### create vnet
```
az network vnet create \
 --resource-group $RESOURCEGROUP \
 --name $VNET \
 --address-prefixes 10.0.0.0/22 --output table 
```

### create control plane and worker subnets
```
az network vnet subnet create \
 --resource-group $RESOURCEGROUP \
 --vnet-name $VNET \
 --name $MASTERSUB \
 --address-prefixes 10.0.0.0/23 \
 --service-endpoints Microsoft.ContainerRegistry --output table 
```
```
az network vnet subnet create \
 --resource-group $RESOURCEGROUP \
 --vnet-name $VNET \
 --name $WORKERSUB \
 --address-prefixes 10.0.2.0/23 \
 --service-endpoints Microsoft.ContainerRegistry --output table 

```

### modify master sub to disable private endpoints so that we can connect to and manage the cluster
```
az network vnet subnet update \
 --name $MASTERSUB \
 --resource-group $RESOURCEGROUP \
 --vnet-name $VNET \
 --disable-private-link-service-network-policies true --output table 
```

### create cluster (pull secret needed)
```
date && az aro create \
--resource-group $RESOURCEGROUP \
--cluster-resource-group $BACKENDRESOURCEGROUP \
--name $CLUSTER \
--vnet $VNET \
--master-subnet $MASTERSUB \
--worker-subnet $WORKERSUB \
--debug \
--tags $TAGS \
--pull-secret @pull-secret.txt \
--no-wait 
```

### check state 
```
az aro list -o table
```

### get kubeadmin creds
```
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP 
```

### get console url
```
az aro list -o table 

```
