---
title: Incorporer un widget de lecteur dans Power BI - Azure Video Analyzer
description: Vous pouvez utiliser Azure Video Analyzer pour un enregistrement vidéo en continu ou un enregistrement basé sur les événements. Cet article explique comment incorporer des vidéos dans Microsoft Power BI pour fournir à vos utilisateurs une interface utilisateur personnalisable.
ms.service: azure-video-analyzer
ms.topic: how-to
ms.date: 08/06/2021
ms.openlocfilehash: cbb40db29f9f2257d058268e6c639d2a6b89822e
ms.sourcegitcommit: 2d412ea97cad0a2f66c434794429ea80da9d65aa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/14/2021
ms.locfileid: "122563610"
---
# <a name="embed-player-widget-in-power-bi"></a>Incorporer un widget de lecteur dans Power BI

Azure Video Analyzer vous permet d’[enregistrer](detect-motion-record-video-clips-cloud.md) de la vidéo et les métadonnées d’inférence associées dans votre ressource cloud Video Analyzer. Video Analyzer possède un [widget de lecteur](player-widget.md), c’est-à-dire, un widget facile à incorporer qui permet aux applications clientes de lire la vidéo et les métadonnées d’inférence.

Les tableaux de bord sont un moyen utile pour superviser votre activité et voir vos métriques les plus importantes en un clin d’œil. Un tableau de bord Power BI est un outil puissant pour combiner la vidéo avec plusieurs sources de données, dont la télémétrie à partir d’IoT Hub. Dans ce tutoriel, vous allez apprendre à ajouter un ou plusieurs widgets de lecteur à un tableau de bord à l’aide du service web [Microsoft Power BI](https://powerbi.microsoft.com/).

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée

- [Widget de lecteur](player-widget.md) Azure Video Analyzer
- Introduction aux [tableaux de bord Power BI](/power-bi/create-reports/service-dashboards)

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. Si vous n’en avez pas déjà un, [créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Complétez [Détecter les mouvements et enregistrer la vidéo](detect-motion-record-video-clips-cloud.md) ou [Enregistrement vidéo continu](continuous-video-recording.md) : un pipeline avec un récepteur vidéo est requis.

  > [!NOTE] 
  > Votre compte Video Analyzer doit avoir au moins une vidéo enregistrée pour continuer. Consultez la liste des vidéos en vous connectant à votre compte Azure Video Analyzer > Vidéos > section Video Analyzer.

- Un compte [Power BI](https://powerbi.microsoft.com/).

## <a name="create-a-token"></a>Créer un jeton

1. Suivez les étapes pour [créer un jeton](player-widget.md#create-a-token).
2. Veillez à enregistrer les valeurs générées pour l’_émetteur, l’audience, le type de clé, l’algorithme, l’ID de clé, le module de clé RSA, l’exposant de clé RSA, le jeton_. Vous aurez besoin de ces valeurs lors de la création d’une stratégie d’accès ci-dessous.

## <a name="get-embed-code-for-player-widget"></a>Obtenir du code incorporé pour le widget de lecteur

1. Connectez-vous au [portail Azure](https://portal.azure.com/) avec vos informations d’identification. Localisez votre compte Video Analyzer utilisé pour remplir les prérequis et ouvrez le volet du compte Video Analyzer.
2. Suivez les étapes pour [créer une stratégie d’accès](player-widget.md#create-an-access-policy).
3. Sélectionnez **Vidéos** dans la section **Video Analyzer**.
4. Sélectionnez une vidéo dans la liste.
5. Cliquez sur la configuration **Widget**. Un volet **Utiliser le widget dans votre application** s’ouvre sur le côté droit. Faites défiler jusqu’à **Option 2 – utilisation de HTML** et copiez-collez le code dans un éditeur de texte. Cliquez sur le bouton **Fermer**.

   :::image type="content" source="./media/power-bi/widget-code.png" alt-text="Capture d’écran de la copie du code HTML du widget à partir du portail AVA et enregistrement pour une utilisation ultérieure.":::

6. Modifier le code HTML copié à l’étape 5 pour lequel remplacer les valeurs
   - Jeton **AVA-API-JWT-TOKEN** : remplacez par la valeur de jeton que vous avez enregistrée dans l’étape « Créer un jeton ». Veillez à supprimer les crochets angulaires.
   - Facultatif : vous pouvez apporter d’autres modifications à l’interface utilisateur dans ce code, par exemple, en remplaçant l’en-tête « Exemple de widget de lecteur » par « Enregistrement vidéo continu ».

## <a name="add-widget-in-power-bi-dashboard"></a>Ajouter un widget dans le tableau de bord Power BI

1. Ouvrez le [service Power BI](http://app.powerbi.com/) dans votre navigateur. Dans le volet de navigation, sélectionnez **Mon espace de travail**.

   :::image type="content" source="./media/power-bi/power-bi-workspace.png" alt-text="Capture d’écran de la page d’accueil de l’espace de travail Power BI.":::

2. Créez un nouveau tableau de bord en cliquant sur **Nouveau** > **Tableau de bord** ou ouvrez un tableau de bord existant. Sélectionnez la flèche déroulante **Modifier** et **Ajouter une vignette**. Sélectionnez **Contenu web** > **Suivant**.
3. Dans la **vignette Ajouter du contenu web**, entrez votre **code incorporé** de la section précédente. Cliquez sur **Appliquer**.

   :::image type="content" source="./media/power-bi/embed-code.png" alt-text="Capture d’écran de l’incorporation du code html dans une vignette de tableau de bord Power BI.":::

4. Vous verrez un widget de lecteur épinglé au tableau de bord avec une vidéo.

   :::image type="content" source="./media/power-bi/one-player-added.png" alt-text="Capture d’écran d’un widget de lecteur vidéo ajouté.":::

5. Pour ajouter d’autres vidéos à partir de la section Vidéos Azure Video Analyzer, suivez les mêmes étapes que celles décrites dans cette section.

> [!NOTE] 
> Pour ajouter plusieurs vidéos à partir du même compte Video Analyzer, un seul ensemble de stratégie d’accès et un jeton suffisent.

Voici un exemple de plusieurs vidéos épinglées à un seul tableau de bord Power BI.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/power-bi/two-players-added.png" alt-text="Capture d’écran de deux widgets de lecteur vidéo ajoutés à titre d’exemple.":::

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’[API widget](https://github.com/Azure/video-analyzer/tree/main/widgets)