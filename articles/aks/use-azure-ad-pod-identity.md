---
title: Utiliser des identités managées par pod Azure Active Directory dans Azure Kubernetes Service (préversion)
description: Découvrez comment utiliser des identités managées par pod AAD dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 3/12/2021
ms.openlocfilehash: 1b1e8ab4e95a0f721f83f933b527cc40b9d5747c
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/08/2021
ms.locfileid: "111592584"
---
# <a name="use-azure-active-directory-pod-managed-identities-in-azure-kubernetes-service-preview"></a>Utiliser des identités managées par pod Azure Active Directory dans Azure Kubernetes Service (préversion)

Les identités managées par pod Azure Active Directory utilisent des primitives Kubernetes pour associer des [identités managées pour les ressources Azure][az-managed-identities] et des identités dans Azure Active Directory (AAD) avec des pods. Les administrateurs créent des identités et des liaisons en tant que primitives Kubernetes qui permettent aux pods d’accéder aux ressources Azure qui reposent sur AAD en tant que fournisseur d’identité.

> [!NOTE]
>La fonctionnalité décrite dans ce document, les identités managées par pod (préversion), sera remplacée par les identités managées par pod V2 (préversion).
> Si vous disposez d’une installation existante de AADPODIDENTITY, vous devez supprimer l’installation existante. L’activation de cette fonctionnalité signifie que le composant MIC n’est pas nécessaire.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Avant de commencer

La ressource suivante doit être installée :

* Azure CLI, version 2.20.0 ou ultérieure
* L’extension `azure-preview` version 0.5.5 ou ultérieure

### <a name="limitations"></a>Limites

* 200 identités de pods maximum sont autorisées pour un cluster.
* 200 exceptions d’identités de pods maximum sont autorisées pour un cluster.
* Les identités managées par pod sont disponibles uniquement sur les pools de nœuds Linux.

### <a name="register-the-enablepodidentitypreview"></a>Inscrivez `EnablePodIdentityPreview`

Inscrivez la fonctionnalité `EnablePodIdentityPreview` :

```azurecli
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
```

### <a name="install-the-aks-preview-azure-cli"></a>Installez l’interface de ligne de commande Azure `aks-preview`

