---
title: 'Démarrage rapide : Bibliothèque cliente Analyse de texte v3 pour Java | Microsoft Docs'
description: Découvrez comment utiliser la bibliothèque cliente Analyse de texte v3 pour Java.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 08/17/2021
ms.custom: devx-track-java
ms.author: aahi
ms.reviewer: tasharm, assafi, sumeh
ms.openlocfilehash: 3284cef03ce899a096816ba23736d29fdc228eb5
ms.sourcegitcommit: 7854045df93e28949e79765a638ec86f83d28ebc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2021
ms.locfileid: "122864853"
---
<a name="HOLTop"></a>

# <a name="version-32-preview1"></a>[Version 3.2-preview.1](#tab/version-3-2)

[Documentation de référence de la version v3.2-preview.1](/java/api/overview/azure/ai-textanalytics-readme?preserve-view=true&view=azure-java-preview) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/textanalytics/azure-ai-textanalytics) | [Package v3.2-preview](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.2.0-beta.1) | [Exemples](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/textanalytics/azure-ai-textanalytics/src/samples)

# <a name="version-31"></a>[Version 3.1](#tab/version-3-1)

[Documentation de référence](/java/api/overview/azure/ai-textanalytics-readme?preserve-view=true&view=azure-java-stable) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-java/blob/azure-ai-textanalytics_5.0.0/sdk/textanalytics/azure-ai-textanalytics) | [Package v3.1](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.1.0) | [Exemples](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-textanalytics_5.0.0/sdk/textanalytics/azure-ai-textanalytics/src/samples/java/com/azure/ai/textanalytics)

---

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* [Kit de développement Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html) avec version 8 ou ultérieure
* [!INCLUDE [contributor-requirement](../../../includes/quickstarts/contributor-requirement.md)]
* Une fois que vous avez votre abonnement Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="créez une ressource Analyse de texte"  target="_blank">Créer une ressource Analyse de texte </a> dans le portail Azure pour obtenir votre clé et votre point de terminaison.  Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour connecter votre application à l’API Analyse de texte. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.
* Pour utiliser la fonctionnalité d’analyse, vous aurez besoin d’une ressource Analyse de texte avec le niveau tarifaire Standard (S).

## <a name="setting-up"></a>Configuration

### <a name="add-the-client-library"></a>Ajouter la bibliothèque de client

# <a name="version-32-preview"></a>[Version 3.2-preview](#tab/version-3-2)

Créez un projet Maven dans l’IDE ou l’environnement de développement de votre choix. Ensuite, ajoutez la dépendance suivante au fichier *pom.xml* de votre projet : Vous trouverez la syntaxe d’implémentation [pour d’autres outils de génération](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.2.0-beta.1) en ligne.

```xml
<dependencies>
     <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-ai-textanalytics</artifactId>
        <version>5.2.0-beta.1</version>
    </dependency>
</dependencies>
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

Créez un projet Maven dans l’IDE ou l’environnement de développement de votre choix. Ensuite, ajoutez la dépendance suivante au fichier *pom.xml* de votre projet : Vous trouverez la syntaxe d’implémentation [pour d’autres outils de génération](https://mvnrepository.com/artifact/com.azure/azure-ai-textanalytics/5.1.0) en ligne.

```xml
<dependencies>
     <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-ai-textanalytics</artifactId>
        <version>5.1.0</version>
    </dependency>
</dependencies>
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

Créez un fichier Java nommé `TextAnalyticsSamples.java`. Ouvrez le fichier et ajoutez les instructions `import` suivantes :

```java
import com.azure.ai.textanalytics.TextAnalyticsAsyncClient;
import com.azure.core.credential.AzureKeyCredential;
import com.azure.ai.textanalytics.models.*;
import com.azure.ai.textanalytics.TextAnalyticsClientBuilder;
import com.azure.ai.textanalytics.TextAnalyticsClient;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

import java.util.Arrays;
import com.azure.core.util.Context;
import com.azure.core.util.polling.SyncPoller;

import com.azure.core.util.polling.LongRunningOperationStatus;
import com.azure.ai.textanalytics.util.*;
import com.azure.core.http.rest.PagedResponse;
```

