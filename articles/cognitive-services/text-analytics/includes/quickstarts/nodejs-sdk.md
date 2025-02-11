---
title: 'Démarrage rapide : Bibliothèque cliente Analyse de texte v3 pour Node.js | Microsoft Docs'
description: Découvrez comment utiliser la bibliothèque cliente Analyse de texte v3 pour Node.js.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 08/17/2021
ms.author: aahi
ms.reviewer: sumeh, assafi
ms.custom: devx-track-js
ms.openlocfilehash: c00334378ebbc2f399a9daa1c20ed4c47c84df65
ms.sourcegitcommit: 7854045df93e28949e79765a638ec86f83d28ebc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2021
ms.locfileid: "122864848"
---
<a name="HOLTop"></a>

# <a name="version-32-preview1"></a>[Version 3.2-preview.1](#tab/version-3-2)

[Documentation de référence de la version v3.2-preview](/javascript/api/overview/azure/ai-text-analytics-readme?preserve-view=true&view=azure-node-preview) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics) | [Package v3.2-preview (NPM)](https://www.npmjs.com/package/@azure/ai-text-analytics/v/5.2.0-beta.1) | [Exemples](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/textanalytics/ai-text-analytics/samples)


# <a name="version-31"></a>[Version 3.1](#tab/version-3-1)

[Documentation de référence de la version v3.1](/javascript/api/overview/azure/ai-text-analytics-readme?preserve-view=true&view=azure-node-latest) | [Code source de la bibliothèque v3](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics) | [Package v3.1 (NPM)](https://www.npmjs.com/package/@azure/ai-text-analytics/v/5.1.0) | [Exemples](https://github.com/Azure/azure-sdk-for-js/tree/main/sdk/textanalytics/ai-text-analytics/samples)

---

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* Version actuelle de [Node.js](https://nodejs.org/).
* [!INCLUDE [contributor-requirement](../../../includes/quickstarts/contributor-requirement.md)]
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="créez une ressource Analyse de texte"  target="_blank">Créer une ressource Analyse de texte </a> dans le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Analyse de texte. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.
* Pour utiliser la fonctionnalité d’analyse, vous aurez besoin d’une ressource Analyse de texte avec le niveau tarifaire Standard (S).

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-nodejs-application"></a>Création d’une application Node.js

Dans une fenêtre de console (telle que cmd, PowerShell ou bash), créez un répertoire pour votre application et accédez-y. 

```console
mkdir myapp 

cd myapp
```

Exécutez la commande `npm init` pour créer une application de nœud avec un fichier `package.json`. 

```console
npm init
```
### <a name="install-the-client-library"></a>Installer la bibliothèque de client

# <a name="version-32-preview1"></a>[Version 3.2-preview.1](#tab/version-3-2)

Installez le package NPM `@azure/ai-text-analytics` :

```console
npm install @azure/ai-text-analytics@5.2.0-beta.1
```

Fonctionnalités incluses dans cette version de l’API Analyse de texte :

* analyse de sentiments
* Exploration des opinions
* Détection de la langue
* Reconnaissance d’entité
* Liaison d’entités
* Reconnaissance des informations d’identification personnelle
* Extraction d’expressions clés
* Méthodes asynchrones
* Analyse de texte pour la santé
* Synthèse de texte

# <a name="version-31"></a>[Version 3.1](#tab/version-3-1)

Installez le package NPM `@azure/ai-text-analytics` :

```console
npm install --save @azure/ai-text-analytics@5.1.0
```

Fonctionnalités incluses dans cette version de l’API Analyse de texte :

* analyse de sentiments
* Exploration des opinions
* Détection de la langue
* Reconnaissance d’entité
* Liaison d’entités
* Reconnaissance des informations d’identification personnelle
* Extraction d’expressions clés
* Méthodes asynchrones
* Analyse de texte pour la santé

---

Le fichier `package.json` de votre application sera mis à jour avec les dépendances.
Créez un fichier nommé `index.js` et ajoutez ceci :

```javascript
"use strict";

const { TextAnalyticsClient, AzureKeyCredential } = require("@azure/ai-text-analytics");
```

Créez des variables pour le point de terminaison et la clé Azure de votre ressource.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```javascript
const key = '<paste-your-text-analytics-key-here>';
const endpoint = '<paste-your-text-analytics-endpoint-here>';
```

## <a name="object-model"></a>Modèle objet

Le client Analyse de texte est un objet `TextAnalyticsClient` qui s’authentifie auprès d’Azure à l’aide de votre clé. Le client fournit plusieurs méthodes pour analyser le texte, en tant que chaîne unique ou lot.

Le texte est envoyé à l’API sous la forme d’une liste de `documents`, qui sont des objets `dictionary` contenant une combinaison d’attributs `id`, `text` et `language` en fonction de la méthode utilisée. L’attribut `text` stocke le texte à analyser dans le `language` d’origine, et vous pouvez utiliser n’importe quelle valeur pour `id`. 

L’objet Response est une liste contenant les informations d’analyse de chaque document. 

[!INCLUDE [text-analytics-character-limits](../character-limits.md)]

## <a name="authenticate-the-client"></a>Authentifier le client

Créez un objet `TextAnalyticsClient` avec vos clé et point de terminaison comme paramètres.

```javascript
const textAnalyticsClient = new TextAnalyticsClient(endpoint,  new AzureKeyCredential(key));
```

## <a name="sentiment-analysis"></a>analyse de sentiments

Créez un tableau de chaînes contenant le document que vous souhaitez analyser. Appelez la méthode `analyzeSentiment()` du client et récupérez l’objet `SentimentBatchResult` retourné. Effectuez une itération dans la liste des résultats, puis affichez l’ID de chaque document et le sentiment au niveau du document avec les scores de confiance. Pour chaque document, le résultat contient le sentiment au niveau des phrases ainsi que les décalages, la longueur et les scores de confiance.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function sentimentAnalysis(client){

    const sentimentInput = [
        "I had the best day of my life. I wish you were there with me."
    ];
    const sentimentResult = await client.analyzeSentiment(sentimentInput);

    sentimentResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Sentiment: ${document.sentiment}`);
        console.log(`\tDocument Scores:`);
        console.log(`\t\tPositive: ${document.confidenceScores.positive.toFixed(2)} \tNegative: ${document.confidenceScores.negative.toFixed(2)} \tNeutral: ${document.confidenceScores.neutral.toFixed(2)}`);
        console.log(`\tSentences Sentiment(${document.sentences.length}):`);
        document.sentences.forEach(sentence => {
            console.log(`\t\tSentence sentiment: ${sentence.sentiment}`)
            console.log(`\t\tSentences Scores:`);
            console.log(`\t\tPositive: ${sentence.confidenceScores.positive.toFixed(2)} \tNegative: ${sentence.confidenceScores.negative.toFixed(2)} \tNeutral: ${sentence.confidenceScores.neutral.toFixed(2)}`);
        });
    });
}
sentimentAnalysis(textAnalyticsClient)
```

Exécutez votre code avec `node index.js` dans la fenêtre de votre console.

### <a name="output"></a>Output

```console
ID: 0
        Document Sentiment: positive
        Document Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
        Sentences Sentiment(2):
                Sentence sentiment: positive
                Sentences Scores:
                Positive: 1.00  Negative: 0.00  Neutral: 0.00
                Sentence sentiment: neutral
                Sentences Scores:
                Positive: 0.21  Negative: 0.02  Neutral: 0.77
```


## <a name="opinion-mining"></a>Exploration des opinions

Pour effectuer une analyse des sentiments avec exploration des opinions, créez un tableau de chaînes contenant le document à analyser. Appelez la méthode `analyzeSentiment()` du client en ajoutant l’indicateur facultatif `includeOpinionMining: true` et récupérez l’objet `SentimentBatchResult` retourné. Effectuez une itération dans la liste des résultats, puis affichez l’ID de chaque document et le sentiment au niveau du document avec les scores de confiance. Pour chaque document, le résultat contient non seulement le sentiment au niveau des phrases comme indiqué ci-dessus, mais également le sentiment au niveau de l’aspect et de l’opinion.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function sentimentAnalysisWithOpinionMining(client){

  const sentimentInput = [
    {
      text: "The food and service were unacceptable, but the concierge were nice",
      id: "0",
      language: "en"
    }
  ];
  const results = await client.analyzeSentiment(sentimentInput, { includeOpinionMining: true });

  for (let i = 0; i < results.length; i++) {
    const result = results[i];
    console.log(`- Document ${result.id}`);
    if (!result.error) {
      console.log(`\tDocument text: ${sentimentInput[i].text}`);
      console.log(`\tOverall Sentiment: ${result.sentiment}`);
      console.log("\tSentiment confidence scores:", result.confidenceScores);
      console.log("\tSentences");
      for (const { sentiment, confidenceScores, opinions } of result.sentences) {
        console.log(`\t- Sentence sentiment: ${sentiment}`);
        console.log("\t  Confidence scores:", confidenceScores);
        console.log("\t  Mined opinions");
        for (const { target, assessments } of opinions) {
          console.log(`\t\t- Target text: ${target.text}`);
          console.log(`\t\t  Target sentiment: ${target.sentiment}`);
          console.log("\t\t  Target confidence scores:", target.confidenceScores);
          console.log("\t\t  Target assessments");
          for (const { text, sentiment } of assessments) {
            console.log(`\t\t\t- Text: ${text}`);
            console.log(`\t\t\t  Sentiment: ${sentiment}`);
          }
        }
      }
    } else {
      console.error(`\tError: ${result.error}`);
    }
  }
}
sentimentAnalysisWithOpinionMining(textAnalyticsClient)
```

Exécutez votre code avec `node index.js` dans la fenêtre de votre console.

### <a name="output"></a>Output

```console
- Document 0
        Document text: The food and service were unacceptable, but the concierge were nice
        Overall Sentiment: positive
        Sentiment confidence scores: { positive: 0.84, neutral: 0, negative: 0.16 }
        Sentences
        - Sentence sentiment: positive
          Confidence scores: { positive: 0.84, neutral: 0, negative: 0.16 }
          Mined opinions
                - Target text: food
                  Target sentiment: negative
                  Target confidence scores: { positive: 0.01, negative: 0.99 }
                  Target assessments
                        - Text: unacceptable
                          Sentiment: negative
                - Target text: service
                  Target sentiment: negative
                  Target confidence scores: { positive: 0.01, negative: 0.99 }
                  Target assessments
                        - Text: unacceptable
                          Sentiment: negative
                - Target text: concierge
                  Target sentiment: positive
                  Target confidence scores: { positive: 1, negative: 0 }
                  Target assessments
                        - Text: nice
                          Sentiment: positive
```

## <a name="language-detection"></a>Détection de la langue

Créez un tableau de chaînes contenant le document que vous souhaitez analyser. Appelez la méthode `detectLanguage()` du client et récupérez l’objet `DetectLanguageResultCollection` retourné. Effectuez ensuite une itération dans les résultats, puis affichez l’ID de chaque document avec la langue principale respective.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function languageDetection(client) {

    const languageInputArray = [
        "Ce document est rédigé en Français."
    ];
    const languageResult = await client.detectLanguage(languageInputArray);

    languageResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tPrimary Language ${document.primaryLanguage.name}`)
    });
}
languageDetection(textAnalyticsClient);
```

Exécutez votre code avec `node index.js` dans la fenêtre de votre console.

### <a name="output"></a>Output

```console
ID: 0
        Primary Language French
