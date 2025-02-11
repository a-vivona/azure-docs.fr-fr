---
title: Migrer depuis Indexer v1 et v2 vers Azure Media Services Video Indexer | Microsoft Docs
description: Cette rubrique explique comment effectuer une migration depuis Azure Media Indexer v1 et v2 vers Azure Media Services Video Indexer.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/26/2021
ms.author: inhenkel
ms.openlocfilehash: 3a2871bd42e6b4da9ec20dc918c5c0ab79ed1a64
ms.sourcegitcommit: bb1c13bdec18079aec868c3a5e8b33ef73200592
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/27/2021
ms.locfileid: "114721564"
---
# <a name="migrate-from-media-indexer-and-media-indexer-2-to-video-analyzer-for-media"></a>Migrer depuis Media Indexer et Media Indexer 2 vers Video Analyzer for Media 

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!IMPORTANT]
> Il est recommandé aux clients de procéder à une migration de la version 1 ou 2 d'Indexer vers l'utilisation du [mode De base de Media Services V3 AudioAnalyzerPreset](../latest/analyze-video-audio-files-concept.md). Les processeurs multimédias [Azure Media Indexer](media-services-index-content.md) et [Azure Media Indexer 2 Preview](./legacy-components.md) sont en phase de mise hors service. Pour connaître les dates de mise hors service, consultez la rubrique [Composants hérités](legacy-components.md).

La solution Azure Video Analyzer for Media est basée sur Azure Media Analytics, la Recherche cognitive et Cognitive Services (par exemple, l’API Visage, Microsoft Translator, l’API Vision par ordinateur et Custom Speech Service). Elle vous permet d’extraire les insights de vos vidéos à l’aide de modèles vidéo et audio Video Analyzer for Media. Pour connaître les scénarios dans lesquels Video Analyzer for Media peut être utilisé, les fonctionnalités qu’il propose et comment le prendre en main, consultez [Modèles vidéo et audio Video Analyzer for Media](../../azure-video-analyzer/video-analyzer-for-media-docs/video-indexer-overview.md). 

