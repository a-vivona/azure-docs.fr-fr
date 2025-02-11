---
title: Connecter l’espace de travail Synapse à Azure Purview 
description: Connectez un compte Azure Purview à un espace de travail Synapse.
author: Jejiang
ms.service: synapse-analytics
ms.subservice: purview
ms.topic: quickstart
ms.date: 08/24/2021
ms.author: jejiang
ms.reviewer: jrasnick
ms.openlocfilehash: 5cf2149591776b66a2bb646c8de23e0669b31f25
ms.sourcegitcommit: 03f0db2e8d91219cf88852c1e500ae86552d8249
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2021
ms.locfileid: "123033842"
---
# <a name="quickstartconnect-a-synapse-workspace-to-an-azure-purview-account"></a>Démarrage rapide : Connectez un compte Azure Purview à un espace de travail Synapse

Dans ce guide de démarrage rapide, vous allez inscrire un compte Azure Purview auprès d’un espace de travail Synapse. Cette connexion vous permet de découvrir les ressources Azure Purview, d’interagir avec elles grâce aux fonctionnalités de Synapse et de transmettre des informations sur de lignage à Purview.

Vous pouvez effectuer les tâches suivantes dans Synapse :
- Utiliser la zone de recherche en haut pour rechercher des ressources Purview en fonction de mots clés 
- Comprendre les données en fonction des métadonnées, de la [traçabilité](../../purview/catalog-lineage-user-guide.md), des annotations 
- Connecter ces données à votre espace de travail avec des services liés ou des jeux de données d’intégration 
- Analyser ces jeux de données avec Synapse Apache Spark, Synapse SQL et Data Flow 
- Exécuter des pipelines et [envoyer des informations de lignage à Purview](../../purview/how-to-lineage-azure-synapse-analytics.md)

## <a name="prerequisites"></a>Prérequis 
- [Compte Azure Purview](../../purview/create-catalog-portal.md) 
- [Espace de travail Synapse](../quickstart-create-workspace.md) 

## <a name="permissions-for-connecting-an-azure-purview-account"></a>Autorisations pour la connexion d’un compte Azure Purview 

Pour connecter un compte Azure Purview à un espace de travail Synapse, vous devez disposer d’un rôle **Contributeur** dans l’espace de travail Synapse provenant de la gestion des identités et des accès (IAM) du portail Azure et d’un accès à ce compte Azure Purview. Pour plus d’informations, consultez [Autorisations Azure Purview](../../purview/catalog-permissions.md).

## <a name="connect-an-azure-purview-account"></a>Se connecter à un compte Azure Purview  

Suivez les étapes pour connecter un compte Azure Purview :

