---
title: Gérer des sources de données dans Azure Purview (préversion)
description: Découvrez comment inscrire de nouvelles sources de données, gérer des collections de sources de données et afficher des sources dans Azure Purview (préversion).
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/25/2020
ms.openlocfilehash: c1e60ae792921ef4e218918f093001ee9947975d
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122531444"
---
# <a name="manage-data-sources-in-azure-purview-preview"></a>Gérer des sources de données dans Azure Purview (préversion)

Cet article explique comment inscrire de nouvelles sources de données, gérer des collections de sources de données et à afficher des sources dans Azure Purview (préversion).

## <a name="register-a-new-source"></a>Inscrire une nouvelle source

Suivez cette procédure pour inscrire une nouvelle source.

1. Ouvrez Purview Studio et sélectionnez la vignette **Inscrire des sources**.

   :::image type="content" source="media/manage-data-sources/purview-studio.png" alt-text="Azure Purview Studio":::

1. Sélectionnez **Inscrire**, puis sélectionnez un type de source. Cet exemple utilise Stockage Blob Azure. Sélectionnez **Continuer**.

   :::image type="content" source="media/manage-data-sources/select-source-type.png" alt-text="Sélectionner un type de source de données dans la page Inscrire les sources":::

2. Remplissez le formulaire sur la page **Inscrire des sources**. Sélectionnez un nom pour votre source et entrez les informations pertinentes. Si vous avez choisi **À partir d’un abonnement Azure** comme méthode de sélection de votre compte, les sources de votre abonnement s’affichent dans une liste déroulante. 

   :::image type="content" source="media/manage-data-sources/register-sources-form.png" alt-text="Formulaire pour les informations sur la source de données":::

3. Sélectionnez **Inscription**.

## <a name="view-sources"></a>Afficher les sources

Vous pouvez afficher toutes les sources inscrites sous l’onglet **Data Map** d’Azure Purview Studio. Il existe deux types d’affichages : la vue cartographique et le mode liste.

### <a name="map-view"></a>Affichage de la carte

Dans la vue cartographique, vous pouvez voir toutes vos sources et collections. L’image suivante contient une source Stockage Blob Azure. À partir de chaque vignette source, vous pouvez modifier la source, démarrer une nouvelle analyse ou afficher les détails de la source.

:::image type="content" source="media/manage-data-sources/map-view.png" alt-text="Vue Azure Purview du mappage des sources de données":::

### <a name="list-view"></a>Affichage Liste

En mode liste, vous pouvez voir une liste de sources pouvant être triée. Pointez sur la source pour faire apparaître les options permettant de la modifier, de la supprimer ou de commencer une nouvelle analyse.

:::image type="content" source="media/manage-data-sources/list-view.png" alt-text="Vue Azure Purview de la liste des sources de données":::

## <a name="manage-collections"></a>Gérer des regroupements

Vous pouvez regrouper vos sources de données dans des collections. Pour créer une collection, sélectionnez **+ Nouvelle collection** dans la page *Sources* d’Azure Purview Studio. Donnez un nom à la collection et sélectionnez *Aucun* en tant que parent. La nouvelle collection s’affiche dans la vue cartographique.

Pour ajouter des sources à une collection, sélectionnez le crayon **Modifier** sur la source et choisissez une collection dans le menu déroulant **Sélectionner une collection**.

Pour créer une hiérarchie de collections, assignez des collections de niveau supérieur en tant que parent aux collections de niveau inférieur. Dans l’image suivante, *Fabrikam* est un parent de la collection *Finance*, qui contient une source de données Stockage Blob Azure. Vous pouvez réduire ou développer les collections en cliquant sur le cercle attaché à la flèche entre les niveaux.

:::image type="content" source="media/manage-data-sources/collections.png" alt-text="Hiérarchie de collections dans Azure Purview Studio":::

Vous pouvez supprimer des sources d’une hiérarchie en sélectionnant *Aucun* pour le parent. Les sources sans parent sont regroupées dans une zone en pointillés dans la vue cartographique, sans flèches les reliant à un parent.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment inscrire et analyser diverses sources de données :

* [Azure Data Lake Storage Gen 2](register-scan-adls-gen2.md)
* [Locataire Power BI](register-scan-power-bi-tenant.md)
* [Azure SQL Database](register-scan-azure-sql-database.md)