Vous pouvez extraire les insights de vos fichiers vidéo et audio à l’aide des [présélections de l’analyseur d’Azure Media Services v3](../latest/analyze-video-audio-files-concept.md) ou directement en utilisant les [API Video Analyzer for Media](https://api-portal.videoindexer.ai/). Certaines fonctionnalités actuellement proposées par les API Video Analyzer for Media et les API Media Services v3 se chevauchent.

> [!NOTE]
> Pour comprendre les différences entre Video Analyzer for Media et les présélections de l’analyseur de médias et de Media Services, consultez le [document de comparaison](../../azure-video-analyzer/video-analyzer-for-media-docs/compare-video-indexer-with-media-services-presets.md).

Cet article décrit les étapes de la migration depuis Azure Media Indexer et Azure Media Indexer 2 vers Video Analyzer for Media.  

## <a name="migration-options"></a>Options de migration

|Si vous avez besoin  |, puis |
|---|---|
|D’une solution qui fournit une transcription de la voix en texte pour n’importe quel format de fichier multimédia dans un format de fichier de sous-titres : VTT, SRT ou TTML<br/>Ainsi que d’insights audio supplémentaires, tels que : mots clés, inférence de rubrique, événements acoustiques, diarisation de l’orateur, extraction et traduction d’entités| mettez à jour vos applications pour utiliser les fonctionnalités Video Analyzer for Media par le biais de l’API REST Video Analyzer for Media v2 ou la présélection de l’analyseur audio d’Azure Media Services v3.|
|Fonctionnalités de reconnaissance vocale| Utilisez l’API Speech Cognitive Services directement.|  

## <a name="getting-started-with-video-analyzer-for-media"></a>Bien démarrer avec Video Analyzer for Media

La section suivante vous indique des liens utiles : [Comment bien démarrer Video Analyzer for Media ?](../../azure-video-analyzer/video-analyzer-for-media-docs/video-indexer-overview.md#how-can-i-get-started-with-video-analyzer-for-media) 

## <a name="getting-started-with-media-services-v3-apis"></a>Bien démarrer avec les API Media Services v3

L’API Azure Media Services v3 vous permet d’extraire les insights de vos fichiers vidéo et audio par le biais des [présélections de l’analyseur d’Azure Media Services v3](../latest/analyze-video-audio-files-concept.md).

**AudioAnalyzerPreset** vous permet d’extraire plusieurs insights audio d’un fichier audio ou vidéo. La sortie inclut un fichier VTT ou TTML pour la transcription audio et un fichier JSON (avec tous les insights audio supplémentaires). Les insights audio incluent les mots clés, l’indexation de l’orateur et l’analyse du sentiment vocal. AudioAnalyzerPreset prend également en charge la détection de la langue pour des langues spécifiques. Pour plus d’informations, consultez [Transformations](/rest/api/media/transforms/createorupdate#audioanalyzerpreset).

### <a name="get-started"></a>Bien démarrer

Pour commencer, consultez :

* [Didacticiel](../latest/analyze-videos-tutorial.md)
* Exemples AudioAnalyzerPreset : [SDK Java](https://github.com/Azure-Samples/media-services-v3-java/tree/master/AudioAnalytics/AudioAnalyzer) ou [SDK .NET](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/master/AudioAnalytics/AudioAnalyzer)
* Exemples VideoAnalyzerPreset : [SDK Java](https://github.com/Azure-Samples/media-services-v3-java/tree/master/VideoAnalytics/VideoAnalyzer) ou [SDK .NET](https://github.com/Azure-Samples/media-services-v3-dotnet/tree/master/VideoAnalytics/VideoAnalyzer)

## <a name="getting-started-with-cognitive-services-speech-services"></a>Bien démarrer avec le service Cognitive Services Speech Services

[Azure Cognitive Services](../../cognitive-services/index.yml) fournit un service de reconnaissance vocale qui transcrit en temps réel des flux audio en texte que vos applications, outils ou appareils peuvent consommer ou afficher. Vous pouvez utiliser la reconnaissance vocale pour [personnaliser votre propre modèle acoustique, de langage ou de prononciation](../../cognitive-services/speech-service/how-to-custom-speech-train-model.md). Pour plus d’informations, consultez [Reconnaissance vocale Cognitive Services](../../cognitive-services/speech-service/speech-to-text.md). 

> [!NOTE] 
> Le service de reconnaissance vocale n’accepte pas les formats de fichier vidéo ; il accepte uniquement [certains formats audio](../../cognitive-services/speech-service/rest-speech-to-text.md#audio-formats). 

Pour plus d’informations sur le service de reconnaissance vocale et sa prise en main, consultez [Qu’est-ce que la reconnaissance vocale ?](../../cognitive-services/speech-service/speech-to-text.md).

## <a name="known-differences-from-deprecated-services"></a>Différences connues par rapport aux services dépréciés

Vous constaterez que les services Video Analyzer for Media, AudioAnalyzerPreset d’Azure Media Services v3 et Cognitive Services Speech Services sont plus fiables et produisent une sortie de qualité supérieure à celle des processeurs Azure Media Indexer 1 et Azure Media Indexer 2 mis hors service.  

Voici certaines différences connues :

* Le service Cognitive Services Speech Services ne prend pas en charge l’extraction de mots clés. Toutefois, Video Analyzer for Media et AudioAnalyzerPreset de Media Services V3 offrent tous deux un ensemble de mots clés plus robuste au format de fichier JSON.

## <a name="support"></a>Support

Vous pouvez ouvrir un ticket de support en accédant à [Nouvelle demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)

## <a name="next-steps"></a>Étapes suivantes

* [Composants hérités](legacy-components.md)
* [Page des tarifs](https://azure.microsoft.com/pricing/details/media-services/#encoding)