```


## <a name="named-entity-recognition-ner"></a>Reconnaissance d’entité nommée (NER)

Créez un tableau de chaînes contenant le document que vous souhaitez analyser. Appelez la méthode `recognizeEntities()` du client et récupérez l’objet `RecognizeEntitiesResult`. Effectuez une itération dans la liste des résultats et affichez le nom de l’entité, le type, le sous-type, le décalage, la longueur et le score.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function entityRecognition(client){

    const entityInputs = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800",
        "La sede principal de Microsoft se encuentra en la ciudad de Redmond, a 21 kilómetros de Seattle."
    ];
    const entityResults = await client.recognizeEntities(entityInputs);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.text} \tCategory: ${entity.category} \tSubcategory: ${entity.subCategory ? entity.subCategory : "N/A"}`);
            console.log(`\tScore: ${entity.confidenceScore}`);
        });
    });
}
entityRecognition(textAnalyticsClient);
```

Exécutez votre code avec `node index.js` dans la fenêtre de votre console.

### <a name="output"></a>Output

```console
Document ID: 0
        Name: Microsoft         Category: Organization  Subcategory: N/A
        Score: 0.29
        Name: Bill Gates        Category: Person        Subcategory: N/A
        Score: 0.78
        Name: Paul Allen        Category: Person        Subcategory: N/A
        Score: 0.82
        Name: April 4, 1975     Category: DateTime      Subcategory: Date
        Score: 0.8
        Name: 8800      Category: Quantity      Subcategory: Number
        Score: 0.8
