---
title: Gérer les incidents de sécurité dans Azure Security Center | Microsoft Docs
description: Ce document est conçu pour vous aider à utiliser d’Azure Security Center pour gérer les incidents de sécurité.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 02/17/2021
ms.author: memildin
ms.openlocfilehash: e385dec04db44e88ef5759cf8e2391b19b77dccc
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/16/2021
ms.locfileid: "112237530"
---
# <a name="manage-security-incidents-in-azure-security-center"></a>Gérer les incidents de sécurité dans Azure Security Center

Procéder au triage et à l’examen des alertes de sécurité peut prendre beaucoup de temps, même pour le plus compétent des analystes de sécurité. Pour de nombreuses personnes, il est difficile de savoir où commencer. 

Security Center utilise l’[analyse](./security-center-alerts-overview.md) pour connecter les informations entre des [alertes de sécurité](security-center-managing-and-responding-alerts.md) distinctes. À l’aide de ces connexions, Security Center peut fournir une vue unique d’une campagne d’attaque et de ses alertes associées pour vous aider à comprendre les actions de l’attaquant et les ressources affectées.

Cette page offre une vue d’ensemble des incidents dans Security Center.

## <a name="what-is-a-security-incident"></a>Qu’est-ce qu’un incident de sécurité ?

Dans le Centre de sécurité, un incident de sécurité est un regroupement de toutes les alertes d’une ressource correspondant à des modèles de [chaîne de destruction](alerts-reference.md#intentions) . Les incidents apparaissent dans la page [Alertes de sécurité](security-center-managing-and-responding-alerts.md). Sélectionnez un incident pour afficher les alertes associées et obtenir plus d’informations.

## <a name="managing-security-incidents"></a>Gestion des incidents de sécurité

1. Sur la page des alertes de Security Center, utilisez le bouton **Ajouter un filtre** pour filtrer, par nom d’alerte, le nom de l’alerte **Incident de sécurité détecté sur plusieurs ressources**. 

    :::image type="content" source="media/security-center-incident/locating-incidents.png" alt-text="Recherche des incidents sur la page des alertes dans Azure Security Center.":::

    La liste est désormais filtrée pour afficher uniquement les incidents. Notez que l’icône des incidents de sécurité est différente de celle des alertes de sécurité.

    :::image type="content" source="media/security-center-incident/incidents-list.png" alt-text="Liste des incidents sur la page des alertes dans Azure Security Center.":::

1. Pour consulter les détails d’un incident, sélectionnez-le dans la liste. Un volet latéral s’affiche avec plus de détails sur l’incident.

    :::image type="content" source="media/security-center-incident/incident-quick-peek.png" alt-text="Volet latéral présentant les détails de l’incident.":::

1. Pour voir plus de détails, sélectionnez **Afficher les détails complets**.

    [![Répondre aux incidents de sécurité dans Azure Security Center.](media/security-center-incident/incident-details.png)](media/security-center-incident/incident-details.png#lightbox)

    Le volet gauche de la page de l’incident de sécurité affiche des informations générales sur l’incident de sécurité : titre, gravité, état, durée d’activité, description et ressource affectée. En regard de la ressource affectée, vous pouvez voir les balises Azure correspondantes. Utilisez-les pour déduire le contexte organisationnel de la ressource lors de l’examen de l’alerte.

    Le volet droit comprend l’onglet **Alertes** avec les alertes de sécurité corrélées dans le cadre de cet incident. 

    >[!TIP]
    > Pour plus d’informations sur une alerte spécifique, sélectionnez-la. 

    [![Onglet Entreprendre une action pour l’incident.](media/security-center-incident/incident-take-action-tab.png)](media/security-center-incident/incident-take-action-tab.png#lightbox)

    Pour basculer vers l’onglet **Entreprendre une action**, sélectionnez l’onglet ou le bouton en bas du volet droit. Utilisez cet onglet pour entreprendre d’autres actions telles que :
    - *Atténuer la menace* : fournit des étapes de correction manuelle pour cet incident de sécurité
    - *Empêcher les attaques futures* : fournit des recommandations de sécurité pour aider à réduire la surface d’attaque, améliorer la posture de sécurité et empêcher les attaques futures
    - *Déclencher une réponse automatisée* : permet de déclencher une application logique en guide de réponse à cet incident de sécurité
    - *Supprimer les alertes similaires* : permet de supprimer les alertes futures ayant des caractéristiques similaires si l’alerte n’est pas pertinente pour votre organisation 

   > [!NOTE]
   > Une même alerte peut s’afficher dans le cadre d’un incident, mais également apparaître sous la forme d’une alerte autonome.

1. Pour atténuer les menaces dans l’incident, suivez les étapes de correction fournies avec chaque alerte.


## <a name="next-steps"></a>Étapes suivantes

Cette page a expliqué les fonctionnalités d’incident de sécurité de Security Center. Pour accéder à des informations connexes, consultez les rubriques suivantes :

- [Alertes de sécurité dans Security Center](security-center-alerts-overview.md)
- [Gérer et répondre aux alertes de sécurité](security-center-managing-and-responding-alerts.md)