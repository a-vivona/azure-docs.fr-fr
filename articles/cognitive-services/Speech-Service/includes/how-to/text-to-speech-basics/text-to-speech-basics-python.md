---
author: PatrickFarley
ms.service: cognitive-services
ms.topic: include
ms.date: 07/02/2021
ms.author: pafarley
ms.openlocfilehash: 3aa8b987c51691d9f13df83945e39931ae0743ca
ms.sourcegitcommit: f2d0e1e91a6c345858d3c21b387b15e3b1fa8b4c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/07/2021
ms.locfileid: "123539331"
---
Dans ce guide de démarrage rapide, vous découvrez les modèles de conception courants qui permettent d’utiliser la synthèse vocale au moyen du kit SDK Speech. Vous commencez par créer une configuration et une synthèse de base, puis passez à des exemples plus poussés en matière de développement d’applications personnalisées, notamment :

* Obtention de réponses en tant que flux en mémoire
* Personnalisation du taux d’échantillonnage de sortie et de la vitesse de transmission
* Envoi de demandes de synthèse à l’aide de SSML (Speech Synthesis Markup Language)
* Utilisation de voix neurales

## <a name="skip-to-samples-on-github"></a>Passer aux exemples sur GitHub

Si vous souhaitez passer directement à l’exemple de code, consultez les [exemples de démarrage rapide Python](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/python/text-to-speech) sur GitHub.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Avant de pouvoir faire quoi que ce soit, vous devez installer le SDK Speech.

```Python
pip install azure-cognitiveservices-speech
```

Si vous utilisez macOS et rencontrez des problèmes d’installation, vous devrez peut-être exécuter cette commande en premier.

```Python
python3 -m pip install --upgrade pip
```

Une fois le SDK Speech installé, incluez les instructions d’importation suivantes en haut de votre script.

```Python
from azure.cognitiveservices.speech import AudioDataStream, SpeechConfig, SpeechSynthesizer, SpeechSynthesisOutputFormat
from azure.cognitiveservices.speech.audio import AudioOutputConfig
```

## <a name="create-a-speech-configuration"></a>Créer une configuration Speech

Pour appeler le service Speech à l’aide du SDK Speech, vous devez créer une classe [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig). Celle-ci comprend des informations sur votre abonnement, telles que votre clé Speech et l’emplacement/la région, le point de terminaison, l’hôte ou le jeton d’autorisation associés.

> [!NOTE]
> Quand vous procédez à une reconnaissance vocale, une synthèse vocale, une traduction ou une reconnaissance intentionnelle, vous devez toujours créer une configuration.

Vous pouvez initialiser une [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig) de plusieurs façons :

* Avec un abonnement : transmettez une clé Speech et l’emplacement/la région qui lui correspond.
* Avec un point de terminaison : transmettez un point de terminaison de service Speech. Une clé Speech ou un jeton d’autorisation est facultatif.
* Avec un hôte : transmettez une adresse d’hôte. Une clé Speech ou un jeton d’autorisation est facultatif.
* Avec un jeton d’autorisation : transmettez un jeton d’autorisation et la localisation/la région qui lui correspond.