Document ID: 1
        Name: 21        Category: Quantity      Subcategory: Number
        Score: 0.8
        Name: Seattle   Category: Location      Subcategory: GPE
        Score: 0.25
```

## <a name="personally-identifying-information-pii-recognition"></a>Reconnaissance des informations d’identification personnelle (PII)

Créez un tableau de chaînes contenant le document que vous souhaitez analyser. Appelez la méthode `recognizePiiEntities()` du client et récupérez l’objet `RecognizePIIEntitiesResult`. Effectuez une itération dans la liste des résultats et affichez le nom de l’entité, le type et le score.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function piiRecognition(client) {

    const documents = [
        "The employee's phone number is (555) 555-5555."
    ];

    const results = await client.recognizePiiEntities(documents, "en");
    for (const result of results) {
        if (result.error === undefined) {
            console.log("Redacted Text: ", result.redactedText);
            console.log(" -- Recognized PII entities for input", result.id, "--");
            for (const entity of result.entities) {
                console.log(entity.text, ":", entity.category, "(Score:", entity.confidenceScore, ")");
            }
        } else {
            console.error("Encountered an error:", result.error);
        }
    }
}
piiRecognition(textAnalyticsClient)
```

Exécutez votre code avec `node index.js` dans la fenêtre de votre console.

