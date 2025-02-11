---
title: Créer un registre de schémas Azure Event Hubs
description: Cet article explique comment créer un registre de schémas dans un espace de noms Azure Event Hubs.
ms.topic: how-to
ms.date: 06/01/2021
ms.custom: references_regions
ms.openlocfilehash: 1155ef818cf6c82d3159c146006b16d811efeac8
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/02/2021
ms.locfileid: "110788398"
---
# <a name="create-an-azure-event-hubs-schema-registry-preview"></a>Créer un registre de schémas Azure Event Hubs (préversion)
Cet article explique comment créer un groupe de schémas avec des schémas dans un registre de schémas hébergé par Azure Event Hubs. Pour obtenir une vue d’ensemble de la fonctionnalité Registre de schémas d’Azure Event Hubs, consultez [Azure Schema Registry dans Event Hubs](schema-registry-overview.md).

> [!NOTE]
> - La fonctionnalité **Registre de schémas** est actuellement en **préversion** et n’est pas recommandée pour les charges de travail de production.
> - La fonctionnalité n’est pas disponible au niveau **de base**.
> - Si l'instance d'Event Hub se trouve dans un **réseau virtuel**, vous ne pouvez pas créer de schémas sur le portail Azure, sauf si vous accédez au portail à partir d'une machine virtuelle du même réseau virtuel. 

## <a name="prerequisites"></a>Prérequis
[Créez un espace de noms Event Hubs](event-hubs-create.md#create-an-event-hubs-namespace). Vous pouvez également utiliser un espace de noms existant. 

## <a name="create-a-schema-group"></a>Créer un groupe de schémas
1. Accédez à la page **Espace de noms Event Hubs**. 
1. Dans le menu de gauche, sélectionnez **Registre de schémas**. Pour créer un groupe de schémas, sélectionnez **+ Groupe de schémas** dans la barre d’outils. 

    :::image type="content" source="./media/create-schema-registry/namespace-page.png" alt-text="Capture d’écran montrant la page Registre de schémas dans le portail Azure":::
1. Dans la page **Créer un groupe de schémas**, procédez comme suit :
    1. Entrez un **nom** pour le groupe de schémas.
    1. Pour **Type de sérialisation**, choisissez un format de sérialisation qui s’applique à tous les schémas du groupe de schémas. **Avro** étant actuellement le seul type pris en charge, sélectionnez **Avro**. 
    1. Sélectionnez un **mode de compatibilité** pour tous les schémas du groupe. Pour **Avro**, les modes de compatibilité ascendante et descendante sont pris en charge. 
    1. Sélectionnez ensuite **Créer** pour créer le groupe de schémas. 
    
        :::image type="content" source="./media/create-schema-registry/create-schema-group-page.png" alt-text="Image montrant la page de création d’un groupe de schémas":::
1. Sélectionnez le nom du **groupe de schémas** dans la liste des groupes de schémas.

    :::image type="content" source="./media/create-schema-registry/select-schema-group.png" alt-text="Image montrant un groupe de schémas dans la liste sélectionnée.":::    
1. La page **Groupe de schémas** s’affiche pour le groupe.

    :::image type="content" source="./media/create-schema-registry/schema-group-page.png" alt-text="Image montrant la page Groupe de schémas":::
    

## <a name="add-a-schema-to-the-schema-group"></a>Ajouter un schéma au groupe de schémas
Dans cette section, vous allez ajouter un schéma au groupe de schémas à l’aide du portail Azure. 

1. Dans la page **Groupe de schémas**, sélectionnez **+ Schéma** dans la barre d’outils. 
1. Dans la page **Créer un schéma**, procédez comme suit :
    1. Entrez un **nom** pour le schéma.
    1. Entrez le **schéma** suivant dans la zone de texte. Vous pouvez également sélectionner un fichier avec le schéma.
    
        ```json
        {
            "type": "record",
            "name": "AvroUser",
            "namespace": "com.azure.schemaregistry.samples",
            "fields": [
                {
                    "name": "name",
                    "type": "string"
                },
                {
                    "name": "favoriteNumber",
                    "type": "int"
                }
            ]
        }
        ```
    1. Sélectionnez **Create** (Créer). 
1. Sélectionnez le **schéma** dans la liste des schémas. 

    :::image type="content" source="./media/create-schema-registry/select-schema.png" alt-text="Image montrant le schéma sélectionné.":::
1. La page **Vue d’ensemble du schéma** s’affiche pour le schéma. 

    :::image type="content" source="./media/create-schema-registry/schema-overview-page.png" alt-text="Image montrant la page Vue d’ensemble du schéma":::    
1. S’il existe plusieurs versions d’un schéma, vous les voyez dans la liste déroulante **Versions**. Sélectionnez une version pour passer à ce schéma de version. 

## <a name="create-a-new-version-of-schema"></a>Créer une nouvelle version du schéma

1. Mettez à jour le schéma dans la zone de texte, puis sélectionnez **Valider**. Dans l’exemple suivant, un nouveau champ `id` a été ajouté au schéma. 

    :::image type="content" source="./media/create-schema-registry/update-schema.png" alt-text="Image montrant la page Mettre à jour le schéma":::    
    
1. Vérifiez l’état de validation et les modifications, puis sélectionnez **Enregistrer**. 

    :::image type="content" source="./media/create-schema-registry/compare-save-schema.png" alt-text="Image montrant la page Vue d’ensemble qui affiche l’état de validation, les modifications et l’enregistrement":::     
1. Vous pouvez voir que `2` est sélectionné pour la **version** sur la page **Vue d’ensemble du schéma**. 

    :::image type="content" source="./media/create-schema-registry/new-version.png" alt-text="Image montrant la nouvelle version du schéma":::    
1. Sélectionnez `1` pour voir la version 1 du schéma. 


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le registre de schémas, consultez [Azure Schema Registry dans Event Hubs](schema-registry-overview.md).

