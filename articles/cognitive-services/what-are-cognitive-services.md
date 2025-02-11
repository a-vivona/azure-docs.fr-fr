---
title: Présentation d’Azure Cognitive Services
titleSuffix: Azure Cognitive Services
description: Cognitive Services rend l’IA accessible à tout développeur sans exiger d’expertise en matière de machine learning et de science des données. Vous devez simplement effectuer un appel d’API à partir de votre application pour ajouter la possibilité de voir (recherche et reconnaissance d’images avancées), écouter, parler, rechercher et prendre des décisions dans vos applications.
services: cognitive-services
author: nitinme
manager: nitinme
keywords: cognitive services, intelligence cognitive, solutions cognitives, services d’IA, compréhension cognitive, caractéristiques cognitives
ms.service: cognitive-services
ms.subservice: ''
ms.topic: overview
ms.date: 04/16/2021
ms.author: nitinme
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: 4badad1f7e22fcb82e122a56e219fc61973634a2
ms.sourcegitcommit: 7d63ce88bfe8188b1ae70c3d006a29068d066287
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/22/2021
ms.locfileid: "114451261"
---
# <a name="what-are-azure-cognitive-services"></a>Présentation d’Azure Cognitive Services

Azure Cognitive Services regroupe des services cloud dotés d’API REST et de SDK de bibliothèque de client qui vous aident à intégrer une intelligence cognitive à vos applications. Vous pouvez ajouter des fonctionnalités cognitives à vos applications sans avoir de compétences en matière de science des données ou d’intelligence artificielle (IA). Azure Cognitive Services comprend différents services d’IA qui vous permettent de créer des solutions cognitives pouvant voir, entendre, parler, comprendre et même prendre des décisions.

## <a name="categories-of-cognitive-services"></a>Catégories de services cognitifs

Le catalogue des services cognitifs fournissant une compréhension cognitive est classé en cinq piliers principaux :

* Vision
* Speech
* Langage
* Décision
* Recherche

Les sections suivantes de cet article fournissent la liste des services qui font partie de ces cinq piliers.

## <a name="vision-apis"></a>Les API Vision

|Nom du service|Description du service|
|:-----------|:------------------|
|[Vision par ordinateur](./computer-vision/index.yml "Vision par ordinateur")|Le service Vision par ordinateur vous donne accès à des algorithmes cognitifs avancés pour le traitement d’images et le retour d’informations. Consultez le [Guide de démarrage rapide de la Vision par ordinateur](./computer-vision/quickstarts-sdk/client-library.md) pour commencer à utiliser le service.|
|[Service Custom Vision](./custom-vision-service/index.yml "Service Vision personnalisée")|Le service Custom Vision vous permet de créer, de déployer et d’améliorer vos propres classifieurs d’images. Un classifieur d’images est un service d’IA qui applique des étiquettes à des images sur la base de leurs caractéristiques visuelles. |
|[Visage](./face/index.yml "Face")| Le service Visage donne accès à des algorithmes d’analyse du visage permettant la détection et la reconnaissance d’attributs faciaux. Consultez le [Guide de démarrage rapide de Visage](./face/quickstarts/client-libraries.md) pour commencer à utiliser le service.|

## <a name="speech-apis"></a>Les API Speech

|Nom du service|Description du service|
|:-----------|:------------------|
|[Service Speech](./speech-service/index.yml "Service Speech")|Le service Speech ajoute des fonctionnalités vocales aux applications. Le service Speech comprend diverses fonctionnalités telles que la reconnaissance vocale, la synthèse vocale, la traduction vocale et bien plus encore.|
<!--
|[Speaker Recognition API](./speech-service/speaker-recognition-overview.md "Speaker Recognition API") (Preview)|The Speaker Recognition API provides algorithms for speaker identification and verification.|
|[Bing Speech](./speech-service/how-to-migrate-from-bing-speech.md "Bing Speech") (Retiring)|The Bing Speech API provides you with an easy way to create speech-enabled features in your applications.|
|[Translator Speech](/azure/cognitive-services/translator-speech/ "Translator Speech") (Retiring)|Translator Speech is a machine translation service.|
-->
## <a name="language-apis"></a>Les API Langage

|Nom du service|Description du service|
|:-----------|:------------------|
|[Language Understanding (LUIS)](./luis/index.yml "Language Understanding")|Language Understanding (LUIS) est un service d’IA conversationnelle qui applique une intelligence de machine learning personnalisée au texte en langage naturel des conversations d’un utilisateur afin d’en prédire le sens général et d’en extraire des informations détaillées pertinentes. Consultez le [Guide de démarrage rapide de LUIS](./luis/luis-get-started-create-app.md) pour commencer à utiliser le service.|
|[QnA Maker](./qnamaker/index.yml "QnA Maker")|QnA Maker vous permet de générer un service de questions et réponses à partir de votre contenu semi-structuré. Consultez le [Guide de démarrage rapide de QnA Maker](./qnamaker/quickstarts/create-publish-knowledge-base.md) pour commencer à utiliser le service.|
|[Analyse de texte](./text-analytics/index.yml "Analyse de texte")| Analyse de texte effectue un traitement en langage naturel sur du texte brut pour l’analyse des sentiments, l’extraction de phrases clés et la détection de la langue. Consultez le [Guide de démarrage rapide d’Analyse de texte](./text-analytics/quickstarts/client-libraries-rest-api.md) pour commencer à utiliser le service.|
|[Translator](./translator/index.yml "Convertisseur")|Translator fournit une traduction de texte basée sur l’ordinateur quasiment en temps réel.|

