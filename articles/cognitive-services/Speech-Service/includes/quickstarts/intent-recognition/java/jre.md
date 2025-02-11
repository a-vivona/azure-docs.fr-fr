---
author: PatrickFarley
ms.service: cognitive-services
ms.subservice: speech-service
ms.date: 04/04/2020
ms.topic: include
ms.author: pafarley
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: 316140e49c5ec5674b54ddcd597e428a8191f1c1
ms.sourcegitcommit: f2d0e1e91a6c345858d3c21b387b15e3b1fa8b4c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/07/2021
ms.locfileid: "123543295"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

* <a href="~/articles/cognitive-services/Speech-Service/quickstarts/setup-platform.md?tabs=jre&pivots=programming-language-java" target="_blank">Installez le SDK Speech pour votre environnement de développement, puis créez un exemple de projet vide</a>.

## <a name="create-a-luis-app-for-intent-recognition"></a>Créer une application LUIS pour la reconnaissance de l’intention

[!INCLUDE [Create a LUIS app for intent recognition](../luis-sign-up.md)]

## <a name="open-your-project"></a>Ouvrir votre projet

1. Ouvrez l’IDE de votre choix.
2. Chargez votre projet et ouvrez `Main.java`.

## <a name="start-with-some-boilerplate-code"></a>Commencer avec du code réutilisable

Nous allons ajouter du code qui servira de squelette à notre projet

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=6-20,68-75)]

## <a name="create-a-speech-configuration"></a>Créer une configuration Speech

Pour pouvoir initialiser un objet `IntentRecognizer`, vous devez au préalable créer une configuration qui utilise la clé et la localisation de votre ressource de prédiction LUIS.  

Insérez ce code dans le bloc try/catch dans `main()`. Veillez à mettre à jour ces valeurs :

* Remplacez `"YourLanguageUnderstandingSubscriptionKey"` par votre clé de prédiction LUIS.
* Remplacez `"YourLanguageUnderstandingServiceRegion"` par votre localisation LUIS. Utilisez l’**identificateur Région** de la [région](../../../../regions.md).

>[!TIP]
> Si vous avez besoin d’aide pour trouver ces valeurs, consultez [Créer une application LUIS pour la reconnaissance de l’intention](#create-a-luis-app-for-intent-recognition).

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=27)]

Cet exemple utilise la méthode `FromSubscription()` pour générer la `SpeechConfig`. Pour obtenir la liste complète des méthodes disponibles, consultez la rubrique [Classe SpeechConfig](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig).

Le SDK Speech reconnaît par défaut l’utilisation de la langue en-US. Consultez [Spécifier la langue source pour la reconnaissance vocale](../../../../how-to-specify-source-language.md) pour plus d’informations sur le choix de la langue source.

## <a name="initialize-an-intentrecognizer"></a>Initialiser IntentRecognizer

Créons maintenant un `IntentRecognizer`. Insérez ce code juste en dessous de votre configuration Speech.

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=30)]

## <a name="add-a-languageunderstandingmodel-and-intents"></a>Ajouter un LanguageUnderstandingModel et des intentions

Vous devez associer un `LanguageUnderstandingModel` au module de reconnaissance de l’intention et ajouter les intentions que vous souhaitez reconnaître. Nous allons utiliser les intentions du domaine prédéfini pour la domotique.

Insérez ce code en dessous de votre `IntentRecognizer`. Veillez à remplacer `"YourLanguageUnderstandingAppId"` par votre ID d’application LUIS.

>[!TIP]
> Si vous avez besoin d’aide pour trouver cette valeur, consultez [Créer une application LUIS pour la reconnaissance de l’intention](#create-a-luis-app-for-intent-recognition).

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=33-35)]

Cet exemple utilise la fonction `addIntent()` pour ajouter des intentions individuellement. Si vous souhaitez ajouter toutes les intentions d’un modèle, utilisez `addAllIntents(model)` et transmettez le modèle.

> [!NOTE]
> Le kit SDK Speech prend en charge les points de terminaison LUIS v2.0 uniquement.
> Vous devez modifier manuellement l’URL de point de terminaison v3.0 trouvée dans l’exemple de champ de requête pour utiliser un modèle d’URL v2.0.
> Les points de terminaison LUIS v2.0 suivent toujours l’un de ces deux modèles :
> * `https://{AzureResourceName}.cognitiveservices.azure.com/luis/v2.0/apps/{app-id}?subscription-key={subkey}&verbose=true&q=`
> * `https://{Region}.api.cognitive.microsoft.com/luis/v2.0/apps/{app-id}?subscription-key={subkey}&verbose=true&q=`

## <a name="recognize-an-intent"></a>Reconnaître une intention

À partir de l’objet `IntentRecognizer`, vous devez appeler la méthode `recognizeOnceAsync()`. Cette méthode permet au service Speech de savoir que vous envoyez une seule expression pour reconnaissance, et d’arrêter la reconnaissance une fois que l’expression a été identifiée.

Insérez ce code en dessous de votre modèle :

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=40)]

## <a name="display-the-recognition-results-or-errors"></a>Afficher les résultats de la reconnaissance (ou les erreurs)

Lorsque le résultat de la reconnaissance est retourné par le service de reconnaissance vocale, vous pouvez effectuer une opération avec celui-ci. Nous allons faire simple et imprimer le résultat dans la console.

Insérez ce code sous votre appel à `recognizeOnceAsync()`.

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=43-64)]

## <a name="release-resources"></a>Libérer des ressources

Il est important de libérer les ressources lorsque vous avez fini de les utiliser. Insérez ce code à la fin du bloc try/catch :

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=66-67)]

## <a name="check-your-code"></a>Vérifier votre code

À ce stade, votre code doit ressembler à ceci :

> [!NOTE]
> Nous avons ajouté des commentaires à cette version.

[!code-java[](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/intent-recognition/src/speechsdk/quickstart/Main.java?range=6-75)]

## <a name="build-and-run-your-app"></a>Générer et exécuter votre application

Appuyez sur <kbd>F11</kbd> ou sélectionnez **Run** (Exécuter) > **Debug** (Déboguer).
Les 15 secondes suivantes de saisie vocale provenant de votre microphone seront reconnues et enregistrées dans la fenêtre console.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]
