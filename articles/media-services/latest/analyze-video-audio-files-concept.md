---
title: Analyser des fichiers vidéo et audio
description: Découvrez comment analyser du contenu audio et vidéo à l’aide d’AudioAnalyzerPreset et de VideoAnalyzerPreset dans Azure Media Services.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.date: 07/26/2021
ms.author: inhenkel
ms.openlocfilehash: a9a101afad5bfcdd8ca7ec20bc39b3080790a09b
ms.sourcegitcommit: bb1c13bdec18079aec868c3a5e8b33ef73200592
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/27/2021
ms.locfileid: "114720773"
---
# <a name="analyze-video-and-audio-files-with-azure-media-services"></a>Analyser des fichiers vidéo et audio avec Azure Media Services

[!INCLUDE [regulation](../../azure-video-analyzer/video-analyzer-for-media-docs/includes/regulation.md)]

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services v3 vous permet d’extraire les insights de vos fichiers vidéo et audio avec Azure Video Analyzer for Media (anciennement Video Indexer). Cet article décrit les présélections de l’analyseur Media Services V3 utilisées pour extraire ces Insights. Si vous souhaitez des insights plus détaillés, utilisez directement Video Analyzer for Media. Pour comprendre à quel moment utiliser Video Analyzer for Media plutôt que les présélections de l’analyseur de Media Services, consultez le [document de comparaison](../../azure-video-analyzer/video-analyzer-for-media-docs/compare-video-indexer-with-media-services-presets.md).

Il existe deux modes pour le préréglage de l’analyseur audio, de base et standard. Consultez la description des différences dans le tableau ci-dessous.