## <a name="decision-apis"></a>API de décision

|Nom du service|Description du service|
|:-----------|:------------------|
|[Détecteur d’anomalies](./anomaly-detector/index.yml "Le détecteur d’anomalies") |Le Détecteur d’anomalies vous permet de surveiller et de détecter des anomalies dans vos données de série chronologique. Consultez le [Guide de démarrage rapide du Détecteur d’anomalies](./anomaly-detector/quickstarts/client-libraries.md) pour commencer à utiliser le service.|
|[Content Moderator](./content-moderator/overview.md "Content Moderator")|Content Moderator assure la surveillance de contenu potentiellement choquant, indésirable ou à risque. Consultez le [Guide de démarrage rapide de Content Moderator](./content-moderator/client-libraries.md) pour commencer à utiliser le service.|
|[Personalizer](./personalizer/index.yml "Personalizer")|Personalizer vous permet de choisir la meilleure expérience à montrer à vos utilisateurs, en apprenant de leur comportement en temps réel. Consultez le [Guide de démarrage rapide de Personalizer](./personalizer/quickstart-personalizer-sdk.md) pour commencer à utiliser le service.|

## <a name="search-apis"></a>API de recherche

> [!NOTE]
> Où trouver la [Recherche cognitive Azure](../search/index.yml) ? Bien qu’elle ait recours à Cognitive Services pour certaines tâches, il s’agit d’une technologie de recherche différente qui prend en charge d’autres scénarios.

|Nom du service|Description du service|
|:-----------|:------------------|
|[Recherche d’actualités Bing](/azure/cognitive-services/bing-news-search/ "Recherche Bing Actualités")|La Recherche d’actualités Bing retourne une liste d’articles d’actualités jugés pertinents en réponse à la requête de l’utilisateur.|
|[Recherche de vidéos Bing](/azure/cognitive-services/Bing-Video-Search/ "Recherche de vidéos Bing")|La Recherche de vidéos Bing retourne une liste de vidéos jugées pertinentes en réponse à la requête de l’utilisateur.|
|[Recherche Web Bing](./bing-web-search/index.yml "Recherche web Bing")|La Recherche Web Bing retourne une liste de résultats de recherche jugés pertinentes en réponse à la requête de l’utilisateur.|
|[Suggestion automatique Bing](/azure/cognitive-services/Bing-Autosuggest "Suggestion automatique Bing")|La Suggestion automatique Bing vous permet d’envoyer à Bing un terme de requête de recherche partielle et d’obtenir une liste de requêtes suggérées.|
|[Recherche personnalisée Bing](/azure/cognitive-services/bing-custom-search "Recherche personnalisée Bing")|La Recherche personnalisée Bing vous permet de créer des expériences de recherche sur mesure pour des sujets qui vous intéressent.|
|[Recherche d’entités Bing](/azure/cognitive-services/bing-entities-search/ "Recherche d’entité Bing")|La Recherche d’entités Bing retourne des informations sur des entités que Bing juge pertinentes en lien avec la requête d’un utilisateur.|
|[Recherche d’images Bing](/azure/cognitive-services/bing-image-search "Recherche d’images Bing")|La Recherche d’images Bing affiche des images jugées pertinentes en lien avec la requête d’un utilisateur.|
|[Recherche visuelle Bing](/azure/cognitive-services/bing-visual-search "Recherche visuelle Bing")|La Recherche visuelle Bing retourne des informations en relation avec une image, telles que des images visuellement similaires, des sources d’approvisionnement où acheter des produits illustrés dans l’image, et des recherches associées.|
|[Recherche d’entreprises locales Bing](/azure/cognitive-services/bing-local-business-search/ "La Recherche d’entreprises locales Bing")| L’API Recherche d’entreprises locales Bing permet à vos applications de rechercher les coordonnées d’entreprises à proximité à l’aide de requêtes.|
|[Vérification orthographique Bing](/azure/cognitive-services/bing-spell-check/ "Vérification orthographique Bing")|La Vérification orthographique Bing vous permet d’effectuer une vérification grammaticale et orthographique contextuelle.|

## <a name="get-started-with-cognitive-services"></a>Prise en main de Cognitive Services

Commencez par créer une ressource Cognitive Services avec des guides pratiques de démarrage rapide en utilisant les méthodes suivantes :

