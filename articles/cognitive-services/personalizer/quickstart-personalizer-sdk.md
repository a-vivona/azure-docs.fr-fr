---
title: 'Démarrage rapide : Créer et utiliser une boucle d’apprentissage avec le SDK - Personalizer'
description: Ce guide de démarrage rapide vous montre comment créer et gérer votre base de connaissances à l’aide de la bibliothèque de client Personalizer.
author: jeffmend
ms.author: jeffme
ms.manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: quickstart
ms.date: 08/27/2020
ms.custom: cog-serv-seo-aug-2020
keywords: personalizer, Azure personalizer, machine learning
zone_pivot_groups: programming-languages-set-six
ms.openlocfilehash: f35b20c47c12130423badcac3373be27c01e13c0
ms.sourcegitcommit: 16e25fb3a5fa8fc054e16f30dc925a7276f2a4cb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2021
ms.locfileid: "122829858"
---
# <a name="quickstart-personalizer-client-library"></a>Démarrage rapide : Bibliothèque de client Personalizer

Suivez ce guide de démarrage rapide pour afficher du contenu personnalisé avec le service Personalizer.

Commencez à utiliser la bibliothèque de client Personalizer. Suivez les étapes suivantes pour installer le package et essayer l’exemple de code pour les tâches de base.

 * API Rank : sélectionne l’élément le mieux adapté parmi les actions en fonction des informations en temps réel que vous fournissez sur le contenu et le contexte.
 * API Reward : vous déterminez le score de récompense en fonction des besoins de votre entreprise, puis vous l’envoyez à Personalizer avec cette API. Ce score peut être une valeur unique comme 1 pour Bon et 0 pour Mauvais, ou un algorithme que vous créez en fonction des besoins de votre entreprise.

::: zone pivot="programming-language-csharp"
[!INCLUDE [Get intent with C# SDK](./includes/quickstart-sdk-csharp.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [Get intent with Node.js SDK](./includes/quickstart-sdk-nodejs.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Get intent with Python SDK](./includes/quickstart-sdk-python.md)]
::: zone-end

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

* [Portail](../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Fonctionnement de Personalizer](how-personalizer-works.md)
> [Où utiliser Personalizer ?](where-can-you-use-personalizer.md)