### <a name="output"></a>Output

```console
Redacted Text:  The employee's phone number is **************.
 -- Recognized PII entities for input 0 --
(555) 555-5555 : Phone Number (Score: 0.8 )
```

## <a name="entity-linking"></a>Liaison d’entités

Créez un tableau de chaînes contenant le document que vous souhaitez analyser. Appelez la méthode `recognizeLinkedEntities()` du client et récupérez l’objet `RecognizeLinkedEntitiesResult`. Effectuez une itération dans la liste des résultats et affichez le nom de l’entité, l’ID, la source de données, l’URL et les correspondances. Chaque objet dans le tableau `matches` contient le décalage, la longueur et le score pour la correspondance concernée.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function linkedEntityRecognition(client){

    const linkedEntityInput = [
        "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, to develop and sell BASIC interpreters for the Altair 8800. During his career at Microsoft, Gates held the positions of chairman, chief executive officer, president and chief software architect, while also being the largest individual shareholder until May 2014."
    ];
    const entityResults = await client.recognizeLinkedEntities(linkedEntityInput);

    entityResults.forEach(document => {
        console.log(`Document ID: ${document.id}`);
        document.entities.forEach(entity => {
            console.log(`\tName: ${entity.name} \tID: ${entity.dataSourceEntityId} \tURL: ${entity.url} \tData Source: ${entity.dataSource}`);
            console.log(`\tMatches:`)
            entity.matches.forEach(match => {
                console.log(`\t\tText: ${match.text} \tScore: ${match.confidenceScore.toFixed(2)}`);
        })
        });
    });
}
linkedEntityRecognition(textAnalyticsClient);
```

Exécutez votre code avec `node index.js` dans la fenêtre de votre console.

### <a name="output"></a>Output

```console
Document ID: 0
        Name: Altair 8800       ID: Altair 8800         URL: https://en.wikipedia.org/wiki/Altair_8800  Data Source: Wikipedia
        Matches:
                Text: Altair 8800       Score: 0.88
        Name: Bill Gates        ID: Bill Gates  URL: https://en.wikipedia.org/wiki/Bill_Gates   Data Source: Wikipedia
        Matches:
                Text: Bill Gates        Score: 0.63
                Text: Gates     Score: 0.63
        Name: Paul Allen        ID: Paul Allen  URL: https://en.wikipedia.org/wiki/Paul_Allen   Data Source: Wikipedia
        Matches:
                Text: Paul Allen        Score: 0.60
        Name: Microsoft         ID: Microsoft   URL: https://en.wikipedia.org/wiki/Microsoft    Data Source: Wikipedia
        Matches:
                Text: Microsoft         Score: 0.55
                Text: Microsoft         Score: 0.55
        Name: April 4   ID: April 4     URL: https://en.wikipedia.org/wiki/April_4      Data Source: Wikipedia
        Matches:
                Text: April 4   Score: 0.32
        Name: BASIC     ID: BASIC       URL: https://en.wikipedia.org/wiki/BASIC        Data Source: Wikipedia
        Matches:
                Text: BASIC     Score: 0.33