Dans le fichier Java, ajoutez une nouvelle classe ainsi que la clé et le point de terminaison de votre ressource Azure, comme indiqué ci-dessous.

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

```java
public class TextAnalyticsSamples {
    private static String KEY = "<replace-with-your-text-analytics-key-here>";
    private static String ENDPOINT = "<replace-with-your-text-analytics-endpoint-here>";
}
```

Ajoutez la méthode main suivante à la classe. Vous définirez les méthodes appelées ici ultérieurement.

```java
public static void main(String[] args) {
    //You will create these methods later in the quickstart.
    TextAnalyticsClient client = authenticateClient(KEY, ENDPOINT);
    sentimentAnalysisExample(client);
    sentimentAnalysisWithOpinionMiningExample(client);
    detectLanguageExample(client);
    recognizeEntitiesExample(client);
    recognizePiiEntitiesExample(client);
    recognizeLinkedEntitiesExample(client);
    extractKeyPhrasesExample(client);
    analyzeOperationExample(client);
}
```


## <a name="object-model"></a>Modèle objet

Le client Analyse de texte est un objet `TextAnalyticsClient` qui s’authentifie auprès d’Azure avec votre clé et fournit des fonctions permettant d’accepter du texte sous forme de chaînes individuelles ou de lots. Vous pouvez envoyer du texte à l’API de façon synchrone ou de façon asynchrone. L’objet Response contient les informations d’analyse de chaque document que vous envoyez. 

[!INCLUDE [text-analytics-character-limits](../character-limits.md)]

## <a name="authenticate-the-client"></a>Authentifier le client

Créez une méthode pour instancier l’objet `TextAnalyticsClient` avec la clé et le point de terminaison de votre ressource Analyse de texte. Cet exemple est identique pour les versions 3.0 et 3.1 de l’API.

```java
static TextAnalyticsClient authenticateClient(String key, String endpoint) {
    return new TextAnalyticsClientBuilder()
        .credential(new AzureKeyCredential(key))
        .endpoint(endpoint)
        .buildClient();
}
```


Dans la méthode `main()` de votre programme, appelez la méthode d’authentification pour instancier le client.

## <a name="sentiment-analysis"></a>analyse de sentiments

Créez une fonction appelée `sentimentAnalysisExample()` qui accepte le client que vous avez créé, puis appelez sa fonction `analyzeSentiment()`. L’objet `AnalyzeSentimentResult` retourné contient `documentSentiment` et `sentenceSentiments` si l’opération réussit, ou un `errorMessage` si ce n’est pas le cas. 

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```java
static void sentimentAnalysisExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "I had the best day of my life. I wish you were there with me.";

    DocumentSentiment documentSentiment = client.analyzeSentiment(text);
    System.out.printf(
        "Recognized document sentiment: %s, positive score: %s, neutral score: %s, negative score: %s.%n",
        documentSentiment.getSentiment(),
        documentSentiment.getConfidenceScores().getPositive(),
        documentSentiment.getConfidenceScores().getNeutral(),
        documentSentiment.getConfidenceScores().getNegative());

    for (SentenceSentiment sentenceSentiment : documentSentiment.getSentences()) {
        System.out.printf(
            "Recognized sentence sentiment: %s, positive score: %s, neutral score: %s, negative score: %s.%n",
            sentenceSentiment.getSentiment(),
            sentenceSentiment.getConfidenceScores().getPositive(),
            sentenceSentiment.getConfidenceScores().getNeutral(),
            sentenceSentiment.getConfidenceScores().getNegative());
        }
    }
}
```

### <a name="output"></a>Output

```console
Recognized document sentiment: positive, positive score: 1.0, neutral score: 0.0, negative score: 0.0.
Recognized sentence sentiment: positive, positive score: 1.0, neutral score: 0.0, negative score: 0.0.
Recognized sentence sentiment: neutral, positive score: 0.21, neutral score: 0.77, negative score: 0.02.
```

## <a name="opinion-mining"></a>Exploration des opinions