Dans cet exemple, vous créez un [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig) à l’aide d’une clé Speech et d’un emplacement/d’une région. Pour obtenir ces informations d’identification, suivez les étapes indiquées dans [Essayer le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

```python
speech_config = SpeechConfig(subscription="<paste-your-speech-key-here>", region="<paste-your-speech-location/region-here>")
```

## <a name="synthesize-speech-to-a-file"></a>Synthétiser la voix dans un fichier

Ensuite, vous allez créer un objet [`SpeechSynthesizer`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesizer), qui exécute des conversions de synthèse vocale et des sorties vers des haut-parleurs, des fichiers ou d’autres flux de sortie. [`SpeechSynthesizer`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesizer) accepte en tant que paramètres l’objet [`SpeechConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig) créé à l’étape précédente et un objet [`AudioOutputConfig`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.audiooutputconfig) qui spécifie la façon dont les résultats de sortie doivent être gérés.

Pour commencer, créez un `AudioOutputConfig` pour écrire automatiquement la sortie dans un fichier `.wav` à l’aide du paramètre de constructeur `filename`.

```python
audio_config = AudioOutputConfig(filename="path/to/write/file.wav")
```

Ensuite, instanciez un `SpeechSynthesizer` en transmettant votre objet `speech_config` et l’objet `audio_config` en tant que paramètres. Dès lors, exécuter la synthèse vocale et écrire dans un fichier sont aussi simples que d’exécuter `speak_text_async()` avec une chaîne de texte.

```python
synthesizer = SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)
synthesizer.speak_text_async("A simple test to write to a file.")
```

Exécutez le programme ; un fichier `.wav` synthétisé est alors écrit à l’emplacement que vous avez spécifié. Il s’agit d’un bon exemple de l’utilisation la plus simple, mais vous allez maintenant vous pencher sur la personnalisation de la sortie et sur la gestion de la réponse de sortie sous forme de flux en mémoire pour travailler sur des scénarios personnalisés.

## <a name="synthesize-to-speaker-output"></a>Synthétiser vers la sortie du haut-parleur

Dans certains cas, vous pouvez souhaiter sortir directement la synthèse vocale vers un haut-parleur. Pour cela, utilisez l’exemple de la section précédente, sauf que vous allez modifier `AudioOutputConfig` en supprimant le paramètre `filename` et définir `use_default_speaker=True`. La sortie est alors dirigée vers l’appareil de sortie actuellement actif.

```python
audio_config = AudioOutputConfig(use_default_speaker=True)
```

## <a name="get-result-as-an-in-memory-stream"></a>Obtenir le résultat sous forme de flux en mémoire

Dans de nombreux scénarios de développement d’application vocale, vous aurez probablement besoin des données audio obtenues sous forme de flux en mémoire plutôt que directement écrites dans un fichier. Cela vous permettra de générer un comportement personnalisé, notamment :

* Résumer le tableau d’octets obtenu sous forme de flux pouvant faire l’objet de recherches pour des services en aval personnalisés.
* Intégrer le résultat à d’autres API ou services.
* Modifier les données audio, écrire des en-têtes `.wav` personnalisés, etc.

Il est facile d’apporter cette modification à partir de l’exemple précédent. Tout d’abord, supprimez `AudioConfig`, car vous allez gérer le comportement de sortie manuellement à partir de ce point pour un contrôle accru. Transmettez ensuite `None` pour `AudioConfig` dans le constructeur `SpeechSynthesizer`.

> [!NOTE]
> Le fait de transmettre `None` pour `AudioConfig`, au lieu de l’omettre comme dans l’exemple de sortie de haut-parleur précédent, n’aura pas pour effet de lire par défaut l’audio sur l’appareil de sortie actuellement actif.

Cette fois, vous allez enregistrer le résultat dans une variable [`SpeechSynthesisResult`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesisresult). La propriété `audio_data` contient un objet `bytes` des données de sortie. Vous pouvez utiliser cet objet manuellement ou utiliser la classe [`AudioDataStream`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audiodatastream) pour gérer le flux en mémoire. Dans cet exemple, vous allez utiliser le constructeur `AudioDataStream` pour obtenir un flux à partir du résultat.

```python
synthesizer = SpeechSynthesizer(speech_config=speech_config, audio_config=None)
result = synthesizer.speak_text_async("Getting the response as an in-memory stream.").get()
stream = AudioDataStream(result)
```

À partir de là, vous pouvez implémenter un comportement personnalisé à partir de l’objet `stream` obtenu.

## <a name="customize-audio-format"></a>Personnaliser le format audio

La section suivante montre comment personnaliser les attributs de sortie audio, notamment :

* Le type de fichier audio
* Le taux d’échantillonnage
* La profondeur de bits

Pour changer de format audio, utilisez la fonction `set_speech_synthesis_output_format()` sur l’objet `SpeechConfig`. Cette fonction attend un `enum` de type [`SpeechSynthesisOutputFormat`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesisoutputformat), dont vous allez vous servir pour sélectionner le format de sortie. Consultez les documents de référence pour obtenir la [liste des formats audio](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechsynthesisoutputformat) disponibles.

Il existe diverses options pour les différents types de fichiers, ce qui permet de répondre à vos besoins. Notez que par définition, les formats bruts comme `Raw24Khz16BitMonoPcm` n’incluent pas d’en-têtes audio. N’utilisez les formats bruts que si vous savez que votre implémentation en aval pourra décoder un flux binaire brut ou bien si vous envisagez de créer manuellement des en-têtes en fonction de la profondeur de bits, du taux d’échantillonnage, du nombre de canaux, etc.

Dans cet exemple, vous allez spécifier un format RIFF haute fidélité `Riff24Khz16BitMonoPcm` en définissant `SpeechSynthesisOutputFormat` sur l’objet `SpeechConfig`. Comme dans l’exemple de la section précédente, vous allez utiliser [`AudioDataStream`](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audiodatastream) pour obtenir un flux en mémoire du résultat, puis l’écrire dans un fichier.


```python
speech_config.set_speech_synthesis_output_format(SpeechSynthesisOutputFormat["Riff24Khz16BitMonoPcm"])
synthesizer = SpeechSynthesizer(speech_config=speech_config, audio_config=None)

result = synthesizer.speak_text_async("Customizing audio output format.").get()
stream = AudioDataStream(result)
stream.save_to_wav_file("path/to/write/file.wav")
```

Si vous réexécutez votre programme, un fichier `.wav` personnalisé est écrit dans le chemin spécifié.

## <a name="use-ssml-to-customize-speech-characteristics"></a>Utiliser SSML pour personnaliser les caractéristiques vocales

Le langage SSML (Speech Synthesis Markup Language) vous permet d’ajuster la tonalité, la prononciation, la vitesse d’élocution, le volume et d’autres éléments de sortie de la synthèse vocale en soumettant vos demandes à partir d’un schéma XML. Cette section présente un exemple de modification de la voix, mais pour obtenir pour des indications plus détaillées, consultez l’[article de guide pratique sur SSML](../../../speech-synthesis-markup.md).

Pour commencer à utiliser SSML pour la personnalisation, vous allez apporter une modification simple qui change la voix.
Tout d’abord, créez un fichier XML pour la configuration SSML dans le répertoire racine de votre projet, à savoir `ssml.xml`. L’élément racine est toujours `<speak>`, et le fait d’intégrer le texte dans un élément `<voice>` vous permet de changer la voix à l’aide du paramètre `name`. Consultez la [liste complète](../../../language-support.md#neural-voices) des voix **neuronales** prises en charge.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-US-ChristopherNeural">
    When you're on the freeway, it's a good idea to use a GPS.
  </voice>
</speak>
```

Ensuite, vous devez changer la demande de synthèse vocale de sorte qu’elle fasse référence à votre fichier XML. La demande est presque totalement identique, sauf qu’au lieu d’utiliser la fonction `speak_text_async()`, vous allez utiliser `speak_ssml_async()`. Sachant que cette fonction attend une chaîne XML, vous commencez par lire votre configuration SSML sous forme de chaîne. À partir de là, l’objet obtenu est exactement le même que dans les exemples précédents.

> [!NOTE]
> Si votre `ssml_string` contient `ï»¿` en début de chaîne, vous devez éliminer le format BOM ou le service retourne une erreur. Pour cela, vous devez définir le paramètre `encoding` comme suit : `open("ssml.xml", "r", encoding="utf-8-sig")`.

```python
synthesizer = SpeechSynthesizer(speech_config=speech_config, audio_config=None)

ssml_string = open("ssml.xml", "r").read()
result = synthesizer.speak_ssml_async(ssml_string).get()

stream = AudioDataStream(result)
stream.save_to_wav_file("path/to/write/file.wav")
```

> [!NOTE]
> Pour modifier la voix sans utiliser SSML, vous pouvez définir la propriété sur `SpeechConfig` à l’aide de `SpeechConfig.speech_synthesis_voice_name = "en-US-ChristopherNeural"`

## <a name="get-facial-pose-events"></a>Obtenir des événements de pose faciale

La reconnaissance vocale peut être un bon moyen de piloter l’animation des expressions faciales.
Souvent, les [visèmes](../../../how-to-speech-synthesis-viseme.md) sont utilisées pour représenter les principaux éléments de parole observés, tels que la position des lèvres, de la mâchoire et de la langue lors de la production d’un phonème particulier.
Vous pouvez vous abonner à l’événement de visème dans le kit de développement logiciel (SDK) Speech.
Ensuite, vous pouvez appliquer des événements de visème pour animer le visage d’un personnage au fur et à mesure de la lecture audio.
Découvrez [comment obtenir des événements de visème](../../../how-to-speech-synthesis-viseme.md#get-viseme-events-with-the-speech-sdk).