```

## <a name="key-phrase-extraction"></a>Extraction d’expressions clés

Créez un tableau de chaînes contenant le document que vous souhaitez analyser. Appelez la méthode `extractKeyPhrases()` du client et récupérez l’objet `ExtractKeyPhrasesResult` retourné. Effectuez une itération dans les résultats, puis imprimez l’ID de chaque document et les phases clés détectées.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function keyPhraseExtraction(client){

    const keyPhrasesInput = [
        "My cat might need to see a veterinarian.",
    ];
    const keyPhraseResult = await client.extractKeyPhrases(keyPhrasesInput);
    
    keyPhraseResult.forEach(document => {
        console.log(`ID: ${document.id}`);
        console.log(`\tDocument Key Phrases: ${document.keyPhrases}`);
    });
}
keyPhraseExtraction(textAnalyticsClient);
```

Exécutez votre code avec `node index.js` dans la fenêtre de votre console.

### <a name="output"></a>Output

```console
ID: 0
        Document Key Phrases: cat,veterinarian
```

## <a name="extract-health-entities"></a>Extraire les entités de santé

[!INCLUDE [health operation pricing](../health-operation-pricing-caution.md)]

Vous pouvez utiliser Analyse de texte pour effectuer une demande asynchrone afin d’extraire les entités de santé du texte. Voici un exemple de base. et [sur GitHub](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/textanalytics/ai-text-analytics/samples/v5/javascript/beginAnalyzeHealthcareEntities.js) un exemple plus avancé.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function healthExample(client) {
    console.log("== Recognize Healthcare Entities Sample ==");

    const documents = [
        "Prescribed 100mg ibuprofen, taken twice daily."
      ];
    const poller = await client.beginAnalyzeHealthcareEntities(documents, "en", {
      includeStatistics: true
    });
  
    poller.onProgress(() => {
      console.log(
        `Last time the operation was updated was on: ${poller.getOperationState().lastModifiedOn}`
      );
    });
    console.log(
      `The analyze healthcare entities operation was created on ${
        poller.getOperationState().createdOn
      }`
    );
    console.log(
      `The analyze healthcare entities operation results will expire on ${
        poller.getOperationState().expiresOn
      }`
    );
  
    const results = await poller.pollUntilDone();
  
    for await (const result of results) {
      console.log(`- Document ${result.id}`);
      if (!result.error) {
        console.log("\tRecognized Entities:");
        for (const entity of result.entities) {
          console.log(`\t- Entity "${entity.text}" of type ${entity.category}`);
        }
        if (result.entityRelations && (result.entityRelations.length > 0)) {
          console.log(`\tRecognized relations between entities:`);
          for (const relation of result.entityRelations) {
            console.log(
              `\t\t- Relation of type ${relation.relationType} found between the following entities:`
            );
            for (const role of relation.roles) {
              console.log(`\t\t\t- "${role.entity.text}" with the role ${role.name}`);
            }
          }
        }
      } else console.error("\tError:", result.error);
    }
  }
  
  healthExample(textAnalyticsClient).catch((err) => {
    console.error("The sample encountered an error:", err);
  });