Pour effectuer une analyse des sentiments avec exploration des opinions, créez une fonction appelée `sentimentAnalysisWithOpinionMiningExample()` qui accepte le client que vous avez créé, puis appelez sa fonction `analyzeSentiment()` en définissant l’objet facultatif `AnalyzeSentimentOptions`. L’objet `AnalyzeSentimentResult` retourné contient `documentSentiment` et `sentenceSentiments` si l’opération réussit, ou un `errorMessage` si ce n’est pas le cas. 

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]


```java
static void sentimentAnalysisWithOpinionMiningExample(TextAnalyticsClient client)
{
    // The document that needs be analyzed.
    String document = "Bad atmosphere. Not close to plenty of restaurants, hotels, and transit! Staff are not friendly and helpful.";

    System.out.printf("Document = %s%n", document);

    AnalyzeSentimentOptions options = new AnalyzeSentimentOptions().setIncludeOpinionMining(true);
    final DocumentSentiment documentSentiment = client.analyzeSentiment(document, "en", options);
    SentimentConfidenceScores scores = documentSentiment.getConfidenceScores();
    System.out.printf(
            "Recognized document sentiment: %s, positive score: %f, neutral score: %f, negative score: %f.%n",
            documentSentiment.getSentiment(), scores.getPositive(), scores.getNeutral(), scores.getNegative());


    documentSentiment.getSentences().forEach(sentenceSentiment -> {
        SentimentConfidenceScores sentenceScores = sentenceSentiment.getConfidenceScores();
        System.out.printf("\tSentence sentiment: %s, positive score: %f, neutral score: %f, negative score: %f.%n",
                sentenceSentiment.getSentiment(), sentenceScores.getPositive(), sentenceScores.getNeutral(), sentenceScores.getNegative());
        sentenceSentiment.getOpinions().forEach(opinion -> {
            TargetSentiment targetSentiment = opinion.getTarget();
            System.out.printf("\t\tTarget sentiment: %s, target text: %s%n", targetSentiment.getSentiment(),
                    targetSentiment.getText());
            for (AssessmentSentiment assessmentSentiment : opinion.getAssessments()) {
                System.out.printf("\t\t\t'%s' assessment sentiment because of \"%s\". Is the assessment negated: %s.%n",
                        assessmentSentiment.getSentiment(), assessmentSentiment.getText(), assessmentSentiment.isNegated());
            }
        });
    });
}
```

### <a name="output"></a>Output

```console
Document = Bad atmosphere. Not close to plenty of restaurants, hotels, and transit! Staff are not friendly and helpful.
Recognized document sentiment: negative, positive score: 0.010000, neutral score: 0.140000, negative score: 0.850000.
    Sentence sentiment: negative, positive score: 0.000000, neutral score: 0.000000, negative score: 1.000000.
        Target sentiment: negative, target text: atmosphere
            'negative' assessment sentiment because of "bad". Is the assessment negated: false.
    Sentence sentiment: negative, positive score: 0.020000, neutral score: 0.440000, negative score: 0.540000.
    Sentence sentiment: negative, positive score: 0.000000, neutral score: 0.000000, negative score: 1.000000.
        Target sentiment: negative, target text: Staff
            'negative' assessment sentiment because of "friendly". Is the assessment negated: true.
            'negative' assessment sentiment because of "helpful". Is the assessment negated: true.
```

## <a name="language-detection"></a>Détection de la langue

Créez une fonction appelée `detectLanguageExample()` qui accepte le client que vous avez créé, puis appelez sa fonction `detectLanguage()`. L’objet `DetectLanguageResult` retourné contient une langue principale détectée, une liste d’autres langues détectées si l’opération réussit ou un `errorMessage` si ce n’est pas le cas. Cet exemple est identique pour les versions 3.0 et 3.1 de l’API.

