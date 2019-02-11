# Deploy to Kubernetes Task

## Azure Pipelines

Use this task in a build or release pipeline to deploy, configure, or update a Kubernetes cluster by running kubectl commands.
Service Connection

The task works with two service connection types: 

- Azure Resource Manager and 
- Kubernetes Service Connection, described below.

### 1 of 2: Azure Resource Manager

 - connectionType
   - Service connection type	(Required) Azure Resource Manager when using Azure Kubernetes Service, or Kubernetes Service Connection for any other cluster.
   - Default value: Azure Resource Manager
 - azureSubscriptionEndpoint
   - Azure subscription	(Required) Name of the Azure Service Connection.
 - azureResourceGroup
   - Resource group	(Required) Name of the resource group within the subscription.
 - kubernetesCluster
   - Kubernetes cluster	(Required) Name of the AKS cluster.
 - namespace
   - Namespace	(Optional) The namespace on which the kubectl commands are to be run. If unspecified, the default namespace is used.

#### Examples

```
variables:
    azureSubscriptionEndpoint: Contoso
    azureContainerRegistry: contoso.azurecr.io
    azureResourceGroup: Contoso
    kubernetesCluster: Contoso

steps:
- task: Kubernetes@1
  displayName: kubectl apply
  inputs:
    connectionType: Azure Resource Manager
    azureSubscriptionEndpoint: $(azureSubscriptionEndpoint)
    azureResourceGroup: $(azureResourceGroup)
    kubernetesCluster: $(kubernetesCluster)
 ```


### 2 of 2: Kubernetes Service Connection

- kubernetesServiceEndpoint
  - Kubernetes service connection	(Required) Select a Kubernetes service connection.
- namespace
  - Namespace	(Optional) The namespace on which the kubectl commands are to be run. If not specified, the default namespace is used.

```
- task: Kubernetes@1
  displayName: kubectl apply
  inputs:
    connectionType: Kubernetes Service Connection
    kubernetesServiceEndpoint: Contoso
```

## Commands: How to do a kubectl apply

> Search how to get azureSubscriptionEndpoint, azureResourceGroup, etc.

```
- task: Kubernetes@1
  displayName: kubectl apply using arguments
  inputs:
    connectionType: Azure Resource Manager
    azureSubscriptionEndpoint: $(azureSubscriptionEndpoint)
    azureResourceGroup: $(azureResourceGroup)
    kubernetesCluster: $(kubernetesCluster)
    command: apply
    arguments: -f mhc-aks.yaml
```

### What is a configuration file?

Notice it says "useConfigurationFile".

> Research  - what does this mean?

```
- task: Kubernetes@1
  displayName: kubectl apply using configFile
  inputs:
    connectionType: Azure Resource Manager
    azureSubscriptionEndpoint: $(azureSubscriptionEndpoint)
    azureResourceGroup: $(azureResourceGroup)
    kubernetesCluster: $(kubernetesCluster)
    command: apply
    useConfigurationFile: true
    configuration: mhc-aks.yaml
```    

## Secrets

Kubernetes objects of type secret are intended to hold sensitive information such as passwords, OAuth tokens, and ssh keys.

> Research ImagePullSecrets service account

**secretType** - you can use **dockerRegistry**. Relates to **imagePullSecrets**.

**containerRegistryType** - Container registry type	(Required) Acceptable values: Azure Container Registry, or Container Registry for any other registry.

**azureSubscription, EndpointForSecrets** - Azure subscription	(Required if secretType == dockerRegistry and containerRegistryType == Azure Container Registry) Azure Resource Manager service connection scoped to the subscription containing the Azure Container Registry for which the ImagePullSecret is to be set up.

**azureContainerRegistry** - Azure container registry	(Required if secretType == dockerRegistry and containerRegistryType == Azure Container Registry) The Azure Container Registry for which the ImagePullSecret is to be set up. 


### ImagePullSecrets Example

```
- task: Kubernetes@1
      displayName: kubectl apply for secretType dockerRegistry
      inputs:
        azureSubscriptionEndpoint: $(azureSubscriptionEndpoint)
        azureResourceGroup: $(azureResourceGroup)
        kubernetesCluster: $(kubernetesCluster)
        command: apply
        arguments: -f mhc-aks.yaml
        secretType: dockerRegistry
        containerRegistryType: Azure Container Registry
        azureSubscriptionEndpointForSecrets: $(azureSubscriptionEndpoint)
        azureContainerRegistry: $(azureContainerRegistry)
        secretName: mysecretkey2
        forceUpdate: true
```

## ConfigMap


### Advanced