```

### <a name="output"></a>Output

```console
- Document 0
    Recognized Entities:
    - Entity "100mg" of type Dosage
    - Entity "ibuprofen" of type MedicationName
    - Entity "twice daily" of type Frequency
    Recognized relations between entities:
        - Relation of type DosageOfMedication found between the following entities:   
                - "100mg" with the role Dosage
                - "ibuprofen" with the role Medication
        - Relation of type FrequencyOfMedication found between the following entities:
                - "ibuprofen" with the role Medication
                - "twice daily" with the role Frequency
```


## <a name="use-the-api-asynchronously-with-the-analyze-operation"></a>Utiliser l’API de manière asynchrone avec l’opération d’analyse

L’opération d’analyse peut servir à effectuer différentes demandes de lot asynchrones : la reconnaissance d’entité nommée, l’extraction de phrases clés, l’analyse des sentiments et la détection d’informations d’identification personnelle. Vous trouverez ci-dessous un exemple de base sur une opération. Vous trouverez des exemples plus avancés pour [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/textanalytics/ai-text-analytics/samples/v5/javascript/beginAnalyzeActions.js) et [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/textanalytics/ai-text-analytics/samples/v5/typescript/src/beginAnalyzeActions.ts) sur GitHub.

[!INCLUDE [Analyze Batch Action pricing](../analyze-operation-pricing-caution.md)]

Créez une fonction nommée `analyze_example()` qui doit appeler la fonction `beginAnalyze()`. Il en résultera une opération durable qui sera interrogée pour obtenir des résultats.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```javascript
async function analyze_example(client) {
    const documents = [
        "Microsoft was founded by Bill Gates and Paul Allen.",
    ];

    const actions = {
        recognizeEntitiesActions: [{ modelVersion: "latest" }],
        extractKeyPhrasesActions: [{ modelVersion: "latest" }]
    };
    const poller = await client.beginAnalyzeActions(documents, actions, "en");

    console.log(
        `The analyze batch actions operation was created on ${poller.getOperationState().createdOn}`
    );
    console.log(
        `The analyze batch actions operation results will expire on ${poller.getOperationState().expiresOn
        }`
    );
    const resultPages = await poller.pollUntilDone();
    for await (const page of resultPages) {
        const entitiesAction = page.recognizeEntitiesResults[0];
        if (!entitiesAction.error) {
            for (const doc of entitiesAction.results) {
                console.log(`- Document ${doc.id}`);
                if (!doc.error) {
                    console.log("\tEntities:");
                    for (const entity of doc.entities) {
                        console.log(`\t- Entity ${entity.text} of type ${entity.category}`);
                    }
                } else {
                    console.error("\tError:", doc.error);
                }
            }
        }
        const keyPhrasesAction = page.extractKeyPhrasesResults[0];
        if (!keyPhrasesAction.error) {
            for (const doc of keyPhrasesAction.results) {
                console.log(`- Document ${doc.id}`);
                if (!doc.error) {
                    console.log("\tKey phrases:");
                    for (const phrase of doc.keyPhrases) {
                        console.log(`\t- ${phrase}`);
                    }
                } else {
                    console.error("\tError:", doc.error);
                }
            }
        }
    }
}
analyze_example(textAnalyticsClient)
```

### <a name="output"></a>Sortie

```console
The analyze batch actions operation was created on Fri Jun 18 2021 12:34:52 GMT-0700 (Pacific Daylight Time)
The analyze batch actions operation results will expire on Sat Jun 19 2021 12:34:52 GMT-0700 (Pacific Daylight Time)
- Document 0
        Entities:
        - Entity Microsoft of type Organization
        - Entity Bill Gates of type Person
        - Entity Paul Allen of type Person