> [!Tip]
> Dans certains cas, il peut être difficile de lever toute ambiguïté sur les langues en fonction de l’entrée. Vous pouvez utiliser le paramètre `countryHint` pour spécifier un code de pays à 2 lettres. L’API utilise « US » comme countryHint par défaut. Pour modifier ce comportement, vous pouvez réinitialiser ce paramètre en définissant cette valeur sur une chaîne vide `countryHint = ""`. Pour définir une autre valeur par défaut, définissez la propriété `TextAnalyticsClientOptions.DefaultCountryHint` et transmettez-la pendant l’initialisation du client.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```java
static void detectLanguageExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "Ce document est rédigé en Français.";

    DetectedLanguage detectedLanguage = client.detectLanguage(text);
    System.out.printf("Detected primary language: %s, ISO 6391 name: %s, score: %.2f.%n",
        detectedLanguage.getName(),
        detectedLanguage.getIso6391Name(),
        detectedLanguage.getConfidenceScore());
}
```

### <a name="output"></a>Output

```console
Detected primary language: French, ISO 6391 name: fr, score: 1.00.
```

## <a name="named-entity-recognition-ner"></a>Reconnaissance d’entité nommée (NER)

Créez une fonction appelée `recognizeEntitiesExample()` qui accepte le client que vous avez créé, puis appelez sa fonction `recognizeEntities()`. L’objet `CategorizedEntityCollection` retourné contient une liste de `CategorizedEntity` si l’opération réussit, ou un `errorMessage` si ce n’est pas le cas.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```java
static void recognizeEntitiesExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "I had a wonderful trip to Seattle last week.";

    for (CategorizedEntity entity : client.recognizeEntities(text)) {
        System.out.printf(
            "Recognized entity: %s, entity category: %s, entity sub-category: %s, score: %s, offset: %s, length: %s.%n",
            entity.getText(),
            entity.getCategory(),
            entity.getSubcategory(),
            entity.getConfidenceScore(),
            entity.getOffset(),
            entity.getLength());
    }
}
```

### <a name="output"></a>Output

```console
Recognized entity: trip, entity category: Event, entity sub-category: null, score: 0.61, offset: 8, length: 4.
Recognized entity: Seattle, entity category: Location, entity sub-category: GPE, score: 0.82, offset: 16, length: 7.
Recognized entity: last week, entity category: DateTime, entity sub-category: DateRange, score: 0.8, offset: 24, length: 9.
```


## <a name="personally-identifiable-information-pii-recognition"></a>Reconnaissance des informations d’identification personnelle

Créez une fonction appelée `recognizePiiEntitiesExample()` qui accepte le client que vous avez créé, puis appelez sa fonction `recognizePiiEntities()`. L’objet `PiiEntityCollection` retourné contient une liste de `PiiEntity` si l’opération réussit, ou un `errorMessage` si ce n’est pas le cas. Il contient également le texte rédigé, qui se compose du texte d’entrée, où toutes les entités identifiables sont remplacées par `*****`.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```java
static void recognizePiiEntitiesExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String document = "My SSN is 859-98-0987";
    PiiEntityCollection piiEntityCollection = client.recognizePiiEntities(document);
    System.out.printf("Redacted Text: %s%n", piiEntityCollection.getRedactedText());
    piiEntityCollection.forEach(entity -> System.out.printf(
        "Recognized Personally Identifiable Information entity: %s, entity category: %s, entity subcategory: %s,"
            + " confidence score: %f.%n",
        entity.getText(), entity.getCategory(), entity.getSubcategory(), entity.getConfidenceScore()));
}
```

### <a name="output"></a>Output

```console
Redacted Text: My SSN is ***********
Recognized Personally Identifiable Information entity: 859-98-0987, entity category: U.S. Social Security Number (SSN), entity subcategory: null, confidence score: 0.650000.
```

## <a name="entity-linking"></a>Liaison d’entités

Créez une fonction appelée `recognizeLinkedEntitiesExample()` qui accepte le client que vous avez créé, puis appelez sa fonction `recognizeLinkedEntities()`. L’objet `LinkedEntityCollection` retourné contient une liste de `LinkedEntity` si l’opération réussit, ou un `errorMessage` si ce n’est pas le cas. Les entités liées étant identifiées de manière unique, les occurrences d’une même entité sont regroupées sous un objet `LinkedEntity` sous la forme d’une liste d’objets `LinkedEntityMatch`.


