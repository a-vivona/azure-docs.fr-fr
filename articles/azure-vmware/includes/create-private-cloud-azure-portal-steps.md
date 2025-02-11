---
title: Créer un cloud privé Azure VMware Solution
description: Étapes de création d’un cloud privé Azure VMware Solution à l’aide du portail Azure.
ms.topic: include
ms.date: 06/17/2021
ms.openlocfilehash: cb357bf5e0fa42c3c67531e45a51b8dbba6aa6da
ms.sourcegitcommit: beff1803eeb28b60482560eee8967122653bc19c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2021
ms.locfileid: "113431086"
---
<!-- Used in deploy-azure-vmware-solution.md and tutorial-create-private-cloud.md -->

Vous pouvez créer un cloud privé Azure VMware Solution en utilisant le portail Azure ou Azure CLI.


### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Créer une ressource**. 

1. Dans la zone de texte **Rechercher dans la Place de marché**, tapez `Azure VMware Solution`, puis sélectionnez **Azure VMware Solution** dans la liste. 

1. Dans la fenêtre **Azure VMware Solution**, sélectionnez **Créer**.

1. Sous l’onglet **De base**, entrez des valeurs pour les champs. 

   >[!TIP]
   >Vous avez collecté ces informations pendant la [phase de planification](../production-ready-deployment-steps.md) de ce démarrage rapide.

   | Champ   | Valeur  |
   | ---| --- |
   | **Abonnement** | Sélectionnez l’abonnement que vous prévoyez d’utiliser pour le déploiement. Toutes les ressources d’un abonnement Azure sont facturées en même temps.|
   | **Groupe de ressources** | Sélectionnez le groupe de ressources pour votre cloud privé. Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Vous pouvez également créer un nouveau groupe de ressources pour votre cloud privé. |
   | **Lieu** | Sélectionnez un emplacement, comme **USA Est**. Il s’agit de la *région* que vous avez définie pendant la phase de planification. |
   | **Nom de la ressource** | Fournissez le nom de votre cloud privé Azure VMware Solution. |
   | **Référence (SKU)** | Sélectionnez **AV36**. |
   | **Hôtes** | Affiche le nombre d’hôtes alloués au cluster de cloud privé. La valeur par défaut est 3. Elle peut être augmentée ou diminuée après le déploiement.  |
   | **Bloc d’adresses** | Fournissez un bloc d’adresses IP pour le cloud privé.  Le CIDR représente le réseau de gestion de cloud privé et sera utilisé pour les services de gestion de cluster, tels que vCenter Server et NSX-T Manager. Utilisez l’espace d’adressage /22, par exemple 10.175.0.0/22.  L’adresse doit être unique et ne pas empiéter sur autres réseaux virtuels Azure et des réseaux locaux. |
   | **Réseau virtuel** | Laissez ce champ vide, car le circuit ExpressRoute Azure VMware Solution est établi comme une étape post-déploiement.   |

   :::image type="content" source="../media/tutorial-create-private-cloud/create-private-cloud.png" alt-text="Capture d’écran représentant l’onglet De base dans la fenêtre Créer un cloud privé." border="true":::

1. Quand vous avez terminé, sélectionnez **Vérifier + créer**. Dans l’écran suivant, vérifiez les informations entrées. Si elles sont toutes correctes, sélectionnez **Créer**.

   > [!NOTE]
   > Cette étape prend environ 3 à 4 minutes. L’ajout d’un seul hôte dans un cluster existant ou le même cluster prend entre 30 et 45 minutes.

1. Vérifiez que déploiement a réussi. Accédez au groupe de ressources que vous avez créé et sélectionnez votre cloud privé.  Quand le déploiement est terminé, vous voyez l’état **Réussi**. 

   :::image type="content" source="../media/tutorial-create-private-cloud/validate-deployment.png" alt-text="Capture d’écran montrant que le déploiement a réussi." border="true":::


### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Pour créer un cloud privé Azure VMware Solution, vous pouvez utiliser l’interface Azure CLI à l’aide d’Azure Cloud Shell au lieu du portail Azure. Pour obtenir la liste des commandes que vous pouvez utiliser avec Azure VMware Solution, consultez [Commandes Azure VMware](/cli/azure/ext/vmware/vmware).

Pour commencer avec Azure CLI :

[!INCLUDE [azure-cli-prepare-your-environment-no-header](../../../includes/azure-cli-prepare-your-environment-no-header.md)]


1. Créez un groupe de ressources avec la commande ['az group create'](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

   ```azurecli-interactive
   
   az group create --name myResourceGroup --location eastus
   ```

2. Spécifiez un nom pour le groupe de ressources et le cloud privé, un emplacement et la taille du cluster.

   | Propriété  | Description  |
   | --------- | ------------ |
   | **-g** (nom de groupe de ressources)     | Nom du groupe de ressources pour vos ressources de cloud privé.        |
   | **-n** (nom du cloud privé)     | Nom de votre cloud privé Azure VMware Solution.        |
   | **--location**     | Région utilisée pour votre cloud privé.         |
   | **--cluster-size**     | Taille du cluster La valeur minimale est 3.         |
   | **--network-block**     | Bloc réseau d’adresses IP CIDR à utiliser pour votre cloud privé. Le bloc d’adresses ne doit pas chevaucher ceux utilisés dans d’autres réseaux virtuels se trouvant dans votre abonnement et des réseaux locaux.        |
   | **--sku** | Valeur de référence SKU : AV36 |

   ```azurecli-interactive 
   az vmware private-cloud create -g myResourceGroup -n myPrivateCloudName --location eastus --cluster-size 3 --network-block xx.xx.xx.xx/22 --sku AV36
   ```