Vous devez également disposer de l’extension Azure CLI *aks-preview* version 0.4.64 ou ultérieure. Installez l’extension d’Azure CLI *aks-preview* à l’aide de la commande [az extension add][az-extension-add]. Ou installez toutes les mises à jour disponibles à l’aide de la commande [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

## <a name="create-an-aks-cluster-with-azure-cni"></a>Créer un cluster AKS avec Azure CNI

> [!NOTE]
> Il s’agit de la configuration recommandée par défaut.

Créez un cluster AKS avec Azure CNI et une identité managée par pod activée. Les commande suivantes utilisent [az group create][az-group-create] pour créer un groupe de ressources nommé *myResourceGroup* et la commande [az aks create][az-aks-create] pour créer un cluster AKS nommé *myAKSCluster* dans le groupe de ressources *myResourceGroup*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
az aks create -g myResourceGroup -n myAKSCluster --enable-pod-identity --network-plugin azure
```

Utilisez [az aks get-credentials][az-aks-get-credentials] pour vous connecter à votre cluster AKS. Cette commande télécharge et configure également le certificat client `kubectl` sur votre ordinateur de développement.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="update-an-existing-aks-cluster-with-azure-cni"></a>Mettre à jour un cluster AKS existant avec Azure CNI

Mettez à jour un cluster AKS existant avec Azure CNI pour inclure l’identité managée par pod.

```azurecli-interactive
az aks update -g $MY_RESOURCE_GROUP -n $MY_CLUSTER --enable-pod-identity --network-plugin azure
```
## <a name="using-kubenet-network-plugin-with-azure-active-directory-pod-managed-identities"></a>Utilisation du plug-in réseau Kubenet avec des identités managées par pod Azure Active Directory 

> [!IMPORTANT]
> L’exécution d’aad-pod-identity dans un cluster avec Kubenet n’est pas une configuration recommandée en raison de l’implication de sécurité. Suivez les étapes d’atténuation et configurez les stratégies avant d’activer aad-pod-identity dans un cluster avec Kubenet.

## <a name="mitigation"></a>Limitation des risques

Pour atténuer la vulnérabilité au niveau du cluster, vous pouvez utiliser la stratégie intégrée Azure « Les conteneurs de cluster Kubernetes doivent utiliser uniquement des fonctionnalités autorisées » pour limiter les attaques de type CAP_NET_RAW.  

Ajouter NET_RAW à « Fonctionnalités Drop requises »

![image](https://user-images.githubusercontent.com/50749048/118558790-206b8880-b735-11eb-9e48-236b81116812.png)

Si vous n’utilisez pas Azure Policy, vous pouvez combiner le contrôleur d’admission OpenPolicyAgent et le webhook de validation de Gatekeeper. Si Gatekeeper est déjà installé dans votre cluster, ajoutez le modèle ConstraintTemplate de type K8sPSPCapabilities :

```
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper-library/master/library/pod-security-policy/capabilities/template.yaml
```
Ajoutez un modèle pour limiter la génération de Pods à l’aide de la fonctionnalité NET_RAW :

```
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPSPCapabilities
metadata:
  name: prevent-net-raw
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    excludedNamespaces:
      - "kube-system"
  parameters:
    requiredDropCapabilities: ["NET_RAW"]
```

## <a name="create-an-aks-cluster-with-kubenet-network-plugin"></a>Créer un cluster AKS avec le plug-in réseau Kubenet

Créez un cluster AKS avec un plug-in réseau Kubenet et une identité managée par pod activée.

```azurecli-interactive
az aks create -g $MY_RESOURCE_GROUP -n $MY_CLUSTER --enable-pod-identity --enable-pod-identity-with-kubenet
```

## <a name="update-an-existing-aks-cluster-with-kubenet-network-plugin"></a>Mettre à jour un cluster AKS existant avec le plug-in réseau Kubenet

Mettez à jour un cluster AKS existant avec le plug-in réseau Kubenet pour inclure l’identité managée par pod.

```azurecli-interactive
az aks update -g $MY_RESOURCE_GROUP -n $MY_CLUSTER --enable-pod-identity --enable-pod-identity-with-kubenet
```

## <a name="create-an-identity"></a>Créer une identité

Créez une identité avec [az identity create][az-identity-create] et définissez les variables *IDENTITY_CLIENT_ID* et *IDENTITY_RESOURCE_ID*.

```azurecli-interactive
az group create --name myIdentityResourceGroup --location eastus
export IDENTITY_RESOURCE_GROUP="myIdentityResourceGroup"
export IDENTITY_NAME="application-identity"
az identity create --resource-group ${IDENTITY_RESOURCE_GROUP} --name ${IDENTITY_NAME}
export IDENTITY_CLIENT_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query clientId -otsv)"
export IDENTITY_RESOURCE_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query id -otsv)"
```

## <a name="assign-permissions-for-the-managed-identity"></a>Attribuer des autorisations pour l’identité managée

L’identité managée *IDENTITY_CLIENT_ID* doit avoir des autorisations Opérateur d’identité managée dans le groupe de ressources qui contient le groupe de machines virtuelles identiques de votre cluster AKS.

```azurecli-interactive
NODE_GROUP=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NODES_RESOURCE_ID=$(az group show -n $NODE_GROUP -o tsv --query "id")
az role assignment create --role "Managed Identity Operator" --assignee "$IDENTITY_CLIENT_ID" --scope $NODES_RESOURCE_ID
```

## <a name="create-a-pod-identity"></a>Créer une identité de pod

Créez une identité de pod pour le cluster à l’aide de `az aks pod-identity add`.

> [!IMPORTANT]
> Vous devez disposer des autorisations appropriées, telles que `Owner`, sur votre abonnement pour créer l’identité et la liaison de rôle.

```azurecli-interactive
export POD_IDENTITY_NAME="my-pod-identity"
export POD_IDENTITY_NAMESPACE="my-app"
az aks pod-identity add --resource-group myResourceGroup --cluster-name myAKSCluster --namespace ${POD_IDENTITY_NAMESPACE}  --name ${POD_IDENTITY_NAME} --identity-resource-id ${IDENTITY_RESOURCE_ID}
```

> [!NOTE]
> Lorsque vous activez l’identité managée par pod sur votre cluster AKS, une AzurePodIdentityException nommée *aks-addon-exception* est ajoutée à l’espace de noms *kube-system*. Une AzurePodIdentityException permet aux pods avec certaines étiquettes d’accéder au point de terminaison Azure Instance Metadata Service (IMDS) sans être interceptés par le serveur d’identité managée par nœud (NMI). *aks-addon-exception* permet aux modules complémentaires internes AKS, comme une identité managée par pod AAD, de fonctionner sans avoir à configurer manuellement une AzurePodIdentityException. Si vous le souhaitez, vous pouvez ajouter, supprimer et mettre à jour une AzurePodIdentityException à l’aide de `az aks pod-identity exception add`, `az aks pod-identity exception delete`, `az aks pod-identity exception update` ou `kubectl`.

## <a name="run-a-sample-application"></a>Exécuter un exemple d’application

Pour qu’un pod utilise l’identité managée par pod AAD, le pod a besoin d’une étiquette *aadpodidbinding* avec une valeur qui correspond à un sélecteur issu d’une *AzureIdentityBinding*. Pour exécuter un exemple d’application à l’aide d’une identité managée par pod AAD, créez un fichier `demo.yaml` avec le contenu suivant. Remplacez *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* et *IDENTITY_RESOURCE_GROUP* par les valeurs issues des étapes précédentes. Remplacez *SUBSCRIPTION_ID* par l’ID de votre abonnement.

> [!NOTE]
> Au cours des étapes précédentes, vous avez créé les variables *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* et *IDENTITY_RESOURCE_GROUP*. Vous pouvez utiliser une commande telle que `echo` pour afficher la valeur que vous définissez pour les variables, par exemple `echo $IDENTITY_NAME`.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: demo
  labels:
    aadpodidbinding: POD_IDENTITY_NAME
spec:
  containers:
  - name: demo
    image: mcr.microsoft.com/oss/azure/aad-pod-identity/demo:v1.6.3
    args:
      - --subscriptionid=SUBSCRIPTION_ID
      - --clientid=IDENTITY_CLIENT_ID
      - --resourcegroup=IDENTITY_RESOURCE_GROUP
    env:
      - name: MY_POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: MY_POD_NAMESPACE
        valueFrom:
          fieldRef:
            fieldPath: metadata.namespace
      - name: MY_POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
  nodeSelector:
    kubernetes.io/os: linux
```