```java
static void recognizeLinkedEntitiesExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "Microsoft was founded by Bill Gates and Paul Allen on April 4, 1975, " +
        "to develop and sell BASIC interpreters for the Altair 8800. " +
        "During his career at Microsoft, Gates held the positions of chairman, " +
        "chief executive officer, president and chief software architect, " +
        "while also being the largest individual shareholder until May 2014.";

    System.out.printf("Linked Entities:%n");
    for (LinkedEntity linkedEntity : client.recognizeLinkedEntities(text)) {
        System.out.printf("Name: %s, ID: %s, URL: %s, Data Source: %s.%n",
            linkedEntity.getName(),
            linkedEntity.getDataSourceEntityId(),
            linkedEntity.getUrl(),
            linkedEntity.getDataSource());
        System.out.printf("Matches:%n");
        for (LinkedEntityMatch linkedEntityMatch : linkedEntity.getMatches()) {
            System.out.printf("Text: %s, Score: %.2f, Offset: %s, Length: %s%n",
            linkedEntityMatch.getText(),
            linkedEntityMatch.getConfidenceScore(),
            linkedEntityMatch.getOffset(),
            linkedEntityMatch.getLength());
        }
    }
}
```

### <a name="output"></a>Output

```console
Linked Entities:
Name: Microsoft, ID: Microsoft, URL: https://en.wikipedia.org/wiki/Microsoft, Data Source: Wikipedia.
Matches:
Text: Microsoft, Score: 0.55, Offset: 9, Length: 0
Text: Microsoft, Score: 0.55, Offset: 9, Length: 150
Name: Bill Gates, ID: Bill Gates, URL: https://en.wikipedia.org/wiki/Bill_Gates, Data Source: Wikipedia.
Matches:
Text: Bill Gates, Score: 0.63, Offset: 10, Length: 25
Text: Gates, Score: 0.63, Offset: 5, Length: 161
Name: Paul Allen, ID: Paul Allen, URL: https://en.wikipedia.org/wiki/Paul_Allen, Data Source: Wikipedia.
Matches:
Text: Paul Allen, Score: 0.60, Offset: 10, Length: 40
Name: April 4, ID: April 4, URL: https://en.wikipedia.org/wiki/April_4, Data Source: Wikipedia.
Matches:
Text: April 4, Score: 0.32, Offset: 7, Length: 54
Name: BASIC, ID: BASIC, URL: https://en.wikipedia.org/wiki/BASIC, Data Source: Wikipedia.
Matches:
Text: BASIC, Score: 0.33, Offset: 5, Length: 89
Name: Altair 8800, ID: Altair 8800, URL: https://en.wikipedia.org/wiki/Altair_8800, Data Source: Wikipedia.
Matches:
Text: Altair 8800, Score: 0.88, Offset: 11, Length: 116
```

## <a name="key-phrase-extraction"></a>Extraction d’expressions clés

Créez une fonction appelée `extractKeyPhrasesExample()` qui accepte le client que vous avez créé, puis appelez sa fonction `extractKeyPhrases()`. L’objet `ExtractKeyPhraseResult` retourné contient une liste d’expressions clés si l’opération réussit, ou un `errorMessage` si ce n’est pas le cas. Cet exemple est identique pour les versions 3.0 et 3.1 de l’API.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```java
static void extractKeyPhrasesExample(TextAnalyticsClient client)
{
    // The text that need be analyzed.
    String text = "My cat might need to see a veterinarian.";

    System.out.printf("Recognized phrases: %n");
    for (String keyPhrase : client.extractKeyPhrases(text)) {
        System.out.printf("%s%n", keyPhrase);
    }
}
```

### <a name="output"></a>Output

```console
Recognized phrases: 
cat
veterinarian
```
---

## <a name="extract-health-entities"></a>Extraire les entités de santé

[!INCLUDE [health operation pricing](../health-operation-pricing-caution.md)]

Vous pouvez utiliser Analyse de texte pour effectuer une demande asynchrone afin d’extraire les entités de santé du texte. Voici un exemple de base. et [sur GitHub](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/textanalytics/azure-ai-textanalytics/src/samples/java/com/azure/ai/textanalytics/lro/AnalyzeHealthcareEntities.java) un exemple plus avancé.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]