- Document 0
        Key phrases:
        - Bill Gates
        - Paul Allen
        - Microsoft
```

Vous pouvez également utiliser l’opération d’analyse pour exécuter NER, l’extraction d’expressions clés, l’analyse des sentiments et la détection PII. Consultez les exemples d’analyse pour [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/textanalytics/ai-text-analytics/samples/v5/javascript/beginAnalyzeActions.js) et [TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/textanalytics/ai-text-analytics/samples/v5/typescript/src) sur GitHub.

## <a name="text-summarization"></a>Synthèse de texte

# <a name="version-32-preview1"></a>[Version 3.2-preview.1](#tab/version-3-2)

Vous pouvez utiliser l’Analyse de texte pour synthétiser de blocs de texte volumineux. La méthode est asynchrone et retourne les premières phrases quand l’opération de longue durée se termine.

```javascript
async function summarization_example(client) {
    const documents = [`The extractive summarization feature in Text Analytics uses natural language processing techniques to locate key sentences in an unstructured text document. 
        These sentences collectively convey the main idea of the document. This feature is provided as an API for developers. 
        They can use it to build intelligent solutions based on the relevant information extracted to support various use cases. 
        In the public preview, extractive summarization supports several languages. It is based on pretrained multilingual transformer models, part of our quest for holistic representations. 
        It draws its strength from transfer learning across monolingual and harness the shared nature of languages to produce models of improved quality and efficiency.`];

    console.log("== Analyze Sample For Extract Summary ==");

    const actions = {
        extractSummaryActions: [{ modelVersion: "latest", orderBy: "Rank", maxSentenceCount: 5 }],
    };
    const poller = await client.beginAnalyzeActions(documents, actions, "en");

    poller.onProgress(() => {
        console.log(
            `Number of actions still in progress: ${poller.getOperationState().actionsInProgressCount}`
        );
    });

    console.log(`The analyze actions operation created on ${poller.getOperationState().createdOn}`);

    console.log(
        `The analyze actions operation results will expire on ${poller.getOperationState().expiresOn}`
    );

    const resultPages = await poller.pollUntilDone();

    for await (const page of resultPages) {
        const extractSummaryAction = page.extractSummaryResults[0];
        if (!extractSummaryAction.error) {
            for (const doc of extractSummaryAction.results) {
                console.log(`- Document ${doc.id}`);
                if (!doc.error) {
                    console.log("\tSummary:");
                    for (const sentence of doc.sentences) {
                        console.log(`\t- ${sentence.text}`);
                    }
                } else {
                    console.error("\tError:", doc.error);
                }
            }
        }
    } 
}
summarization_example(textAnalyticsClient).catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

# <a name="version-31"></a>[Version 3.1](#tab/version-3-1)

Cette fonctionnalité n’est pas disponible dans la version 3.1

---

Exécutez l’application avec la commande `node` de votre fichier de démarrage rapide.

```console
node index.js
```