1. Accédez à  [https://web.azuresynapse.net](https://web.azuresynapse.net) et connectez-vous à votre espace de travail Synapse. 
2. Allez dans **Gérer** -> **Azure Purview**, sélectionnez **Connexion à un compte Purview**.
3. Vous pouvez choisir **Depuis un abonnement Azure** ou **Entrer manuellement**. **Depuis un abonnement Azure**, vous pouvez sélectionner le compte auquel vous avez accès.
4. Une fois connecté, vous pouvez voir le nom du compte Purview sous l’onglet **Compte Azure Purview**. 

Les informations de connexion de Purview sont stockées dans la ressource de l’espace de travail Synapse, comme ci-dessous. Pour établir la connexion par programme, vous pouvez mettre à jour l’espace de travail Synapse et ajouter les paramètres `purviewConfiguration`.

```json
{
    "name": "ContosoSynapseWorkspace",
    "type": "Microsoft.Synapse/workspaces",
    "location": "<region>",
    "properties": {
        ...
        "purviewConfiguration": {
            "purviewResourceId": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupname>/providers/Microsoft.Purview/accounts/<PurviewAccountName>"
        }
    },
    "identity": {...},
    ...
}
```

## <a name="set-up-authentication"></a>Configurer l’authentification

L’identité managée de l’espace de travail Synapse est utilisée pour authentifier les opérations d’envoi de lignage de l’espace de travail Synapse dans Purview.

- Pour un compte Purview créé **le 18 août 2021 ou après**, accordez l’identité managée de l’espace de travail Synapse **Curateur de données** à votre **collection racine** Purview. En savoir plus sur le [Contrôle d’accès dans Azure Purview](../../purview/catalog-permissions.md) et comment [Ajouter des rôles et restreindre l’accès par le biais de regroupements](../../purview/how-to-create-and-manage-collections.md#add-roles-and-restrict-access-through-collections).

    Lors de la connexion de l’espace de travail Synapse à Purview dans Synapse Studio, Synapse tente d’ajouter automatiquement une telle attribution de rôle. Si vous avez un rôle **Administrateurs de collection** sur le regroupement racine Purview, cette opération s’effectue avec succès.

- Pour un compte Purview créé **avant le 18 août 2021**, accordez à l’identité managée de l’espace de travail Synapse le rôle Azure [**Curateur de données Purview**](../../role-based-access-control/built-in-roles.md#purview-data-curator) intégré à votre compte Purview. En savoir plus sur le [Contrôle d’accès dans Azure Purview - Autorisations héritées](../../purview/catalog-permissions.md#legacy-permission-guide).

    Lors de la connexion de l’espace de travail Synapse à Purview dans Synapse Studio, Synapse tente d’ajouter automatiquement une telle attribution de rôle. Si vous avez le rôle intégré Azure **Propriétaire** ou **Administrateur de l’accès utilisateur** sur le compte Purview, cette opération est effectuée avec succès.

Vous pouvez voir l’avertissement ci-dessous si vous avez le privilège de lire les informations d’attribution de rôle Purview et que le rôle requis n’est pas accordé. Pour vous assurer que la connexion est correctement configurée pour le push de lignage de pipeline, accédez à votre compte Purview et vérifiez si le rôle **Curateur de données Purview** est accordé à l’identité managée de l’espace de travail Synapse. Si ce n’est pas le cas, ajoutez manuellement l’attribution de rôle.

:::image type="content" source="./media/register-purview-account-warning.png" alt-text="Capture d’écran de l’avertissement lors de l’inscription d’un compte Purview.":::

## <a name="report-lineage-to-azure-purview"></a>Rapporter la traçabilité à Azure Purview

Une fois que vous avez connecté l’espace de travail Synapse à un compte Purview, lorsque vous exécutez des pipelines, Synapse signale les informations de lignage au compte Purview. Pour connaître le détail des fonctionnalités prises en charge et une présentation complète, voir [Métadonnées et lignage à partir d’Azure Synapse Analytics](../../purview/how-to-lineage-azure-synapse-analytics.md).

## <a name="discover-and-explore-data-using-purview"></a>Découvrir et explorer des données avec Purview

Une fois que vous avez connecté l’espace de travail Synapse à un compte Purview, vous pouvez utiliser la barre de recherche au centre de l’espace de travail Synapse pour rechercher des données et effectuer des actions. Consultez [Découvrir, connecter et explorer des données dans Synapse en utilisant Azure Purview](how-to-discover-connect-analyze-azure-purview.md) pour en savoir plus.

## <a name="nextsteps"></a>Étapes suivantes 

[Découvrir, connecter et explorer des données dans Synapse en utilisant Azure Purview](how-to-discover-connect-analyze-azure-purview.md)

[Métadonnées et traçabilité depuis Azure Synapse Analytics](../../purview/how-to-lineage-azure-synapse-analytics.md)

[Inscrire et analyser des ressources Azure Synapse dans Azure Purview](../../purview/register-scan-azure-synapse-analytics.md)

[Tracer les données à partir de Power BI dans Azure Purview](../../purview/how-to-lineage-powerbi.md)

[Se connecter à Azure Data Share et Azure Purview](../../purview/how-to-link-azure-data-share.md)