* [Azure portal](cognitive-services-apis-create-account.md?tabs=multiservice%2Cwindows "Portail Azure")
* [Azure CLI](cognitive-services-apis-create-account-cli.md?tabs=windows "Azure CLI")
* [Bibliothèques clientes du kit SDK Azure](cognitive-services-apis-create-account-cli.md?tabs=windows "cognitive-services-apis-create-account-client-library?pivots=programming-language-csharp")
* [Modèles Azure Resource Manager (ARM)](./create-account-resource-manager-template.md?tabs=portal "Modèles Azure Resource Manager (ARM)")

## <a name="using-cognitive-services-in-different-development-environments"></a>Utilisation de Cognitive Services dans différents environnements de développement

Avec Azure et Cognitive Services, vous avez accès à diverses options de développement, dont les suivantes :

* Outils d’automatisation et d’intégration tels que Logic Apps et Power Automate
* Options de déploiement telles qu’Azure Functions et App Service 
* Conteneurs Docker Cognitive Services pour un accès sécurisé
* Des outils comme Apache Spark, Azure Databricks, Azure Synapse Analytics et Azure Kubernetes Service pour les scénarios de Big Data 

Pour plus d’informations, consultez [Options de développement Cognitive Services](./cognitive-services-development-options.md).

<!--
## Subscription management

Once you are signed in with your Microsoft Account, you can access [My subscriptions](https://www.microsoft.com/cognitive-services/subscriptions "My subscriptions") to show the products you are using, the quota remaining, and the ability to add additional products to your subscription.

## Upgrade to unlock higher limits

All APIs have a free tier, which has usage and throughput limits.  You can increase these limits by using a paid offering and selecting the appropriate pricing tier option when deploying the service in the Azure portal. [Learn more about the offerings and pricing](https://azure.microsoft.com/pricing/details/cognitive-services/ "offerings and pricing"). You'll need to set up an Azure subscriber account with a credit card and a phone number. If you have a special requirement or simply want to talk to sales, click "Contact us" button at the top the pricing page.
-->

## <a name="using-cognitive-services-securely"></a>Utilisation de Cognitive Services de manière sécurisée

Azure Cognitive Services propose un modèle de sécurité en couches, notamment une [authentification](authentication.md "Authentification") via les informations d’identification Azure Active Directory, une clé de ressource valide et les [réseaux virtuels Azure](cognitive-services-virtual-networks.md "Réseaux virtuels Azure").

## <a name="containers-for-cognitive-services"></a>Conteneurs pour Cognitive Services

 Azure Cognitive Services offre plusieurs conteneurs Docker qui vous permettent d’utiliser localement les mêmes API que celles disponibles dans Azure. Grâce à ces conteneurs, Cognitive Services se trouve au plus près de vos données du point de vue de la conformité, de la sécurité et de l’exploitation. En savoir plus sur les [Conteneurs Cognitive Services](cognitive-services-container-support.md "Conteneurs Cognitive Services").

## <a name="regional-availability"></a>Disponibilité régionale

Les API figurant dans Cognitive Services sont hébergées sur un réseau croissant de centres de données Azure gérés par Microsoft. Vous trouverez la disponibilité régionale de chaque API dans la [liste des régions Azure](https://azure.microsoft.com/regions "Liste des régions Azure").

À la recherche d’une région encore non prise en charge ? Faites-le nous savoir en créant une demande de fonctionnalité sur notre [forum UserVoice](https://feedback.azure.com/forums/932041-azure-cognitive-services "Forum UserVoice").

## <a name="supported-cultural-languages"></a>Langues prises en charge

Cognitive Services prend en charge un large éventail de langues au niveau du service. Vous trouverez les langues disponibles pour chaque API dans la [liste des langues prises en charge](language-support.md "liste des langages pris en charge").

## <a name="certifications-and-compliance"></a>Certifications et conformité

Cognitive Services a reçu des certifications telles que la Certification de CSA STAR, FedRAMP Moderate et HIPAA BAA. Vous pouvez [télécharger](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942 "télécharger") des certifications pour vos propres audits et révisions de sécurité.

Pour comprendre la gestion de la confidentialité et des données, accédez au [Centre de gestion de la confidentialité](https://servicetrust.microsoft.com/ "Centre de gestion de la confidentialité").

## <a name="support"></a>Support

Cognitive Services propose plusieurs options de support pour vous aider à créer des applications intelligentes. Cognitive Services a également une communauté forte de développeurs qui peuvent vous aider à répondre à vos questions spécifiques. Pour obtenir la liste complète des options à votre disposition, consultez [Options d’aide et de support Cognitive Services](cognitive-services-support-options.md "Options d’aide et de support Cognitive Services").

## <a name="next-steps"></a>Étapes suivantes

* [Créer un compte Cognitive Services](cognitive-services-apis-create-account.md "Créer un compte Cognitive Services")
* [Documentation sur les nouveautés de Cognitive Services](whats-new-docs.md "Documentation sur les nouveautés de Cognitive Services")
* [Planifier et gérer les coûts pour Cognitive Services](plan-manage-costs.md)
