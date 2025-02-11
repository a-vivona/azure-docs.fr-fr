---
title: 'Langage de requête Azure Cosmos DB : fonctions de chaîne'
description: Découvrez les fonctions système SQL de chaîne dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/26/2021
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: c3e9948fa6dd32198cd2584589a251b0c4399817
ms.sourcegitcommit: 2d412ea97cad0a2f66c434794429ea80da9d65aa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/14/2021
ms.locfileid: "122533206"
---
# <a name="string-functions-azure-cosmos-db"></a>Fonctions de chaîne (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](../includes/appliesto-sql-api.md)]

Les fonctions de chaîne vous permettent d’effectuer des opérations sur des chaînes dans Azure Cosmos DB.

## <a name="functions"></a>Fonctions

Les fonctions scalaires ci-dessous effectuent une opération sur une valeur d’entrée de chaîne et retournent une valeur de type chaîne, une valeur numérique ou une valeur booléenne. La colonne **Utilisation de l’index** suppose, le cas échéant, que vous comparez la fonction système de chaîne à une autre valeur avec un filtre d’égalité.

| Fonction système                                 | Utilisation de l’index        | [Utilisation de l’index dans les requêtes avec les fonctions d’agrégation scalaires](../index-overview.md#index-utilization-for-scalar-aggregate-functions) | Notes                                                      |
| ----------------------------------------------- | ------------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| [CONCAT](sql-query-concat.md)                   | Analyse complète          | Analyse complète                                              |                                                              |
| [CONTAINS](sql-query-contains.md)               | Analyse complète de l'index    | Analyse complète                                              |                                                              |
| [ENDSWITH](sql-query-endswith.md)               | Analyse complète de l'index    | Analyse complète                                              |                                                              |
| [INDEX_OF](sql-query-index-of.md)               | Analyse complète          | Analyse complète                                              |                                                              |
| [LEFT](sql-query-left.md)                       | Analyse précise de l'index | Analyse précise de l'index                                     |                                                              |
| [LENGTH](sql-query-length.md)                   | Analyse complète          | Analyse complète                                              |                                                              |
| [LOWER](sql-query-lower.md)                     | Analyse complète          | Analyse complète                                              |                                                              |
| [LTRIM](sql-query-ltrim.md)                     | Analyse complète          | Analyse complète                                              |                                                              |
| [REGEXMATCH](sql-query-regexmatch.md)           | Analyse complète de l'index    | Analyse complète                                              |                                                              |
| [REPLACE](sql-query-replace.md)                 | Analyse complète          | Analyse complète                                              |                                                              |
| [REPLICATE](sql-query-replicate.md)             | Analyse complète          | Analyse complète                                              |                                                              |
| [REVERSE](sql-query-reverse.md)                 | Analyse complète          | Analyse complète                                              |                                                              |
| [RIGHT](sql-query-right.md)                     | Analyse complète          | Analyse complète                                              |                                                              |
| [RTRIM](sql-query-rtrim.md)                     | Analyse complète          | Analyse complète                                              |                                                              |
| [STARTSWITH](sql-query-startswith.md)           | Analyse précise de l'index | Analyse précise de l'index                                     | Il s'agira d'une analyse développée de l'index si l'option Non respect de la casse est vraie. |
| [STRINGEQUALS](sql-query-stringequals.md)       | Recherche dans l'index         | Recherche dans l'index                                             | Il s'agira d'une analyse développée de l'index si l'option Non respect de la casse est vraie. |
| [StringToArray](sql-query-stringtoarray.md)     | Analyse complète          | Analyse complète                                              |                                                              |
| [StringToBoolean](sql-query-stringtoboolean.md) | Analyse complète          | Analyse complète                                              |                                                              |
| [StringToNull](sql-query-stringtonull.md)       | Analyse complète          | Analyse complète                                              |                                                              |
| [StringToNumber](sql-query-stringtonumber.md)   | Analyse complète          | Analyse complète                                              |                                                              |

En savoir plus sur l'[utilisation de l'index](../index-overview.md#index-usage) dans Azure Cosmos DB.

## <a name="next-steps"></a>Étapes suivantes

- [Fonctions système Azure Cosmos DB](sql-query-system-functions.md)
- [Présentation d’Azure Cosmos DB](../introduction.md)
- [Fonctions définies par l’utilisateur](sql-query-udfs.md)
- [Agrégats](sql-query-aggregate-functions.md)
