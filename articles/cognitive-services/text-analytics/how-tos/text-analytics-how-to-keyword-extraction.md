---
title: Extraction de phrases clés à l’aide de l’API REST Analyse de texte
titleSuffix: Azure Cognitive Services
description: Comment extraire des phrases clés à l’aide de l’API REST Analyse de texte d’Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 08/04/2021
ms.author: aahi
ms.openlocfilehash: 6a7dca4fd7fe74515f7532b457ce9e1aaad65b57
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122563033"
---
# <a name="example-how-to-extract-key-phrases-using-text-analytics"></a>Exemple : Comment extraire des phrases clés avec l’Analyse de texte

L’[API Extraction de phrases clés](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/KeyPhrases) évalue un texte non structuré puis, pour chaque document JSON, retourne une liste d’expressions clés.

Cette fonctionnalité est utile si vous avez besoin d’identifier rapidement les principaux points d’une collection de documents. Par exemple, pour le texte d’entrée « The food was delicious and there were wonderful staff » (La nourriture était délicieuse et le personnel adorable), le service retourne les principaux points de discussion : « food » (nourriture) et « wonderful staff » (personnel adorable).

Pour en savoir plus, consultez [Langages pris en charge](../language-support.md).

> [!TIP]
> * Analyse de texte fournit également une image conteneur Docker basée sur Linux pour l’extraction de phrases clés. Vous pouvez ainsi [installer et exécuter le conteneur Analyse de texte](text-analytics-how-to-install-containers.md) à proximité de vos données.
> * Vous pouvez également utiliser cette fonctionnalité [de manière asynchrone](text-analytics-how-to-call-api.md) à l’aide du point de terminaison `/analyze`.

## <a name="preparation"></a>Préparation

L’extraction de phrases clés fonctionne mieux avec une plus grande quantité de texte. Contrairement à l’analyse des sentiments, qui fonctionne mieux sur des petites quantités de texte. Pour obtenir des résultats optimaux pour ces deux opérations, envisagez de restructurer les entrées en conséquence.

Vous devez disposer des documents JSON dans ce format : ID, texte, langue

La taille des documents doit être égale ou inférieure à 5 120 caractères par document, et vous pouvez avoir jusqu’à 10 éléments (ID) par collection. La collection est soumise dans le corps de la demande. L’exemple suivant illustre le contenu qui peut être soumis pour extraction d’expressions clés. 

Pour plus d’informations sur les objets de demande et de réponse, consultez [Comment appeler l’API Analyse de texte](text-analytics-how-to-call-api.md).  

### <a name="example-synchronous-request-object"></a>Exemple d’objet de demande synchrone


```json
    {
        "documents": [
            {
                "language": "en",
                "id": "1",
                "text": "We love this trail and make the trip every year. The views are breathtaking and well worth the hike!"
            },
            {
                "language": "en",
                "id": "2",
                "text": "Poorly marked trails! I thought we were goners. Worst hike ever."
            },
            {
                "language": "en",
                "id": "3",
                "text": "Everyone in my family liked the trail but thought it was too challenging for the less athletic among us. Not necessarily recommended for small children."
            },
            {
                "language": "en",
                "id": "4",
                "text": "It was foggy so we missed the spectacular views, but the trail was ok. Worth checking out if you are in the area."
            },
            {
                "language": "en",
                "id": "5",
                "text": "This is my favorite trail. It has beautiful views and many places to stop and rest"
            }
        ]
    }
```

### <a name="example-asynchronous-request-object"></a>Exemple d’objet de demande asynchrone

Depuis la version `v3.1`, vous pouvez envoyer des demandes NER de façon asynchrone à l’aide du point de terminaison `/analyze` .


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
                "text": "Pike place market is my favorite Seattle attraction."
            }
        ]
    },
    "tasks": {
        "keyPhraseExtractionTasks": [{
            "parameters": {
                "model-version": "latest"
            }
        }],
    }
}
```

## <a name="step-1-structure-the-request"></a>Étape 1 : Structurer la requête

Pour en savoir plus sur la définition de la requête, voir [Comment appeler l’API REST Analyse de texte](text-analytics-how-to-call-api.md). Les points suivants sont réaffirmés pour des raisons pratiques :

+ Créez une demande **POST**. Passez en revue la documentation de l’API pour cette requête : [API d’expressions clés](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/KeyPhrases).

+ Définissez le point de terminaison HTTP pour l’extraction de phrases clés à l’aide d’une ressource Analyse de texte sur Azure, ou avec un [conteneur Analyse de texte](text-analytics-how-to-install-containers.md) instancié. Si vous utilisez l’API de façon synchrone, vous devez inclure `/text/analytics/v3.1/keyPhrases` dans l’URL. Par exemple : `https://<your-custom-subdomain>.api.cognitiveservices.azure.com/text/analytics/v3.1/keyPhrases`.

+ Définissez un en-tête de requête pour inclure la [clé d’accès](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) des opérations Analyse de texte.

+ Dans le corps de la demande, fournissez la collection de documents JSON que vous avez préparée pour cette analyse.