```java
static void healthExample(TextAnalyticsClient client){
    List<TextDocumentInput> documents = Arrays.asList(
            new TextDocumentInput("0",
                    "Prescribed 100mg ibuprofen, taken twice daily."));

    AnalyzeHealthcareEntitiesOptions options = new AnalyzeHealthcareEntitiesOptions().setIncludeStatistics(true);

    SyncPoller<AnalyzeHealthcareEntitiesOperationDetail, AnalyzeHealthcareEntitiesPagedIterable>
            syncPoller = client.beginAnalyzeHealthcareEntities(documents, options, Context.NONE);

    System.out.printf("Poller status: %s.%n", syncPoller.poll().getStatus());
    syncPoller.waitForCompletion();

    // Task operation statistics
    AnalyzeHealthcareEntitiesOperationDetail operationResult = syncPoller.poll().getValue();
    System.out.printf("Operation created time: %s, expiration time: %s.%n",
            operationResult.getCreatedAt(), operationResult.getExpiresAt());
    System.out.printf("Poller status: %s.%n", syncPoller.poll().getStatus());

    for (AnalyzeHealthcareEntitiesResultCollection resultCollection : syncPoller.getFinalResult()) {
        // Model version
        System.out.printf(
                "Results of Azure Text Analytics \"Analyze Healthcare Entities\" Model, version: %s%n",
                resultCollection.getModelVersion());

        for (AnalyzeHealthcareEntitiesResult healthcareEntitiesResult : resultCollection) {
            System.out.println("Document ID = " + healthcareEntitiesResult.getId());
            System.out.println("Document entities: ");
            // Recognized healthcare entities
            for (HealthcareEntity entity : healthcareEntitiesResult.getEntities()) {
                System.out.printf(
                        "\tText: %s, normalized name: %s, category: %s, subcategory: %s, confidence score: %f.%n",
                        entity.getText(), entity.getNormalizedText(), entity.getCategory(),
                        entity.getSubcategory(), entity.getConfidenceScore());
            }
            // Recognized healthcare entity relation groups
            for (HealthcareEntityRelation entityRelation : healthcareEntitiesResult.getEntityRelations()) {
                System.out.printf("Relation type: %s.%n", entityRelation.getRelationType());
                for (HealthcareEntityRelationRole role : entityRelation.getRoles()) {
                    HealthcareEntity entity = role.getEntity();
                    System.out.printf("\tEntity text: %s, category: %s, role: %s.%n",
                            entity.getText(), entity.getCategory(), role.getName());
                }
            }
        }
    }
}
```

Dans votre méthode `main()`, appelez l’exemple de code ci-dessus. 

```java
public static void main(String[] args) {
    TextAnalyticsClient client = authenticateClient(KEY, ENDPOINT);
    healthExample(client);
}
```

### <a name="output"></a>sortie

```console
Poller status: IN_PROGRESS.
Operation created time: 2021-07-20T19:45:50Z, expiration time: 2021-07-21T19:45:50Z.
Poller status: SUCCESSFULLY_COMPLETED.
Results of Azure Text Analytics "Analyze Healthcare Entities" Model, version: 2021-05-15
Document ID = 0
Document entities: 
    Text: 100mg, normalized name: null, category: Dosage, subcategory: null, confidence score: 1.000000.
    Text: ibuprofen, normalized name: ibuprofen, category: MedicationName, subcategory: null, confidence score: 1.000000.
    Text: twice daily, normalized name: null, category: Frequency, subcategory: null, confidence score: 1.000000.
Relation type: DosageOfMedication.
    Entity text: 100mg, category: Dosage, role: Dosage.
    Entity text: ibuprofen, category: MedicationName, role: Medication.
Relation type: FrequencyOfMedication.
    Entity text: ibuprofen, category: MedicationName, role: Medication.
    Entity text: twice daily, category: Frequency, role: Frequency.
```

## <a name="use-the-api-asynchronously-with-the-analyze-operation"></a>Utiliser l’API de manière asynchrone avec l’opération d’analyse