Notez que la définition du pod a une étiquette *aadpodidbinding* avec une valeur qui correspond au nom de l’identité de pod que vous avez exécutée `az aks pod-identity add` à l’étape précédente.

Déployez `demo.yaml` dans le même espace de noms que votre identité de pod à l’aide de `kubectl apply` :

```azurecli-interactive
kubectl apply -f demo.yaml --namespace $POD_IDENTITY_NAMESPACE
```

Vérifiez que l’exemple d’application s’exécute correctement avec `kubectl logs`.

```azurecli-interactive
kubectl logs demo --follow --namespace $POD_IDENTITY_NAMESPACE
```

Vérifiez que les journaux indiquent qu’un jeton a été acquis et que l’opération *GET* est réussie.
 
```output
...
successfully doARMOperations vm count 0
successfully acquired a token using the MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token)
successfully acquired a token, userAssignedID MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token) clientID(xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
successfully made GET on instance metadata
...
```
## <a name="run-an-application-with-multiple-identities"></a>Exécuter une application avec plusieurs identités

## <a name="create-multiple-identities"></a>Créer plusieurs identités

Créez les identités avec [az identity create][az-identity-create] et définissez les variables *IDENTITY_CLIENT_ID* et *IDENTITY_RESOURCE_ID*.

```azurecli-interactive
az group create --name myIdentityResourceGroup --location eastus
export IDENTITY_RESOURCE_GROUP="myIdentityResourceGroup"
export IDENTITY_NAME_1="application-identity_1"
az identity create --resource-group ${IDENTITY_RESOURCE_GROUP} --name ${IDENTITY_NAME_1}
export IDENTITY_NAME_2="application-identity_2"
az identity create --resource-group ${IDENTITY_RESOURCE_GROUP} --name ${IDENTITY_NAME_2}
export IDENTITY_CLIENT_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME_1} --query clientId -otsv)"
export IDENTITY_RESOURCE_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME_1} --query id -otsv)"
export IDENTITY_CLIENT_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME_2} --query clientId -otsv)"
export IDENTITY_RESOURCE_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME_2} --query id -otsv)"
```

## <a name="assign-permissions-for-the-managed-identities"></a>Attribuer des autorisations aux identités managées

L’identité managée *IDENTITY_CLIENT_ID* doit avoir des autorisations de Lecteur dans le groupe de ressources qui contient le groupe de machines virtuelles identiques de votre cluster AKS.

```azurecli-interactive
NODE_GROUP=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NODES_RESOURCE_ID=$(az group show -n $NODE_GROUP -o tsv --query "id")
az role assignment create --role "Reader" --assignee "$IDENTITY_CLIENT_ID_1" --scope $NODES_RESOURCE_ID
az role assignment create --role "Reader" --assignee "$IDENTITY_CLIENT_ID_2" --scope $NODES_RESOURCE_ID
```

## <a name="create-pod-identities"></a>Créer des identités de pod

Créez des identités de pod pour le cluster en utilisant `az aks pod-identity add`.

> [!IMPORTANT]
> Vous devez disposer des autorisations appropriées, telles que `Owner`, sur votre abonnement pour créer l’identité et la liaison de rôle.

```azurecli-interactive
export POD_IDENTITY_NAME="my-pod-identity"
export POD_IDENTITY_NAMESPACE="my-app"
az aks pod-identity add --resource-group myResourceGroup --cluster-name myAKSCluster --namespace ${POD_IDENTITY_NAMESPACE}  --name ${POD_IDENTITY_NAME} --identity-resource-id ${IDENTITY_RESOURCE_ID_1} --binding-selector foo
az aks pod-identity add --resource-group myResourceGroup --cluster-name myAKSCluster --namespace ${POD_IDENTITY_NAMESPACE}  --name ${POD_IDENTITY_NAME} --identity-resource-id ${IDENTITY_RESOURCE_ID_2} --binding-selector foo
```