> [!Tip]
> Utilisez [Postman](text-analytics-how-to-call-api.md) ou ouvrez la **console de test d’API** dans la [documentation](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/KeyPhrases) pour structurer une demande et la publier dans le service.

## <a name="step-2-post-the-request"></a>Étape 2 : Publier la requête

L’analyse est effectuée à la réception de la demande. Pour connaître la taille et le nombre de demandes que vous pouvez envoyer par minute et par seconde, consultez l’article [Limites de données](../concepts/data-limits.md).

Rappelez-vous que le service est sans état. Aucune donnée n’est stockée dans votre compte. Les résultats sont retournés immédiatement dans la réponse.

## <a name="step-3-view-results"></a>Étape 3 : Afficher les résultats

Toutes les demandes POST retournent une réponse au format JSON avec les ID et les propriétés détectées. L’ordre des expressions clés retournées est déterminé en interne par le modèle.

La sortie est retournée immédiatement. Vous pouvez diffuser en continu les résultats dans une application qui accepte le code JSON ou enregistrer la sortie dans un fichier sur le système local, puis l’importer dans une application qui vous permet de trier, rechercher et manipuler les données.

Voici un exemple de sortie pour l’extraction de phrases clés du point de terminaison v3.1 :

### <a name="synchronous-result"></a>Résultat synchrone

```json
{
    "documents": [
        {
            "id": "1",
            "keyPhrases": [
                "trail",
                "trip",
                "views",
                "hike"
            ],
            "warnings": []
        },
        {
            "id": "2",
            "keyPhrases": [
                "Worst hike",
                "trails"
            ],
            "warnings": []
        },
        {
            "id": "3",
            "keyPhrases": [
                "less athletic",
                "small children",
                "Everyone",
                "family",
                "trail"
            ],
            "warnings": []
        },
        {
            "id": "4",
            "keyPhrases": [
                "spectacular views",
                "trail",
                "area"
            ],
            "warnings": []
        },
        {
            "id": "5",
            "keyPhrases": [
                "favorite trail",
                "beautiful views",
                "many places"
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2021-06-01"
}
```
Comme indiqué, l’analyseur recherche et ignore les mots non essentiels, et conserve les termes ou expressions uniques qui semblent constituer le sujet ou l’objet d’une phrase.

### <a name="asynchronous-result"></a>Résultat asynchrone

Si vous utilisez le point de terminaison `/analyze` pour une opération asynchrone, vous obtenez une réponse contenant les tâches que vous avez envoyées à l’API.

```json
{
    "jobId": "fa813c9a-0d96-4a34-8e4f-a2a6824f9190",
    "lastUpdateDateTime": "2021-07-07T18:16:45Z",
    "createdDateTime": "2021-07-07T18:16:15Z",
    "expirationDateTime": "2021-07-08T18:16:15Z",
    "status": "succeeded",
    "errors": [],
    "displayName": "My Job",
    "tasks": {
        "completed": 1,
        "failed": 0,
        "inProgress": 0,
        "total": 1,
        "keyPhraseExtractionTasks": [
            {
                "lastUpdateDateTime": "2021-07-07T18:16:45.0623454Z",
                "taskName": "KeyPhraseExtraction_latest",
                "state": "succeeded",
                "results": {
                    "documents": [
                        {
                            "id": "doc1",
                            "keyPhrases": [],
                            "warnings": []
                        },
                        {
                            "id": "doc2",
                            "keyPhrases": [
                                "Pike place market",
                                "Seattle attraction",
                                "favorite"
                            ],
                            "warnings": []
                        }
                    ],
                    "errors": [],
                    "modelVersion": "2021-06-01"
                }
            }
        ]
    }
}
```


## <a name="summary"></a>Résumé

Dans cet article, vous avez vu les concepts et le flux de travail lié à l’extraction de phrases clés à l’aide de l’API Analyse de texte dans Cognitive Services. En résumé :

+ L’[API Extraction de phrases clés](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1/operations/KeyPhrases) est disponible pour les langues sélectionnées.
+ Les documents JSON figurant dans le corps de la demande incluent un ID, un texte et un code de langue.
+ La demande POST est envoyée à un point de terminaison `/keyphrases` ou `/analyze`, à l’aide d’une [clé d’accès et d’un point de terminaison](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) personnalisés valides pour votre abonnement.
+ La sortie de réponse, qui se compose de mots et de phrases clés pour chaque ID de document, peut être diffusée vers n’importe quelle application qui accepte JSON, y compris Microsoft Office Excel et Power BI, pour n’en citer que quelques-unes.

## <a name="see-also"></a>Voir aussi

 [Vue d’ensemble d’Analyse de texte](../overview.md)[Questions fréquentes (FAQ)](../text-analytics-resource-faq.yml)</br>
 [Page produit d’Analyse de texte](//go.microsoft.com/fwlink/?LinkID=759712)

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Analyse de texte](../overview.md)
* [Utilisation de la bibliothèque cliente Analyse de texte](../quickstarts/client-libraries-rest-api.md)
* [Nouveautés](../whats-new.md)
* [Versions du modèle](../concepts/model-versioning.md)