L’opération d’analyse peut servir à effectuer différentes demandes de lot asynchrones : la reconnaissance d’entité nommée, l’extraction de phrases clés, l’analyse des sentiments et la détection d’informations d’identification personnelle. Vous trouverez ci-dessous un exemple de base sur une opération. Vous trouverez un exemple plus élaboré [sur GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample_AnalyzeActions.md)

[!INCLUDE [Analyze Batch Action pricing](../analyze-operation-pricing-caution.md)]

Créez une fonction nommée `analyzeBatchActionsExample()` qui doit appeler la fonction `beginAnalyzeBatchActions()`. Il en résultera une opération durable qui sera interrogée pour obtenir des résultats.

[!INCLUDE [The following method applies to both v3.1 and v3.2-preview](../method-applies-both-versions.md)]

```java
static void analyzeActionsExample(TextAnalyticsClient client){
    List<TextDocumentInput> documents = new ArrayList<>();
    documents.add(new TextDocumentInput("0", "Microsoft was founded by Bill Gates and Paul Allen."));


    SyncPoller<AnalyzeActionsOperationDetail, AnalyzeActionsResultPagedIterable> syncPoller =
            client.beginAnalyzeActions(documents,
                    new TextAnalyticsActions().setDisplayName("Example analyze task")
                            .setRecognizeEntitiesActions(new RecognizeEntitiesAction())
                            .setExtractKeyPhrasesActions(
                                    new ExtractKeyPhrasesAction().setModelVersion("latest")),
                    new AnalyzeActionsOptions().setIncludeStatistics(false),
                    Context.NONE);

    // Task operation statistics details
    while (syncPoller.poll().getStatus() == LongRunningOperationStatus.IN_PROGRESS) {
        final AnalyzeActionsOperationDetail operationDetail = syncPoller.poll().getValue();
        System.out.printf("Action display name: %s, Successfully completed actions: %d, in-process actions: %d,"
                        + " failed actions: %d, total actions: %d%n",
                operationDetail.getDisplayName(), operationDetail.getSucceededCount(),
                operationDetail.getInProgressCount(), operationDetail.getFailedCount(),
                operationDetail.getTotalCount());
    }

    syncPoller.waitForCompletion();

    Iterable<PagedResponse<AnalyzeActionsResult>> pagedResults = syncPoller.getFinalResult().iterableByPage();
    for (PagedResponse<AnalyzeActionsResult> perPage : pagedResults) {
        System.out.printf("Response code: %d, Continuation Token: %s.%n", perPage.getStatusCode(),
                perPage.getContinuationToken());
        for (AnalyzeActionsResult actionsResult : perPage.getElements()) {
            System.out.println("Entities recognition action results:");
            for (RecognizeEntitiesActionResult actionResult : actionsResult.getRecognizeEntitiesResults()) {
                if (!actionResult.isError()) {
                    for (RecognizeEntitiesResult documentResult : actionResult.getDocumentsResults()) {
                        if (!documentResult.isError()) {
                            for (CategorizedEntity entity : documentResult.getEntities()) {
                                System.out.printf(
                                        "\tText: %s, category: %s, confidence score: %f.%n",
                                        entity.getText(), entity.getCategory(), entity.getConfidenceScore());
                            }
                        } else {
                            System.out.printf("\tCannot recognize entities. Error: %s%n",
                                    documentResult.getError().getMessage());
                        }
                    }
                } else {
                    System.out.printf("\tCannot execute Entities Recognition action. Error: %s%n",
                            actionResult.getError().getMessage());
                }
            }

            System.out.println("Key phrases extraction action results:");
            for (ExtractKeyPhrasesActionResult actionResult : actionsResult.getExtractKeyPhrasesResults()) {
                if (!actionResult.isError()) {
                    for (ExtractKeyPhraseResult documentResult : actionResult.getDocumentsResults()) {
                        if (!documentResult.isError()) {
                            System.out.println("\tExtracted phrases:");
                            for (String keyPhrases : documentResult.getKeyPhrases()) {
                                System.out.printf("\t\t%s.%n", keyPhrases);
                            }
                        } else {
                            System.out.printf("\tCannot extract key phrases. Error: %s%n",
                                    documentResult.getError().getMessage());
                        }
                    }
                } else {
                    System.out.printf("\tCannot execute Key Phrases Extraction action. Error: %s%n",
                            actionResult.getError().getMessage());
                }
            }
        }
    }
}
```

