---
title: Spécifier la version du modèle dans Analyse de texte v3
titleSuffix: Azure Cognitive Services
description: Découvrez comment spécifier le modèle d’API Analyse de texte utilisé sur vos données.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 06/17/2021
ms.author: aahi
ms.openlocfilehash: d1804505a73be2db8d7088caf74063381fa0ba33
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122524583"
---
# <a name="model-versioning-in-the-text-analytics-api"></a>Contrôle de version de modèle dans l’API Analyse de texte

La version 3 de l’API Analyse de texte vous permet de choisir la version de modèle utilisée sur vos données. Utilisez le paramètre facultatif `model-version` pour sélectionner la version du modèle dans vos demandes d’API. Par exemple : `<resource-url>/text/analytics/v3.0/sentiment?model-version=2020-04-01`. Si ce paramètre n’est pas spécifié, l’API a la dernière version stable par défaut. 

## <a name="available-versions"></a>Versions disponibles

Utilisez le tableau ci-dessous pour rechercher les versions de modèle prises en charge par chaque point de terminaison hébergé.


| Point de terminaison                        | Versions prises en charge                                     | version la plus récente |
|---------------------------------|--------------------------------------------------------|----------------|
| `/sentiment`                    | `2019-10-01`, `2020-04-01`                             | `2020-04-01`   |
| `/languages`                    | `2019-10-01`, `2020-07-01`, `2020-09-01`, `2021-01-05` | `2021-01-05`   |
| `/entities/linking`             | `2019-10-01`, `2020-02-01`                             | `2020-02-01`   |
| `/entities/recognition/general` | `2019-10-01`, `2020-02-01`, `2020-04-01`,`2021-01-15`,`2021-06-01`  | `2021-06-01`   |
| `/entities/recognition/pii`     | `2019-10-01`, `2020-02-01`, `2020-04-01`,`2020-07-01`, `2021-01-15`  | `2021-01-15`   |
| `/entities/health`              | `2021-05-15`                           | `2021-05-15`   |
| `/keyphrases`                   | `2019-10-01`, `2020-07-01`, `2021-06-01`  | `2021-06-01`   |


Pour plus d’informations sur les mises à jour de ces modèles, consultez la section [Nouveautés](../whats-new.md).

## <a name="extractive-summarization"></a>Résumé extractif

Le résumé extractif est disponible à partir de la `version 3.1-preview.1` par le biais du point de terminaison `analyze` asynchrone. 

La version de modèle actuelle est `2021-08-01`.

## <a name="text-analytics-for-health"></a>Analyse de texte pour la santé

Le conteneur [Analyse de texte pour l’intégrité](../how-tos/text-analytics-for-health.md) utilise une gestion de versions de modèle distinct des points de terminaison d’API ci-dessus.  Notez qu’une seule version de modèle est disponible par image conteneur.

| Point de terminaison                        | Étiquette de l’image conteneur                     | Version du modèle |
|---------------------------------|-----------------------------------------|---------------|
| `/entities/health`              | `3.0.016230002-onprem-amd64` ou la plus récente            | `2021-05-15`  |
| `/entities/health`              | `3.0.015370001-onprem-amd64`            | `2021-03-01`  |
| `/entities/health`              | `1.1.013530001-amd64-preview`           | `2020-09-03`  |
| `/entities/health`              | `1.1.013150001-amd64-preview`           | `2020-07-24`  |
| `/domains/health`               | `1.1.012640001-amd64-preview`           | `2020-05-08`  |
| `/domains/health`               | `1.1.012420001-amd64-preview`           | `2020-05-08`  |
| `/domains/health`               | `1.1.012070001-amd64-preview`           | `2020-04-16`  |


## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Analyse de texte](../overview.md)
* [Analyse des sentiments](../how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Reconnaissance d’entités](../how-tos/text-analytics-how-to-entity-linking.md)
