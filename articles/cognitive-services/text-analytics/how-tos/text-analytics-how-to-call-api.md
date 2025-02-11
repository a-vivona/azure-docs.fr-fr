---
title: Appeler l’API Text Analytics
titleSuffix: Azure Cognitive Services
description: Cet article explique comment appeler l’API REST Analyse de texte Azure Cognitive Services et Postman.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 06/10/2021
ms.author: aahi
ms.openlocfilehash: b9e18bb9bf313ce2bbf1441b319b841a3153f38b
ms.sourcegitcommit: ddac53ddc870643585f4a1f6dc24e13db25a6ed6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/18/2021
ms.locfileid: "122535075"
---
# <a name="how-to-call-the-text-analytics-rest-api"></a>Comment appeler l’API REST Analyse de texte

Dans cet article, nous utilisons l’API REST Analyse de texte et [Postman](https://www.postman.com/downloads/) pour présenter des concepts clés. L’API fournit plusieurs points de terminaison synchrones et asynchrones pour l’utilisation des fonctionnalités du service. 

## <a name="create-a-text-analytics-resource"></a>Créer une ressource Analyse de texte

> [!NOTE]
> * Si vous souhaitez utiliser les points de terminaison `/analyze` ou `/health`, vous aurez besoin d’une ressource d’Analyse de texte utilisant un [niveau de tarification](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) Standard (S). Le point de terminaison `/analyze` est inclus dans votre [niveau tarifaire](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/).

Avant d’utiliser l’API Analyse de texte, vous devrez créer une ressource Azure avec une clé et un point de terminaison pour vos applications. 

1.  Tout d’abord, accédez au [portail Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) et créez une ressource d’Analyse de texte si vous n’en avez pas encore. Choisissez un [niveau tarifaire](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/).

2.  Sélectionnez la région que votre point de terminaison doit utiliser.

3.  Créez la ressource Analyse de texte et accédez à la section « Clés et point de terminaison » sous Gestion des ressources dans la partie gauche de la page. Copiez la clé à utiliser ultérieurement lors de l’appel des API. Vous l’ajouterez plus tard en tant que valeur pour l’en-tête `Ocp-Apim-Subscription-Key`.

4. Pour vérifier le nombre d’enregistrements de texte qui ont été envoyés à l’aide de votre ressource Analyse de texte :

    1. Accédez à votre ressource Analyse de texte dans le portail Azure. 
    2. Cliquez sur **Mesures**, situées sous **Surveillance** dans le menu de navigation de gauche. 
    3. Sélectionnez *Enregistrements de textes traités* dans la zone de liste déroulante pour **Métrique**.
    
Un enregistrement texte est une unité de texte d’entrée de 1 000 caractères au maximum.  Par exemple, 1 500 caractères envoyés en texte d’entrée comptent pour deux enregistrements texte.

## <a name="change-your-pricing-tier"></a>Changer votre niveau tarifaire 

Si vous disposez d’une ressource Analyse de texte avec un niveau tarifaire compris entre S0 et S4, vous devez la mettre à jour pour utiliser le [niveau tarifaire](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) Standard (S). Les niveaux tarifaires S0 à S4 vont être supprimés. Pour mettre à jour le tarif de votre ressource :

1. Accédez à votre ressource Analyse de texte dans le [portail Azure](https://portal.azure.com/).
2. Dans le menu de navigation de gauche, sélectionnez **Niveau tarifaire**. Il se trouve sous **GESTION DES RESSOURCES**. 
3. Choisissez le niveau tarifaire Standard (S). Puis cliquez sur **Sélectionner**.

Vous pouvez également créer une ressource Analyse de texte avec le niveau tarifaire Standard (S), et migrer vos applications pour utiliser les informations d’identification de la nouvelle ressource. 

## <a name="using-the-api-synchronously"></a>Utilisation de l’API de manière asynchrone

Vous pouvez appeler l’API Analyse de texte de façon synchrone (pour des scénarios à faible latence). Vous devez appeler chaque API (fonctionnalité) séparément lors de l’utilisation de l’API synchrone. Si vous devez appeler plusieurs fonctionnalités, consultez la section ci-dessous pour savoir comment appeler l’API Analyse de texte de façon asynchrone. 

## <a name="using-the-api-asynchronously"></a>Utilisation de l’API de manière asynchrone

L’API Analyse de texte v3.1 fournit deux points de terminaison asynchrones : 

* Le point de terminaison `/analyze` pour l’Analyse de texte vous permet d’analyser le même jeu de documents texte avec plusieurs fonctionnalités d’analyse de texte en un seul appel d’API. Auparavant, pour utiliser plusieurs fonctionnalités, vous deviez effectuer des appels d’API distincts pour chaque opération. Tenez compte de cette fonctionnalité lorsque vous devez analyser de grands ensembles de documents avec plusieurs fonctionnalités d’Analyse de texte.

* Le point de terminaison `/health` pour l’Analyse de texte en lien avec la santé, qui peut extraire et étiqueter des informations médicales pertinentes à partir de documents cliniques.  

Consultez le tableau ci-dessous pour savoir quelles fonctionnalités vous pouvez utiliser de manière asynchrone. Notez que vous ne pouvez appeler que quelques fonctionnalités à partir du point de terminaison `/analyze`. 

| Fonctionnalité | Synchrone | Asynchrone |
|--|--|--|
| Détection de la langue | ✔ |  |
| analyse de sentiments | ✔ | ✔* |
| Exploration des opinions | ✔ | ✔* |
| Extraction d’expressions clés | ✔ | ✔* |
| Reconnaissance d’entité nommée (y compris PII et PHI) | ✔ | ✔* |
| Liaison d’entités | ✔ | ✔* |
| Analyse de texte pour la santé (conteneur) | ✔ |  |
| Analyse de texte pour la santé (API) |  | ✔  |
| Synthèse de texte |  | ✔  |

`*` - Appelé de façon asynchrone via le point de terminaison `/analyze`.


[!INCLUDE [text-analytics-api-references](../includes/text-analytics-api-references.md)]

[!INCLUDE [text-analytics-character-limits](../includes/character-limits.md)]

<a name="json-schema"></a>

## <a name="api-request-formats"></a>Formats de demande API

Vous pouvez envoyer des appels synchrones et asynchrones à l’API Analyse de texte.

#### <a name="synchronous"></a>[Synchrone](#tab/synchronous)

### <a name="synchronous-requests"></a>Demandes synchrones

Le format des demandes d’API est le même pour toutes les opérations synchrones. Les documents sont envoyés dans un objet JSON sous forme de texte brut non structuré. XML n’est pas pris en charge. Le schéma JSON se compose des éléments décrits ci-dessous.

| Élément | Valeurs valides | Requis ? | Usage |
|---------|--------------|-----------|-------|
|`id` |Les données dont de type chaîne mais, dans la pratique, les ID de document tendent à être des entiers. | Obligatoire | Le système utilise les ID que vous fournissez pour structurer la sortie. Des codes langue, phrases clés et scores de sentiment sont générés pour chaque ID dans la demande.|
|`text` | Texte brut non structuré, jusqu’à 5 120 caractères. | Obligatoire | Pour la détection de la langue, le texte peut être exprimé dans toute langue. Pour l’analyse des sentiments, l’extraction de phrases clés et l’identification d’entité, le texte doit être dans une [langue prise en charge](../language-support.md). |
|`language` | Code [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) de 2 caractères d’une [langue prise en charge](../language-support.md) | Variable | Obligatoire pour l’analyse des sentiments, l’extraction de phrases clés et la liaison d’entité. Facultatif pour la détection de la langue. Son omission ne génère aucune erreur, mais affaiblit l’analyse. Le code de langue doit correspondre au `text` que vous fournissez. |

Voici un exemple de demande API pour les points de terminaison d’Analyse de texte synchrone. 

```json
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Sample text to be sent to the text analytics api."
    }
  ]
}
```

#### <a name="asynchronous"></a>[Asynchrone](#tab/asynchronous)

### <a name="asynchronous-requests-to-the-analyze-endpoint"></a>Demandes asynchrones adressées au point de terminaison `/analyze`

> [!NOTE]
> La dernière préversion de la bibliothèque de client d’Analyse de texte vous permet d’appeler des opérations d’Analyse asynchrone en utilisant un objet client. Vous trouverez des exemples sur GitHub :
* [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
* [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
* [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)

Le point de terminaison `/analyze` vous permet de choisir les fonctionnalités d’Analyse de texte prises en charge que vous souhaitez utiliser dans un appel d’API. Ce point de terminaison prend actuellement en charge les aspects suivants :

* Extraction d’expressions clés 
* Reconnaissance d’entité nommée (y compris PII et PHI)
* Liaison d’entités
* Analyse des sentiments
* Exploration des opinions
* Synthèse de texte

| Élément | Valeurs valides | Requis ? | Usage |
|---------|--------------|-----------|-------|
|`displayName` | String | Facultatif | Utilisé comme nom d’affichage pour l’identificateur unique du travail.|
|`analysisInput` | Inclut le champ `documents` en dessous | Obligatoire | Contient les informations relatives aux documents que vous souhaitez envoyer. |
|`documents` | Inclut les champs `id` et `text` en dessous | Obligatoire | Contient des informations pour chaque document envoyé, ainsi que le texte brut du document. |
|`id` | String | Obligatoire | Les ID que vous fournissez sont utilisés pour structurer la sortie. |
|`text` | Texte brut non structuré, jusqu’à 125 000 caractères. | Obligatoire | Doit être en anglais, qui est la seule langue actuellement prise en charge. |
|`tasks` | Inclut les fonctionnalités d’Analyse de texte suivantes : `entityRecognitionTasks`, `entityLinkingTasks`, `keyPhraseExtractionTasks`, `entityRecognitionPiiTasks`, `extractiveSummarizationTasks` ou `sentimentAnalysisTasks`. | Obligatoire | Une ou plusieurs fonctionnalités d’Analyse de texte que vous souhaitez utiliser. Notez que `entityRecognitionPiiTasks` a un paramètre `domain` facultatif qui peut avoir la valeur `pii` ou `phi` et le `piiCategories` pour la détection de types d’entité sélectionnés. Si le paramètre `domain` n’est pas spécifié, la valeur système par défaut est `pii`. De même, `sentimentAnalysisTasks` a le paramètre booléen `opinionMining` qui permet d’inclure les résultats de l’exploration des opinions dans la sortie de l’Analyse des sentiments. |
|`parameters` | Inclut les champs `model-version` et `stringIndexType` en dessous | Obligatoire | Ce champ est inclus dans les tâches de fonctionnalités au-dessus que vous choisissez. Ils contiennent des informations sur la version du modèle que vous souhaitez utiliser et sur le type d’index. |
|`model-version` | String | Obligatoire | Spécifiez la version du modèle appelée que vous souhaitez utiliser.  |
|`stringIndexType` | String | Obligatoire | Spécifiez le décodeur de texte correspondant à votre environnement de programmation.  Les types pris en charge sont `textElement_v8` (par défaut), `unicodeCodePoint` et `utf16CodeUnit`. Pour plus d’informations, consultez l’article sur les [décalages de texte](../concepts/text-offsets.md#offsets-in-api-version-31).  |
|`domain` | String | Facultatif | S’applique uniquement en tant que paramètre à la tâche `entityRecognitionPiiTasks` et peut être défini sur `pii` ou `phi`. Si la valeur n’est pas spécifiée, par défaut, il s’agit de `pii`.  |

```json
{
    "displayName": "My Job",
    "analysisInput": {
        "documents": [
            {
                "id": "doc1",
                "text": "It's incredibly sunny outside! I'm so happy"
            },
            {
                "id": "doc2",
                "text": "Pike place market is my favorite Seattle attraction. The shops have very good food."
            }
        ]
    },
    "tasks": {
        "entityRecognitionTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "loggingOptOut": false
                }
            }
        ],
        "entityRecognitionPiiTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "loggingOptOut": true,
                    "domain": "phi",
                    "piiCategories":["default"]
                }
            }
        ],
        "entityLinkingTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "loggingOptOut": false
                }
            }
        ],
        "keyPhraseExtractionTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "loggingOptOut": false
                }
            }
        ],
        "sentimentAnalysisTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "loggingOptOut": false,
                    "opinionMining": true
                }
            }
        ]
    }
}

```

### <a name="asynchronous-requests-to-the-health-endpoint"></a>Demandes asynchrones adressées au point de terminaison `/health`

Le format des demandes d’API adressées à l’Analyse de texte pour l’API hébergée de santé est le même que pour son conteneur. Les documents sont envoyés dans un objet JSON sous forme de texte brut non structuré. XML n’est pas pris en charge. Le schéma JSON se compose des éléments décrits ci-dessous.  Renseignez et envoyez le [formulaire de demande Cognitive Services](https://aka.ms/csgate) pour demander l’accès à Analyse de texte pour la santé.

| Élément | Valeurs valides | Requis ? | Usage |
|---------|--------------|-----------|-------|
|`id` |Les données dont de type chaîne mais, dans la pratique, les ID de document tendent à être des entiers. | Obligatoire | Le système utilise les ID que vous fournissez pour structurer la sortie. |
|`text` | Texte brut non structuré, jusqu’à 5 120 caractères. | Obligatoire | Notez que seul du texte en anglais est actuellement pris en charge. |
|`language` | Code [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) de 2 caractères d’une [langue prise en charge](../language-support.md) | Obligatoire | Seule la langue `en` est actuellement prise en charge. |

Voici un exemple de demande API pour l’Analyse de texte pour des points de terminaison de santé. 

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Subject was administered 100mg remdesivir intravenously over a period of 120 min"
    }
  ]
}
```

---

>[!TIP]
> Pour plus d’informations sur les limites de débits et de taille pour l’envoi de données à l’API Analyse de texte, consultez l’article [Limites de données et de débit](../concepts/data-limits.md) .


## <a name="set-up-a-request"></a>Configurer une demande 

Dans Postman (ou un autre outil de test d’API web), ajoutez le point de terminaison pour la fonctionnalité que vous souhaitez utiliser. Utilisez le tableau en dessous pour rechercher le format de point de terminaison approprié, puis remplacez `<your-text-analytics-resource>` par le point de terminaison de votre ressource. Par exemple :

> [!TIP]
> Vous pouvez appeler la version 3.0 des points de terminaison synchrones ci-dessous en remplaçant `/v3.1` par `/v3.0/`.

`https://my-resource.cognitiveservices.azure.com/text/analytics/v3.1/languages`

#### <a name="synchronous"></a>[Synchrone](#tab/synchronous)

### <a name="endpoints-for-sending-synchronous-requests"></a>Points de terminaison pour l’envoi de demandes synchrones

| Fonctionnalité | Type de demande | Points de terminaison de ressource |
|--|--|--|
| Détection de la langue | POST | `<your-text-analytics-resource>/text/analytics/v3.1/languages` |
| Analyse des sentiments | POST | `<your-text-analytics-resource>/text/analytics/v3.1/sentiment` |
| Exploration des opinions | POST | `<your-text-analytics-resource>/text/analytics/v3.1/sentiment?opinionMining=true` |
| Extraction d’expressions clés | POST | `<your-text-analytics-resource>/text/analytics/v3.1/keyPhrases` |
| Reconnaissance d’entité nommée – Général | POST | `<your-text-analytics-resource>/text/analytics/v3.1/entities/recognition/general` |
| Reconnaissance d’entité nommée – PII | POST | `<your-text-analytics-resource>/text/analytics/v3.1/entities/recognition/pii` |
| Reconnaissance d’entité nommée – PHI | POST |  `<your-text-analytics-resource>/text/analytics/v3.1/entities/recognition/pii?domain=phi` |
| Liaison d’entités | POST | `<your-text-analytics-resource>/text/analytics/v3.1/entities/linking` |

#### <a name="asynchronous"></a>[Asynchrone](#tab/asynchronous)

### <a name="endpoints-for-sending-asynchronous-requests-to-the-analyze-endpoint"></a>Points de terminaison pour l’envoi de demandes asynchrones au point de terminaison `/analyze`

| Fonctionnalité | Type de demande | Points de terminaison de ressource |
|--|--|--|
| Envoyer un travail d’analyse | POST | `https://<your-text-analytics-resource>/text/analytics/v3.1/analyze` |
| Obtenir l’état et les résultats d’une analyse | GET | `https://<your-text-analytics-resource>/text/analytics/v3.1/analyze/jobs/<Operation-Location>` |

### <a name="endpoints-for-sending-asynchronous-requests-to-the-health-endpoint"></a>Points de terminaison pour l’envoi de demandes asynchrones au point de terminaison `/health`

| Fonctionnalité | Type de demande | Points de terminaison de ressource |
|--|--|--|
| Envoyer une Analyse de texte pour un travail sur la santé  | POST | `https://<your-text-analytics-resource>/text/analytics/v3.1/entities/health/jobs` |
| Obtenir l’état et les résultats d’un travail | GET | `https://<your-text-analytics-resource>/text/analytics/v3.1/entities/health/jobs/<Operation-Location>` |
| Annuler le travail | DELETE | `https://<your-text-analytics-resource>/text/analytics/v3.1/entities/health/jobs/<Operation-Location>` |

--- 

Une fois que vous avez votre point de terminaison, dans Postman (ou un autre outil de test d’API web) :

1. Choisissez le type de demande pour la fonctionnalité que vous souhaitez utiliser.
2. Collez le point de terminaison de l’opération appropriée souhaitée à partir du tableau ci-dessus.
3. Définissez les trois en-têtes de demande :

   + `Ocp-Apim-Subscription-Key` : votre clé d’accès, obtenue à partir du portail Azure
   + `Content-Type` : application/json
   + `Accept` : application/json

    Si vous utilisez Postman, votre demande doit ressembler à la capture d’écran suivante, en supposant un point de terminaison `/keyPhrases`.
    
    ![Capture d’écran de demande avec point de terminaison et en-têtes](../media/postman-request-keyphrase-1.png)
    
4. Choisissez **brut** pour le format du **Corps**
    
    ![Capture d’écran de demande avec paramètres du corps](../media/postman-request-body-raw.png)

5. Collez des documents JSON dans un format valide. Utilisez les exemples de la section **format de demande API** ci-dessus et, pour plus d’informations et d’exemples, consultez les rubriques ci-dessous :

      + [Détection de la langue](text-analytics-how-to-language-detection.md)
      + [Extraction de phrases clés](text-analytics-how-to-keyword-extraction.md)
      + [Analyse des sentiments](text-analytics-how-to-sentiment-analysis.md)
      + [Reconnaissance d’entités](text-analytics-how-to-entity-linking.md)

## <a name="send-the-request"></a>Envoyer la demande

Envoyez la demande API. Si vous avez appelé un point de terminaison synchrone, la réponse s’affiche immédiatement, sous la forme d’un document JSON unique, avec un élément pour chaque ID de document fourni dans la demande.

Si vous avez appelé les points de terminaison asynchrones `/analyze` ou `/health`, vérifiez que vous avez reçu un code de réponse 202. Vous devrez obtenir la réponse pour afficher les résultats :

1. Dans la réponse de l’API, recherchez `Operation-Location` dans l’en-tête, qui identifie le travail que vous avez envoyé à l’API. 
2. Créez une demande GET pour le point de terminaison que vous avez utilisé. Consultez le [tableau ci-dessus](#set-up-a-request) pour le format du point de terminaison, ainsi que la [documentation de référence de l’API](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/AnalyzeStatus). Par exemple :

    `https://my-resource.cognitiveservices.azure.com/text/analytics/v3.1/analyze/jobs/<Operation-Location>`

3. Ajoutez `Operation-Location` à la demande.

4. La réponse sera un document JSON unique, avec un élément pour chaque ID de document fourni dans la demande.

Notez que pour les opérations asynchrones `/analyze` ou `/health`, les résultats de la demande GET à l’étape 2 ci-dessus sont disponibles pendant 24 heures à partir du moment où le travail a été créé.  Ce délai est indiqué par la valeur `expirationDateTime` dans la réponse GET.  Après cette période, les résultats sont purgés et ne sont plus disponibles pour récupération.    

## <a name="example-api-responses"></a>Exemples de réponses de l’API
 
# <a name="synchronous"></a>[Synchrone](#tab/synchronous)

### <a name="example-responses-for-synchronous-operation"></a>Exemples de réponses pour une opération synchrone

Les réponses de point de terminaison synchrone varient en fonction du point de terminaison que vous utilisez. Pour obtenir des exemples de réponses, consultez les articles suivants.

+ [Détection de la langue](text-analytics-how-to-language-detection.md#step-3-view-the-results)
+ [Extraction de phrases clés](text-analytics-how-to-keyword-extraction.md#step-3-view-results)
+ [Analyse des sentiments](text-analytics-how-to-sentiment-analysis.md#view-the-results)
+ [Reconnaissance d’entités](text-analytics-how-to-entity-linking.md#view-results)

# <a name="asynchronous"></a>[Asynchrone](#tab/asynchronous)

### <a name="example-responses-for-asynchronous-operations"></a>Exemples de réponses pour des opérations asynchrones

En cas de réussite, la demande GET adressée au point de terminaison `/analyze` retourne un objet contenant les tâches affectées. Par exemple, `keyPhraseExtractionTasks`. Ces tâches contiennent l’objet réponse de la fonctionnalité d’Analyse de texte appropriée. Consultez les articles suivants pour plus d’informations.

+ [Extraction de phrases clés](text-analytics-how-to-keyword-extraction.md#step-3-view-results)
+ [Analyse des sentiments](text-analytics-how-to-sentiment-analysis.md#view-the-results)
+ [Reconnaissance d’entités](text-analytics-how-to-entity-linking.md#view-results)
+ [Analyse de texte pour la santé](text-analytics-for-health.md#hosted-asynchronous-web-api-response)

--- 

## <a name="see-also"></a>Voir aussi

* [Vue d’ensemble d’Analyse de texte](../overview.md)
* [Versions du modèle](../concepts/model-versioning.md)
* [Questions fréquentes (FAQ)](../text-analytics-resource-faq.yml)</br>
* [Page produit d’Analyse de texte](//go.microsoft.com/fwlink/?LinkID=759712)
* [Utilisation de la bibliothèque cliente Analyse de texte](../quickstarts/client-libraries-rest-api.md)
* [Nouveautés](../whats-new.md)
