---
title: Utiliser l'Analyse comportementale des entités pour détecter les menaces avancées | Microsoft Docs
description: Activer l'Analyse comportementale des utilisateurs et des entités dans Azure Sentinel, et configurer des sources de données
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/25/2021
ms.author: yelevin
ms.openlocfilehash: 985f1d4edacd81b7567124845836c3d93f976bb2
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122525232"
---
# <a name="enable-user-and-entity-behavior-analytics-ueba-in-azure-sentinel"></a>Activer l'Analyse comportementale des utilisateurs et des entités dans Azure Sentinel 

> [!IMPORTANT]
>
> Les fonctionnalités UEBA et Entity Pages sont désormais en **disponibilité générale** dans **_toutes_** les régions et zones géographiques Azure Sentinel. 

[!INCLUDE [reference-to-feature-availability](includes/reference-to-feature-availability.md)]

## <a name="prerequisites"></a>Prérequis

Pour activer ou désactiver cette fonctionnalité (ces conditions préalables ne sont pas requises pour utiliser la fonctionnalité) :

- Votre utilisateur doit être membre de l’annuaire Azure Active Directory de votre organisation et non un utilisateur invité.

- Les rôles **Administrateur général** ou **Administrateur de sécurité** doivent être attribués à votre utilisateur dans Azure AD.

- Vous devez attribuer au moins l’un des **rôles Azure** suivants à votre utilisateur ([en savoir plus sur Azure RBAC](roles.md)) :
    - **Contributeur Azure Sentinel** au niveau de l’espace de travail ou du groupe de ressources.
    - **Contributeur Log Analytics** au niveau du groupe de ressources ou de l’abonnement.

- Aucun verrou de ressource Azure ne doit être appliqué à votre espace de travail. [Apprenez-en davantage sur le verrouillage des ressources Azure](../azure-resource-manager/management/lock-resources.md).

> [!NOTE]
> Aucune licence spéciale n’est nécessaire pour ajouter la fonctionnalité UEBA à Azure Sentinel. Cependant, des **frais supplémentaires** peuvent s’appliquer.

## <a name="how-to-enable-user-and-entity-behavior-analytics"></a>Activer l'Analyse comportementale des utilisateurs et des entités

1. Dans le menu de navigation d'Azure Sentinel, sélectionnez **Comportement des entités**.

1. Sous l'en-tête **Activer**, placez le commutateur sur **Activé**.

1. Cliquez sur le bouton **Sélectionner des sources de données**.

1. Dans le volet **Sélection des sources de données**, cochez les cases situées en regard des sources de données sur lesquelles vous souhaitez activer l'Analyse comportementale des utilisateurs et des entités, puis sélectionnez **Appliquer**.

    > [!NOTE]
    >
    > La partie inférieure du volet **Sélection des sources de données** répertorie les sources de données prises en charge par l'Analyse comportementale des utilisateurs et des entités et que vous n'avez pas encore activées. 
    >
    > Après avoir activé l'Analyse comportementale des utilisateurs et des entités, vous aurez la possibilité, lors de la connexion de nouvelles sources de données, de les activer à partir du volet des connecteurs de données si elles sont compatibles avec l'Analyse comportementale des utilisateurs et des entités.

1. Sélectionnez **Accéder à la recherche d'entités**. Vous accédez alors au volet de recherche d'entités. Désormais, celui-ci s'affichera lorsque vous choisirez **Comportement des entités** dans le menu principal d'Azure Sentinel.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à activer et configurer l'Analyse comportementale des utilisateurs et des entités dans Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](detect-threats-built-in.md).
