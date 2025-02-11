---
title: API Obtenir tous les jeux de données
description: Utilisez cette API afin d’obtenir tous les jeux de données disponibles pour l’analytique de place de marché commerciale.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: smannepalle
ms.author: smannepalle
ms.reviewer: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 487c1b4ad58eb17fc90bb78f0dbc4346de8a62d3
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122563096"
---
# <a name="get-all-datasets-api"></a>API Obtenir tous les jeux de données

L’API Obtenir tous les jeux de données obtient tous les jeux de données disponibles. Les jeux de données listent les tables, les colonnes, les métriques et les intervalles de temps.

**Syntaxe de la requête**

| **Méthode** | **URI de demande** |
| --- | --- |
| GET | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledDataset?datasetName={Dataset Name}` |

**En-tête de requête**

| **En-tête** | **Type** | **Description** |
| --- | --- | --- |
| Autorisation | string | Obligatoire. Le jeton d’accès Azure Active Directory (Azure AD) au format `Bearer <token>` |
| Content-Type | string | `Application/JSON` |

**Paramètre de chemin**

Aucun

**Paramètre de requête.**

| **Nom du paramètre** | **Type** | **Obligatoire** | **Description** |
| --- | --- | --- | --- |
| `datasetName` | string | Non | Filtrer pour afficher les détails d’un seul jeu de données |

**Charge utile de demande**

Aucun

**Glossaire**

Aucune

**Réponse**

La charge utile de la réponse est structurée comme suit :

Code de réponse : 200, 400, 401, 403, 404, 500

Exemple de charge utile de réponse :

```json
{
   "Value":[
      {
         "DatasetName ":"string",
         "SelectableColumns":[
            "string"
         ],
         "AvailableMetrics":[
            "string"
         ],
         "AvailableDateRanges ":[
            "string"
         ]
      }
   ],
   "TotalCount":int,
   "Message":"<Error Message>",
   "StatusCode": int
}
```

**Glossaire**

Ce tableau définit les éléments clés de la réponse :

| **Paramètre** | **Description** |
| --- | --- |
| `DatasetName` | Nom du jeu de données défini par cet objet de tableau |
| `SelectableColumns` | Colonnes brutes qui peuvent être spécifiées dans les colonnes sélectionnées |
| `AvailableMetrics` | Noms des colonnes d’agrégation/métriques qui peuvent être spécifiées dans les colonnes sélectionnées |
| `AvailableDateRanges` | Plage de dates qui peut être utilisée dans les requêtes de rapport pour le jeu de données |
| `NextLink` | Lien vers la page suivante si les données sont paginées |
| `TotalCount` | Nombre de jeux de données dans le tableau de valeurs |
| `StatusCode` | Code de résultat. Les valeurs possibles sont 200, 400, 401, 403, 500 |