Pour analyser votre contenu à l’aide des préréglages Media Services v3, vous créez une **transformation** et envoyez un **travail** qui utilise l’un de ces préréglages : [VideoAnalyzerPreset](/rest/api/media/transforms/createorupdate#videoanalyzerpreset) ou **AudioAnalyzerPreset**. Pour un didacticiel présentant comment utiliser **VideoAnalyzerPreset**, consultez [Analyser des vidéos avec Azure Media Services](analyze-videos-tutorial.md).

## <a name="compliance-privacy-and-security"></a>Conformité, confidentialité et sécurité

Rappelez-vous que vous devez respecter toute la réglementation applicable dans le cadre de votre utilisation de Video Analyzer for Media, et que vous n’êtes pas autorisé à l’utiliser ou à utiliser tout autre service Azure d’une façon qui porte atteinte aux droits d’autrui ou qui soit préjudiciable pour autrui. Avant de charger des vidéos, en particulier des données biométriques, sur le service Video Analyzer for Media à des fins de traitement et de stockage, vous devez disposer de tous les droits appropriés sur ces vidéos, notamment le consentement des personnes qui y figurent. Pour plus d’informations sur la conformité, la confidentialité et la sécurité dans Video Indexer for Media, consultez les [Conditions générales d’utilisation de Cognitive Services](https://azure.microsoft.com/support/legal/cognitive-services-compliance-and-privacy/). Pour connaître les obligations de Microsoft en matière de confidentialité et de traitement de vos données, consultez la [Déclaration de confidentialité](https://privacy.microsoft.com/PrivacyStatement), les [Conditions d’utilisation des services en ligne](https://www.microsoft.com/licensing/product-licensing/products) (« OST ») et l’[Addenda au traitement des données](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) (« DPA ») de Microsoft. Des informations complémentaires sur la confidentialité, notamment sur la conservation, la suppression et la destruction des données, sont disponibles dans l’OST et [ici](../../azure-video-analyzer/video-analyzer-for-media-docs/faq.md). En utilisant Video Analyzer for Media, vous acceptez de vous conformer aux Conditions générales de Cognitive Services, à l’OST, au DPA et à la Déclaration de confidentialité.

## <a name="built-in-presets"></a>Préréglages intégrés

Actuellement, Media Services prend en charge les préréglages d’analyseur intégrés suivants :  

|**Nom du préréglage**|**Scénario/Mode**|**Détails**|
|---|---|---|
|[AudioAnalyzerPreset](/rest/api/media/transforms/createorupdate#audioanalyzerpreset)|Analyse du mode audio Standard|Ce préréglage applique un ensemble prédéfini d’opérations d’analyse basée sur l’IA, notamment la transcription de la parole. Actuellement, le préréglage prend en charge le traitement du contenu avec une seule piste audio qui inclut la reconnaissance vocale dans une seule langue. Vous pouvez spécifier la langue de la charge utile audio de l’entrée en utilisant le format BCP-47 « balise de langue-région ». Langues prises en charge : anglais (« en-US », « en-GB » et « en-AU »), espagnol (« es-ES » et « es-MX »), français (« fr-FR » et « fr-CA »), italien (« it-IT »), japonais (« ja-JP »), portugais (« pt-BR »), chinois (« zh-CN »), allemand (« de-DE »), arabe (« ar-BH », « ar-EG », « ar-IQ », « ar-JO », « ar-KW », « ar-LB », « ar-OM », « ar-QA », « ar-SA » et « ar-SY »), russe (« ru-RU »), hindi (« hi-IN »), coréen (« ko-KR »), danois (« da-DK »), norvégien (« nb-NO »), suédois (« sv-SE »), finnois (« fi-FI »), thaï (« th-TH ») et turc (« tr-TR »).<br/><br/> Si la langue n’est pas spécifiée ou a la valeur Null, la fonctionnalité de détection automatique de la langue choisit la première langue détectée et continue avec cette langue pendant la durée de traitement du fichier. Cette fonctionnalité prend actuellement en charge les langues suivantes : allemand, anglais, chinois, espagnol, français, italien, japonais, portugais et russe. Elle ne prend pas en charge le basculement dynamique d’une langue à l’autre après la détection de la première langue. La fonctionnalité de détection automatique de la langue fonctionne mieux sur des enregistrements audio avec des voix clairement identifiables. Si la détection automatique de la langue ne parvient pas à trouver la langue, la transcription utilise l’anglais.|
|[AudioAnalyzerPreset](/rest/api/media/transforms/createorupdate#audioanalyzerpreset)|Analyse du mode audio De base|Ce mode prédéfini effectue une transcription de la parole en texte et la génération d’un fichier de sous-titres/CC au format VTT. La sortie de ce mode comprend un fichier JSON Insights incluant uniquement les mots clés, la transcription et les informations relatives au minutage. La détection automatique de la langue et la diarisation de l’orateur ne sont pas incluses dans ce mode. La liste des langues prises en charge est identique à celle du mode Standard ci-dessus.|
|[VideoAnalyzerPreset](/rest/api/media/transforms/createorupdate#videoanalyzerpreset)|Analyse de contenu audio et vidéo|Extrait des insights (métadonnées enrichies) des contenus audio et vidéo, et génère en sortie un fichier au format JSON. Vous pouvez spécifier si vous voulez extraire seulement des insights audio lors du traitement d’un fichier vidéo. Pour plus d’informations, consultez [Analyser un contenu vidéo](analyze-videos-tutorial.md).|
|[FaceDetectorPreset](/rest/api/media/transforms/createorupdate#facedetectorpreset)|Détection des visages présents dans la vidéo|Décrit les paramètres à utiliser lors de l’analyse d’une vidéo pour détecter les visages qui y figurent.|

### <a name="audioanalyzerpreset-standard-mode"></a>Mode standard AudioAnalyzerPreset

Le préréglage vous permet d’extraire plusieurs insights audio d’un fichier audio ou vidéo.

La sortie inclut un fichier JSON (avec tous les insights) et un fichier VTT pour la transcription audio. Ce paramètre accepte une propriété qui spécifie la langue du fichier d’entrée sous la forme d’une chaîne [BCP47](https://tools.ietf.org/html/bcp47). Les analyses audio sont les suivantes :

* **Transcription audio** : transcription des mots prononcés avec horodatages. Plusieurs langues sont prises en charge.
* **Indexation de l'orateur** : mappage des orateurs et des mots prononcés correspondants.
* **Analyse des sentiments dans du texte** : sortie de l’analyse des sentiments effectuée sur la transcription audio.
* **Mots clés** : mots clés extraits de la transcription audio.

### <a name="audioanalyzerpreset-basic-mode"></a>Mode de base AudioAnalyzerPreset

Le préréglage vous permet d’extraire plusieurs insights audio d’un fichier audio ou vidéo.

La sortie inclut un fichier JSON et un fichier VTT pour la transcription audio. Ce paramètre accepte une propriété qui spécifie la langue du fichier d’entrée sous la forme d’une chaîne [BCP47](https://tools.ietf.org/html/bcp47). La sortie comprend les éléments suivants :

* **Transcription audio** : transcription des mots prononcés avec horodatages. Plusieurs langues sont prises en charge, mais la détection automatique de la langue et la diarisation de l’orateur ne sont pas incluses.
* **Mots clés** : mots clés extraits de la transcription audio.

### <a name="videoanalyzerpreset"></a>VideoAnalyzerPreset

Ce préréglage vous permet d’extraire plusieurs insights audio et vidéo à partir d’un fichier vidéo. La sortie inclut un fichier JSON (avec tous les insights), un fichier VTT pour la transcription audio et une collection de miniatures. Ce paramètre accepte également une chaîne [BCP47](https://tools.ietf.org/html/bcp47) (représentant la langue de la vidéo) en tant que propriété. Les insights vidéo incluent tous les insights audio mentionnés ci-dessus en complément des éléments suivants :

* **Suivi du visage** : durée pendant laquelle des visages sont présentes dans la vidéo. Chaque visage est associé à un identifiant de visage et à une collection de miniatures correspondante.
* **Texte visuel** : texte détecté par la reconnaissance optique des caractères. Le texte est horodaté et également utilisé pour extraire des mots clés (en plus de la transcription audio).
* **Images clés** : une collection d’images clés extraites de la vidéo.
* **Modération du contenu visuel** : La partie des vidéos marquée comme adulte ou osé par nature.
* **Annotation** : résultat de l’annotation des vidéos sur la base d’un modèle d’objet prédéfini

## <a name="insightsjson-elements"></a>Éléments insights.json

La sortie inclut un fichier JSON (insights.json) contenant tous les insights trouvés dans le contenu vidéo ou audio. Ce fichier JSON peut contenir les éléments suivants :

### <a name="transcript"></a>transcription

|Nom|Description|
|---|---|
|id|ID de la ligne.|
|text|La transcription proprement dite.|
|langage|La langue de la transcription. Permet de prendre en charge la transcription lorsque chaque ligne peut avoir une langue différente.|
|instances|Liste des intervalles de temps pendant lesquels cette ligne est apparue. Si l’instance est un attribut transcript, il n’y a qu’une seule instance.|

Exemple :

```json
"transcript": [
{
    "id": 0,
    "text": "Hi I'm Doug from office.",
    "language": "en-US",
    "instances": [
    {
        "start": "00:00:00.5100000",
        "end": "00:00:02.7200000"
    }
    ]
},
{
    "id": 1,
    "text": "I have a guest. It's Michelle.",
    "language": "en-US",
    "instances": [
    {
        "start": "00:00:02.7200000",
        "end": "00:00:03.9600000"
    }
    ]
}
] 
```

### <a name="ocr"></a>ocr

|Nom|Description|
|---|---|
|id|ID de la ligne ROC.|
|text|Texte de l’OCR.|
|confidence|Degré de confiance de la reconnaissance.|
|langage|Langue de l’OCR.|
|instances|Liste des intervalles de temps au cours desquels cette OCR est apparue (la même OCR peut apparaître plusieurs fois).|

```json
"ocr": [
    {
      "id": 0,
      "text": "LIVE FROM NEW YORK",
      "confidence": 0.91,
      "language": "en-US",
      "instances": [
        {
          "start": "00:00:26",
          "end": "00:00:52"
        }
      ]
    },
    {
      "id": 1,
      "text": "NOTICIAS EN VIVO",
      "confidence": 0.9,
      "language": "es-ES",
      "instances": [
        {
          "start": "00:00:26",
          "end": "00:00:28"
        },
        {
          "start": "00:00:32",
          "end": "00:00:38"
        }
      ]
    }
  ],
```

### <a name="faces"></a>visages

|Nom|Description|
|---|---|
|id|ID du visage.|
|name|Nom du visage. Il peut s'agir de la valeur « Unknown #0 », d’une célébrité identifiée ou d'une personne formée par le client.|
|confidence|Degré de confiance de l’identification du visage.|
|description|Description de la célébrité. |
|thumbnailId|ID de la miniature de ce visage.|
|knownPersonId|ID interne (s’il s’agit d’une personne connue).|
|referenceId|ID Bing (s’il s’agit d’une célébrité Bing).|
|referenceType|Bing uniquement (pour le moment).|
|title|Poste (dans le cas d’une célébrité, par exemple « PDG de Microsoft »).|
|imageUrl|URL de l’image, s’il s’agit d’une célébrité.|
|instances|Instances où le visage est apparu dans l’intervalle de temps donné. Chaque instance possède également un thumbnailsId. |

```json
"faces": [{
    "id": 2002,
    "name": "Xam 007",
    "confidence": 0.93844,
    "description": null,
    "thumbnailId": "00000000-aee4-4be2-a4d5-d01817c07955",
    "knownPersonId": "8340004b-5cf5-4611-9cc4-3b13cca10634",
    "referenceId": null,
    "title": null,
    "imageUrl": null,
    "instances": [{
        "thumbnailsIds": ["00000000-9f68-4bb2-ab27-3b4d9f2d998e",
        "cef03f24-b0c7-4145-94d4-a84f81bb588c"],
        "adjustedStart": "00:00:07.2400000",
        "adjustedEnd": "00:00:45.6780000",
        "start": "00:00:07.2400000",
        "end": "00:00:45.6780000"
    },
    {
        "thumbnailsIds": ["00000000-51e5-4260-91a5-890fa05c68b0"],
        "adjustedStart": "00:10:23.9570000",
        "adjustedEnd": "00:10:39.2390000",
        "start": "00:10:23.9570000",
        "end": "00:10:39.2390000"
    }]
}]
```

### <a name="shots"></a>captures

|Nom|Description|
|---|---|
|id|ID de la capture.|
|keyFrames|Liste des images clés au sein de la capture (chacune possède un ID et une liste d’intervalles de temps d’instances). Les instances des images clés comptent un champ thumbnailId pourvu de l’ID de miniature de l’élément keyFrame.|
|instances|Liste des intervalles de temps de cette capture (les captures n’ont qu’1 seule instance).|

```json
"Shots": [
    {
      "id": 0,
      "keyFrames": [
        {
          "id": 0,
          "instances": [
            {
                "thumbnailId": "00000000-0000-0000-0000-000000000000",
              "start": "00: 00: 00.1670000",
              "end": "00: 00: 00.2000000"
            }
          ]
        }
      ],
      "instances": [
        {
            "thumbnailId": "00000000-0000-0000-0000-000000000000",  
          "start": "00: 00: 00.2000000",
          "end": "00: 00: 05.0330000"
        }
      ]
    },
    {
      "id": 1,
      "keyFrames": [
        {
          "id": 1,
          "instances": [
            {
                "thumbnailId": "00000000-0000-0000-0000-000000000000",      
              "start": "00: 00: 05.2670000",
              "end": "00: 00: 05.3000000"
            }
          ]
        }
      ],
      "instances": [
        {
          "thumbnailId": "00000000-0000-0000-0000-000000000000",
          "start": "00: 00: 05.2670000",
          "end": "00: 00: 10.3000000"
        }
      ]
    }
  ]
```

### <a name="statistics"></a>statistiques

|Nom|Description|
|---|---|
|CorrespondenceCount|Nombre de correspondances contenues dans la vidéo.|
|WordCount|Nombre de mots par intervenant.|
|SpeakerNumberOfFragments|Quantité de fragments de l’intervenant dans une vidéo.|
|SpeakerLongestMonolog|Monologue le plus long de l’intervenant. Si le monologue de l’intervenant comporte des silences, ils sont inclus. Les silences du début et de la fin du monologue sont supprimés.|
|SpeakerTalkToListenRatio|Le calcul est basé sur le temps passé sur le monologue de l’intervenant (sans les silences intermédiaires) divisé par la durée totale de la vidéo. L’heure est arrondie à la troisième décimale.|


### <a name="sentiments"></a>sentiments

Les sentiments sont regroupés par leur champ sentimentType (neutre/positif/négatif). Par exemple, 0-0.1, 0.1-0.2.

|Nom|Description|
|---|---|
|id|ID du sentiment.|
|averageScore |Moyenne de tous les résultats obtenus pour toutes les instances de ce type de sentiment : neutre/positif/négatif|
|instances|Liste des intervalles de temps au cours desquels ce sentiment est apparu.|
|sentimentType |Le type peut être « Positive », « Neutral » ou «Negative ».|

```json
"sentiments": [
{
    "id": 0,
    "averageScore": 0.87,
    "sentimentType": "Positive",
    "instances": [
    {
        "start": "00:00:23",
        "end": "00:00:41"
    }
    ]
}, {
    "id": 1,
    "averageScore": 0.11,
    "sentimentType": "Positive",
    "instances": [
    {
        "start": "00:00:13",
        "end": "00:00:21"
    }
    ]
}
]
```

### <a name="labels"></a>étiquettes

|Nom|Description|
|---|---|
|id|ID de l’étiquette.|
|name|Nom de l’étiquette (par exemple, « ordinateur », « TV »).|
|langage|Langue du nom de l’étiquette (si traduction). BCP-47|
|instances|Liste des intervalles de temps au cours desquels cette étiquette est apparue (une étiquette peut apparaître plusieurs fois). Chaque instance possède un champ de confiance. |

```json
"labels": [
    {
      "id": 0,
      "name": "person",
      "language": "en-US",
      "instances": [
        {
          "confidence": 1.0,
          "start": "00: 00: 00.0000000",
          "end": "00: 00: 25.6000000"
        },
        {
          "confidence": 1.0,
          "start": "00: 01: 33.8670000",
          "end": "00: 01: 39.2000000"
        }
      ]
    },
    {
      "name": "indoor",
      "language": "en-US",
      "id": 1,
      "instances": [
        {
          "confidence": 1.0,
          "start": "00: 00: 06.4000000",
          "end": "00: 00: 07.4670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 09.6000000",
          "end": "00: 00: 10.6670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 11.7330000",
          "end": "00: 00: 20.2670000"
        },
        {
          "confidence": 1.0,
          "start": "00: 00: 21.3330000",
          "end": "00: 00: 25.6000000"
        }
      ]
    }
  ] 
```

### <a name="keywords"></a>mots clés

|Nom|Description|
|---|---|
|id|ID du mot clé.|
|text|Texte du mot clé.|
|confidence|Degré de confiance de la reconnaissance du mot clé.|
|langage|Langue du mot clé (si traduction).|
|instances|Liste des intervalles de temps pendant lesquels ce mot clé est apparu (un mot clé peut apparaître plusieurs fois).|

```json
"keywords": [
{
    "id": 0,
    "text": "office",
    "confidence": 1.6666666666666667,
    "language": "en-US",
    "instances": [
    {
        "start": "00:00:00.5100000",
        "end": "00:00:02.7200000"
    },
    {
        "start": "00:00:03.9600000",
        "end": "00:00:12.2700000"
    }
    ]
},
{
    "id": 1,
    "text": "icons",
    "confidence": 1.4,
    "language": "en-US",
    "instances": [
    {
        "start": "00:00:03.9600000",
        "end": "00:00:12.2700000"
    },
    {
        "start": "00:00:13.9900000",
        "end": "00:00:15.6100000"
    }
    ]
}
] 
```

#### <a name="visualcontentmoderation"></a>visualContentModeration

Le bloc visualContentModeration contient des intervalles de temps qui sont susceptibles de contenir des éléments pour adultes selon Video Analyzer for Media. Si ce bloc est vide, aucun contenu pour adultes n’a donc été identifié.

Les vidéos trouvées qui contiennent des éléments pour adultes ou choquants peuvent être disponibles pour un affichage privé uniquement. Les utilisateurs peuvent soumettre une demande de révision manuelle du contenu, auquel cas l’attribut `IsAdult` contient le résultat de la révision manuelle.

|Nom|Description|
|---|---|
|id|ID de modération du contenu visuel.|
|adultScore|Degré du contenu pour adultes (d’après Content Moderator).|
|racyScore|Degré du contenu choquant (d’après Content Moderator).|
|instances|Liste des intervalles de temps où cette modération du contenu visuel est affichée.|

```json
"VisualContentModeration": [
{
    "id": 0,
    "adultScore": 0.00069,
    "racyScore": 0.91129,
    "instances": [
    {
        "start": "00:00:25.4840000",
        "end": "00:00:25.5260000"
    }
    ]
},
{
    "id": 1,
    "adultScore": 0.99231,
    "racyScore": 0.99912,
    "instances": [
    {
        "start": "00:00:35.5360000",
        "end": "00:00:35.5780000"
    }
    ]
}
] 
```
## <a name="next-steps"></a>Étapes suivantes

[Tutoriel : Analyser des vidéos avec Azure Media Services](analyze-videos-tutorial.md)
