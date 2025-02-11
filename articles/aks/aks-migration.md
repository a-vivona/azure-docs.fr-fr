---
title: Migrer vers Azure Kubernetes Service (AKS)
description: Effectuez une migration vers Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/25/2021
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 39f1245883c7d296de5b2bad77f1558be9b49910
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/10/2021
ms.locfileid: "111887417"
---
# <a name="migrate-to-azure-kubernetes-service-aks"></a>Migrer vers Azure Kubernetes Service (AKS)

Pour vous aider à planifier et à exécuter la migration vers le service Azure Kubernetes (AKS), ce guide fournit des détails sur la configuration recommandée d’AKS. Bien que cet article ne couvre pas tous les scénarios, il propose des liens vers des informations plus détaillées sur la planification d’une migration réussie.

Ce document facilite la prise en charge des scénarios suivants :

* Conteneurisation de certaines applications et migration de celles-ci vers AKS à l’aide d’[Azure Migrate](../migrate/migrate-services-overview.md).
* Migration d’un cluster AKS reposant sur des [Groupes à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md) vers [Virtual Machine Scale Sets](../virtual-machine-scale-sets/overview.md).
* Migration d’un cluster AKS pour utiliser un [Équilibreur de charge de référence SKU Standard](./load-balancer-standard.md).
* Migration à partir d’[Azure Container Service (ACS) - Mise hors service le 31 janvier 2020](https://azure.microsoft.com/updates/azure-container-service-will-retire-on-january-31-2020/) vers AKS.
* Migration à partir du [Moteur AKS](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) vers AKS.
* Migration à partir de clusters Kubernetes non basés sur Azure vers AKS.
* Déplacement des ressources existantes vers une autre région.

Lors de la migration, assurez-vous que la version de votre Kubernetes cible entre dans le cadre de la prise en charge pour AKS. Les versions antérieures peuvent ne pas figurer dans la plage prise en charge et nécessitent un changement de version qui sera pris en charge par AKS. Pour plus d’informations, consultez la section [Versions de Kubernetes prises en charge dans AKS](./supported-kubernetes-versions.md).

Si vous effectuez une migration vers une version plus récente de Kubernetes, consultez [Stratégie de prise en charge des versions et des différences de version Kubernetes](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-versions).

Plusieurs outils open source peuvent vous être utiles dans votre migration, selon votre scénario :

* [Velero](https://velero.io/) (Nécessite Kubernetes 1.7+)
* [Extension Azure Kube CLI](https://github.com/yaron2/azure-kube-cli)
* [ReShifter](https://github.com/mhausenblas/reshifter)

Dans cet article, nous allons résumer les détails de la migration pour les points suivants :

> [!div class="checklist"]
> * Conteneurisation d’applications via Azure Migrate 
> * AKS avec Standard Load Balancer et Virtual Machine Scale Sets
> * Services Azure attachés existants
> * Garantie de la validité des quotas
> * Haute disponibilité et continuité de l’activité
> * Considérations relatives aux applications sans état
> * Points à prendre en compte sur les applications avec état
> * Déploiement de la configuration du cluster

## <a name="use-azure-migrate-to-migrate-your-applications-to-aks"></a>Utiliser Azure Migrate pour migrer vos applications vers AKS

Azure Migrate fournit une plateforme unifiée pour évaluer et migrer vers Azure des serveurs, une infrastructure, des applications et des données locaux. Pour AKS, vous pouvez utiliser Azure Migrate pour effectuer les tâches suivantes :

* [Conteneuriser les applications ASP.NET et migrer vers AKS](../migrate/tutorial-app-containerization-aspnet-kubernetes.md)
* [Conteneuriser les applications web Java et migrer vers AKS](../migrate/tutorial-containerize-java-kubernetes.md)

## <a name="aks-with-standard-load-balancer-and-virtual-machine-scale-sets"></a>AKS avec Standard Load Balancer et Virtual Machine Scale Sets

AKS est un service managé offrant des fonctionnalités uniques, avec un traitement de gestion moindre. AKS étant un service managé, vous devez sélectionner à partir d’un ensemble de [régions](./quotas-skus-regions.md) celles que AKS prend en charge. Vous devrez peut-être modifier vos applications existantes pour les conserver dans le plan de contrôle géré par AKS lors de la transition de votre cluster existant vers AKS.

Nous vous conseillons d’utiliser des clusters AKS avec des [Groupes de machines virtuelles identiques](../virtual-machine-scale-sets/index.yml) et [Azure Standard Load Balancer](./load-balancer-standard.md) pour vous assurer de disposer des fonctionnalités suivantes :
* [Pools multinœuds](./use-multiple-node-pools.md),
* [Zones de disponibilité](../availability-zones/az-overview.md),
* [Plages IP autorisées](./api-server-authorized-ip-ranges.md),
* [Autoscaler de cluster](./cluster-autoscaler.md),
* [Découvrir Azure Policy pour AKS Engine](../governance/policy/concepts/policy-for-kubernetes.md) et
* D’autres nouvelles fonctionnalités à mesure qu’elles sont publiées.

Les clusters AKS s’appuyant sur des [groupes à haute disponibilité de machines virtuelles](../virtual-machines/availability.md#availability-sets) ne prennent pas en charge la plupart de ces fonctionnalités.

L’exemple suivant crée un cluster AKS avec un pool de nœuds unique soutenu par un groupe de machines virtuelles identiques. Le cluster :
* Il utilise un équilibreur de charge standard. 
* Il active la mise à l’échelle automatique de cluster sur le pool de nœuds pour le cluster.
* Il définit un minimum de *1* et un maximum de *3* nœuds.

```azurecli-interactive
# First create a resource group
az group create --name myResourceGroup --location eastus

# Now create the AKS cluster and enable the cluster autoscaler
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 1 \
  --vm-set-type VirtualMachineScaleSets \
  --load-balancer-sku standard \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 3
```

## <a name="existing-attached-azure-services"></a>Services Azure attachés existants

Au cours de la migration des clusters, vous avez peut-être attaché des services Azure externes. Bien que les services suivants ne demandent pas à recréer les ressources, ils exigent de mettre à jour les connexions des précédents clusters vers les nouveaux clusters pour conserver les fonctionnalités.

* Azure Container Registry
* Log Analytics
* Application Insights
* Traffic Manager
* Compte de stockage
* Bases de données externes

## <a name="ensure-valid-quotas"></a>Garantie de la validité des quotas

Comme d’autres machines virtuelles seront déployées dans votre abonnement pendant la migration, vous devez vérifier que vos quotas et vos limites sont définis sur des valeurs suffisantes pour ces ressources. Si nécessaire, demandez une augmentation du [quota de processeurs virtuels](../azure-portal/supportability/per-vm-quota-requests.md).

Vous devrez peut-être demander une augmentation des [quotas réseau](../azure-portal/supportability/networking-quota-requests.md) pour vous assurer de ne pas épuiser les adresses IP. Pour plus d’informations, consultez [Mise en réseau et plages d’adresses IP pour AKS](./configure-kubenet.md).

Pour plus d’informations, consultez [Azure subscription and service limits (Limites de service et d’abonnement Azure)](../azure-resource-manager/management/azure-subscription-service-limits.md). Pour vérifier vos quotas actuels, dans le Portail Azure, accédez au [panneau Abonnements](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade), sélectionnez votre abonnement, puis sélectionnez **Utilisation + quotas**.

## <a name="high-availability-and-business-continuity"></a>Haute disponibilité et continuité de l’activité

Si votre application ne peut pas gérer les temps d’arrêt, vous devez suivre les meilleures pratiques établies pour les scénarios de migration en haute disponibilité. Plus d’informations sur les [pratiques recommandées pour la planification de la continuité d’activité complexe, la reprise d’activité et l’optimisation du temps d’activité dans Azure Kubernetes Service (AKS)](./operator-best-practices-multi-region.md).

Pour les applications complexes, vous effectuez généralement une migration progressive plutôt qu’en une seule fois, ce qui signifie que les anciens et nouveaux environnements devront peut-être communiquer sur le réseau. Les applications qui utilisaient les services `ClusterIP` pour communiquer devront peut-être être exposées en tant que type `LoadBalancer` et sécurisées de manière adéquate.

Pour effectuer la migration, vous devez rediriger les clients vers les nouveaux services qui exécutent AKS. Nous vous recommandons de rediriger le trafic en mettant à jour le DNS pour qu’il pointe vers l’équilibreur de charge se trouvant devant votre cluster AKS.

[Azure Traffic Manager](../traffic-manager/index.yml) peut diriger les clients vers le cluster Kubernetes et l’instance d’application souhaités. Traffic Manager est un équilibreur de charge de trafic DNS qui peut répartir le trafic réseau entre les régions. Pour optimiser les performances et la redondance, faites transiter l’ensemble du trafic d’application par Traffic Manager avant qu’il parvienne à votre cluster AKS. 

Dans un déploiement impliquant plusieurs clusters, les clients doivent se connecter à un nom DNS Traffic Manager qui pointe vers les services sur chaque cluster AKS. Définissez ces services en utilisant des points de terminaison Traffic Manager. Chaque point de terminaison est l’*adresse IP d’équilibreur de charge de service*. Cette configuration vous permet de diriger le trafic réseau du point de terminaison Traffic Manager d’une région vers le point de terminaison d’une autre région.

![AKS avec Traffic Manager](media/operator-best-practices-bc-dr/aks-azure-traffic-manager.png)

[Azure Front Door Service](../frontdoor/front-door-overview.md) représente une autre solution pour le routage du trafic des clusters AKS. Avec Azure Front Door Service vous pouvez définir, gérer et superviser le routage global de votre trafic web en privilégiant l’optimisation des performances et le basculement instantané global à des fins de haute disponibilité. 

### <a name="considerations-for-stateless-applications"></a>Considérations relatives aux applications sans état

La migration des applications sans état est le cas de migration le plus simple :
1. Appliquez vos définitions de ressources (YAML ou Helm) au nouveau cluster.
1. Vérifiez que tout fonctionne correctement.
1. Redirigez le trafic pour activer votre nouveau cluster.

### <a name="considerations-for-stateful-applications"></a>Points à prendre en compte sur les applications avec état

Planifiez soigneusement la migration des applications avec afin d’éviter la perte de données ou un temps d’arrêt imprévu.

* Si vous utilisez Azure Files, vous pouvez monter le partage de fichiers en tant que volume dans le nouveau cluster. Voir [Monter le service Azure Files statique en tant que volume](./azure-files-volume.md#mount-file-share-as-an-persistent-volume).
* Si vous utilisez des disques managés Azure, vous pouvez monter le disque uniquement s’il n’est pas attaché à une machine virtuelle. Voir [Monter le service Azure Disk statique en tant que volume](./azure-disk-volume.md#mount-disk-as-volume).
* Si aucune de ces approches ne fonctionne, vous pouvez utiliser les options de sauvegarde et de restauration. Voir [Velero sur Azure](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure/blob/master/README.md).

#### <a name="azure-files"></a>Azure Files

Contrairement aux disques, Azure Files peut être monté sur plusieurs hôtes en même temps. Dans votre cluster AKS, Azure et Kubernetes ne vous empêchent pas de créer un pod toujours utilisé par votre cluster AKS. Pour éviter une perte de données et un comportement inattendu, vérifiez que les clusters n’écrivent pas dans les mêmes fichiers en même temps.

Si votre application peut héberger plusieurs réplicas pointant vers le même partage de fichiers, suivez les étapes de migration sans état et déployez vos définitions YAML sur votre nouveau cluster. 

Si ce n’est pas le cas, essayez la méthode suivante :

1. Vérifiez que votre application fonctionne correctement.
1. Pointez votre trafic en direct vers votre nouveau cluster AKS.
1. Déconnectez l’ancien cluster.

Si vous souhaitez commencer avec un partage vide et copier les données sources, vous pouvez utiliser les commandes [`az storage file copy`](/cli/azure/storage/file/copy) pour migrer vos données.


#### <a name="migrating-persistent-volumes"></a>Migration des volumes persistants

Si vous migrez des volumes persistants existants vers AKS, vous suivez généralement les étapes ci-après :

1. Suspendre l’écriture dans l’application. 
    * Cette étape est facultative et nécessite un temps d’arrêt.
1. Prendre des instantanés des disques.
1. Créer des disques managés à partir d’instantanés.
1. Créer des volumes persistants dans AKS.
1. Modifier les spécifications du pod pour qu’il [utilise les volumes existants](./azure-disk-volume.md) plutôt que PersistentVolumeClaims (provisionnement statique).
1. Déployez votre application sur AKS.
1. Vérifiez que votre application fonctionne correctement.
1. Pointez votre trafic en direct vers votre nouveau cluster AKS.

> [!IMPORTANT]
> Si vous ne souhaitez pas suspendre les écritures, vous devez répliquer les données vers le nouveau déploiement. Dans le cas contraire, vous risquez de manquer les données écrites après que vous avez pris les instantanés de disque.

Des outils open source vous aident à créer des disques managés et à migrer des volumes d’un cluster Kubernetes à l’autre :

* L’[extension Azure CLI Disk Copy](https://github.com/noelbundick/azure-cli-disk-copy-extension) copie et convertit des disques dans les groupes de ressources et les régions Azure.
* L’[extension Azure Kube CLI](https://github.com/yaron2/azure-kube-cli) énumère les volumes Kubernetes ACS et effectue leur migration vers un cluster AKS.


### <a name="deployment-of-your-cluster-configuration"></a>Déploiement de la configuration du cluster

Nous vous recommandons d’utiliser votre pipeline d’intégration continue (CI) et de déploiement continu (CD) existant pour déployer une configuration éprouvée sur AKS. Vous pouvez utiliser Azure Pipelines pour [générer et déployer vos applications sur AKS](/azure/devops/pipelines/ecosystems/kubernetes/aks-template). Clonez vos tâches de déploiement existantes et vérifiez que `kubeconfig` pointe vers le nouveau cluster AKS.

Si ce n’est pas possible, exportez les définitions de ressources depuis votre cluster Kubernetes existant, puis appliquez-les à AKS. Vous pouvez utiliser `kubectl` pour exporter des objets.

```console
kubectl get deployment -o=yaml --export > deployments.yaml
```

### <a name="moving-existing-resources-to-another-region"></a>Déplacement des ressources existantes dans une autre région

Vous pouvez déplacer votre cluster AKS dans une [autre région prise en charge par AKS][region-availability]. Nous vous recommandons de créer un cluster dans l’autre région, puis de déployer vos ressources et vos applications sur le nouveau cluster. 

Par ailleurs, si vous disposez de services en cours d’exécution sur votre cluster AKS, vous devez installer et configurer ces services sur votre cluster dans la nouvelle région.


Dans cet article, nous avons synthétisé les détails de la migration pour :

> [!div class="checklist"]
> * AKS avec Standard Load Balancer et Virtual Machine Scale Sets
> * Services Azure attachés existants
> * Garantie de la validité des quotas
> * Haute disponibilité et continuité de l’activité
> * Considérations relatives aux applications sans état
> * Points à prendre en compte sur les applications avec état
> * Déploiement de la configuration du cluster


[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service