> [!NOTE]
> Lorsque vous activez l’identité managée par pod sur votre cluster AKS, une AzurePodIdentityException nommée *aks-addon-exception* est ajoutée à l’espace de noms *kube-system*. Une AzurePodIdentityException permet aux pods avec certaines étiquettes d’accéder au point de terminaison Azure Instance Metadata Service (IMDS) sans être interceptés par le serveur d’identité managée par nœud (NMI). *aks-addon-exception* permet aux modules complémentaires internes AKS, comme une identité managée par pod AAD, de fonctionner sans avoir à configurer manuellement une AzurePodIdentityException. Si vous le souhaitez, vous pouvez ajouter, supprimer et mettre à jour une AzurePodIdentityException à l’aide de `az aks pod-identity exception add`, `az aks pod-identity exception delete`, `az aks pod-identity exception update` ou `kubectl`.

## <a name="run-a-sample-application-with-multiple-identities"></a>Exécuter un exemple d’application avec plusieurs identités

Pour qu’un pod utilise l’identité managée par pod AAD, le pod a besoin d’une étiquette *aadpodidbinding* avec une valeur qui correspond à un sélecteur issu d’une *AzureIdentityBinding*. Pour exécuter un exemple d’application à l’aide d’une identité managée par pod AAD, créez un fichier `demo.yaml` avec le contenu suivant. Remplacez *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* et *IDENTITY_RESOURCE_GROUP* par les valeurs issues des étapes précédentes. Remplacez *SUBSCRIPTION_ID* par l’ID de votre abonnement.

> [!NOTE]
> Au cours des étapes précédentes, vous avez créé les variables *POD_IDENTITY_NAME*, *IDENTITY_CLIENT_ID* et *IDENTITY_RESOURCE_GROUP*. Vous pouvez utiliser une commande telle que `echo` pour afficher la valeur que vous définissez pour les variables, par exemple `echo $IDENTITY_NAME`.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: demo
  labels:
    aadpodidbinding: foo
spec:
  containers:
  - name: demo
    image: mcr.microsoft.com/oss/azure/aad-pod-identity/demo:v1.6.3
    args:
      - --subscriptionid=SUBSCRIPTION_ID
      - --clientid=IDENTITY_CLIENT_ID
      - --resourcegroup=IDENTITY_RESOURCE_GROUP
    env:
      - name: MY_POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: MY_POD_NAMESPACE
        valueFrom:
          fieldRef:
            fieldPath: metadata.namespace
      - name: MY_POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
  nodeSelector:
    kubernetes.io/os: linux
```

Notez que la définition du pod a une étiquette *aadpodidbinding* avec une valeur qui correspond au nom de l’identité de pod que vous avez exécutée `az aks pod-identity add` à l’étape précédente.

Déployez `demo.yaml` dans le même espace de noms que votre identité de pod à l’aide de `kubectl apply` :

```azurecli-interactive
kubectl apply -f demo.yaml --namespace $POD_IDENTITY_NAMESPACE
```

Vérifiez que l’exemple d’application s’exécute correctement avec `kubectl logs`.

```azurecli-interactive
kubectl logs demo --follow --namespace $POD_IDENTITY_NAMESPACE
```

Vérifiez que les journaux indiquent qu’un jeton a été acquis et que l’opération *GET* est réussie.
 
```output
...
successfully doARMOperations vm count 0
successfully acquired a token using the MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token)
successfully acquired a token, userAssignedID MSI, msiEndpoint(http://169.254.169.254/metadata/identity/oauth2/token) clientID(xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
successfully made GET on instance metadata
...
```
export IDENTITY_CLIENT_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query clientId -otsv)" export IDENTITY_RESOURCE_ID="$(az identity show -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME} --query id -otsv)"
```

## Clean up

To remove AAD pod-managed identity from your cluster, remove the sample application and the pod identity from the cluster. Then remove the identity.

```azurecli-interactive
kubectl delete pod demo --namespace $POD_IDENTITY_NAMESPACE
az aks pod-identity delete --name ${POD_IDENTITY_NAME} --namespace ${POD_IDENTITY_NAMESPACE} --resource-group myResourceGroup --cluster-name myAKSCluster
az identity delete -g ${IDENTITY_RESOURCE_GROUP} -n ${IDENTITY_NAME}
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les identités managées, consultez [Identités managées pour les ressources Azure][az-managed-identities].

<!-- LINKS - external -->
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[az-group-create]: /cli/azure/group#az_group_create
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-managed-identities]: ../active-directory/managed-identities-azure-resources/overview.md
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
