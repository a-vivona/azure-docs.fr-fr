---
title: 'Azure Resource Manager : Créer une instance gérée SQL Azure'
description: Découvrez comment créer une instance managée Azure SQL Managed Instance à l’aide d’un modèle Azure Resource Manager.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: subject-armqs, devx-track-azurepowershell
ms.devlang: ''
ms.topic: quickstart
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: mathoma
ms.date: 06/22/2020
ms.openlocfilehash: 882613ef5b27b7021c61cabfabe3f37c3c842bd6
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "121722823"
---
# <a name="quickstart-create-an-azure-sql-managed-instance-using-an-arm-template"></a>Démarrage rapide : Créer une instance managée Azure SQL Managed Instance à l’aide d’un modèle Resource Manager

Ce guide de démarrage rapide porte essentiellement sur le déploiement d’un modèle Azure Resource Manager (modèle ARM) en vue de créer une instance managée Azure SQL Managed Instance et un réseau virtuel. [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md) est une base de données cloud intelligente, complètement managée et évolutive, présentant une parité de fonctionnalités quasi totale avec le moteur de base de données SQL Server.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.sql%2Fsqlmi-new-vnet%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/).

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/sqlmi-new-vnet/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.sql/sqlmi-new-vnet/azuredeploy.json":::

Ces ressources Azure sont définies dans le modèle :

- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.Network/networkSecurityGroups)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.Network/routeTables)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.Network/virtualNetworks)
- [**Microsoft.Sql/managedinstances**](/azure/templates/microsoft.sql/managedinstances)

Vous trouverez d’autres exemples de modèles dans [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sql&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

Sélectionnez **Essayer** à partir du bloc de code PowerShell suivant pour ouvrir Azure Cloud Shell.

> [!IMPORTANT]
> Le déploiement d’une instance managée est une opération longue. Le déploiement de la première instance dans le sous-réseau prend généralement plus de temps que le déploiement dans un sous-réseau avec des instances managées existantes. Pour plus d’informations sur les temps de provisionnement moyens, consultez [Opérations de gestion des instances managées SQL](management-operations-overview.md#duration).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.sql/sqlmi-new-vnet/azuredeploy.json"

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
read -p "Enter a project name that is used for generating resource names:" projectName &&
read -p "Enter the location (i.e. centralus):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.sql/sqlmi-new-vnet/azuredeploy.json" &&
resourceGroupName="${projectName}rg" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

---

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Accédez au [Portail Azure](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups) et vérifiez que l’instance managée se trouve dans votre groupe de ressources sélectionné. Étant donné que la création d’une instance managée peut prendre du temps, vous devrez peut-être vérifier le lien **Déploiements** dans la page **Vue d’ensemble** de votre groupe de ressources.

- Pour suivre un guide de démarrage rapide montrant comment se connecter à SQL Managed Instance à partir d’une machine virtuelle Azure, consultez [Configurer une connexion de machine virtuelle Azure](connect-vm-instance-configure.md).
- Pour suivre un guide de démarrage rapide montrant comment se connecter à SQL Managed Instance à partir d’un ordinateur client local avec une connexion point à site, consultez [Configurer une connexion point à site](point-to-site-p2s-configure.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Conservez l’instance managée si vous souhaitez faire les [étapes suivantes](#next-steps), mais supprimez l’instance managée et les ressources associées au terme des tutoriels supplémentaires. Après la suppression d’une instance managée, consultez [Supprimer un sous-réseau après avoir supprimé une instance managée](virtual-cluster-delete.md).


Pour supprimer le groupe de ressources :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer une machine virtuelle Azure pour qu’elle se connecte à Azure SQL Managed Instance](connect-vm-instance-configure.md)