Après avoir ajouté cet exemple à votre application, appelez-le dans votre méthode `main()`.

```java
analyzeBatchActionsExample(client);
```

### <a name="output"></a>Sortie

```console
Action display name: Example analyze task, Successfully completed actions: 1, in-process actions: 1, failed actions: 0, total actions: 2
Response code: 200, Continuation Token: null.
Entities recognition action results:
    Text: Microsoft, category: Organization, confidence score: 1.000000.
    Text: Bill Gates, category: Person, confidence score: 1.000000.
    Text: Paul Allen, category: Person, confidence score: 1.000000.
Key phrases extraction action results:
    Extracted phrases:
        Bill Gates.
        Paul Allen.
        Microsoft.
```

Vous pouvez également utiliser l’opération d’analyse pour la reconnaissance d’entité nommée (NER), l’extraction d’expressions clés, l’analyse des sentiments et la détection des informations d’identification personnelle (PII). Consultez l’[exemple d’analyse](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/textanalytics/azure-ai-textanalytics/src/samples/java/com/azure/ai/textanalytics/lro/AnalyzeActionsAsync.java) sur GitHub.

## <a name="text-summarization"></a>Synthèse de texte

# <a name="version-32-preview1"></a>[Version 3.2-preview.1](#tab/version-3-2)

Vous pouvez utiliser l’Analyse de texte pour synthétiser de blocs de texte volumineux. La méthode est asynchrone et retourne les premières phrases quand l’opération de longue durée se termine.

```java
static void summarizationExample(TextAnalyticsClient client) {
    List<String> documents = new ArrayList<>();
    documents.add(
            "The extractive summarization feature in Text Analytics uses natural language processing techniques "
            + "to locate key sentences in an unstructured text document. "
            + "These sentences collectively convey the main idea of the document. This feature is provided as an API for developers. "
            + "They can use it to build intelligent solutions based on the relevant information extracted to support various use cases. "
            + "In the public preview, extractive summarization supports several languages. "
            + "It is based on pretrained multilingual transformer models, part of our quest for holistic representations. "
            + "It draws its strength from transfer learning across monolingual and harness the shared nature of languages "
            + "to produce models of improved quality and efficiency.");

    SyncPoller<AnalyzeActionsOperationDetail, AnalyzeActionsResultPagedIterable> syncPoller =
            client.beginAnalyzeActions(documents,
                    new TextAnalyticsActions().setDisplayName("{tasks_display_name}")
                            .setExtractSummaryActions(
                                    new ExtractSummaryAction()),
                    "en",
                    new AnalyzeActionsOptions());

    syncPoller.waitForCompletion();

    syncPoller.getFinalResult().forEach(actionsResult -> {
        System.out.println("Extractive Summarization action results:");
        for (ExtractSummaryActionResult actionResult : actionsResult.getExtractSummaryResults()) {
            if (!actionResult.isError()) {
                for (ExtractSummaryResult documentResult : actionResult.getDocumentsResults()) {
                    if (!documentResult.isError()) {
                        System.out.println("\tExtracted summary sentences:");
                        for (SummarySentence summarySentence : documentResult.getSentences()) {
                            System.out.printf(
                                    "\t\t Sentence text: %s, length: %d, offset: %d, rank score: %f.%n",
                                    summarySentence.getText(), summarySentence.getLength(),
                                    summarySentence.getOffset(), summarySentence.getRankScore());
                        }
                    } else {
                        System.out.printf("\tCannot extract summary sentences. Error: %s%n",
                                documentResult.getError().getMessage());
                    }
                }
            } else {
                System.out.printf("\tCannot execute Extractive Summarization action. Error: %s%n",
                        actionResult.getError().getMessage());
            }
        }
    });
}
```

Après avoir ajouté cet exemple à votre application, appelez-le dans votre méthode `main()`.

```java
summarizationExample(client);
```

# <a name="version-31"></a>[Version 3.1](#tab/version-3-1)

Cette fonctionnalité n’est pas disponible dans la version 3.1.

---
