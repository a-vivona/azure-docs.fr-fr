---
title: Utilisez Azure Security Center pour renforcer la sécurité de vos hôtes Docker et protéger les conteneurs
description: Protéger vos hôtes Docker et vérifier qu’ils sont conformes au test d’évaluation CIS Docker
author: memildin
ms.author: memildin
ms.date: 07/18/2021
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 2e421815fd962a62760c4d16106daa7f85fb1599
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122524453"
---
# <a name="harden-your-docker-hosts"></a>Durcir vos hôtes Docker

Azure Security Center identifie les conteneurs non managés qui sont hébergés sur des machines virtuelles IaaS Linux, ou d’autres machines Linux exécutant des conteneurs Docker. Security Center évalue en continu les configurations de ces conteneurs. Il les compare ensuite au document de référence [Center for Internet Security (CIS) Docker Benchmark](https://www.cisecurity.org/benchmark/docker/).

Security Center inclut la totalité des règles définies dans le CIS Docker Benchmark et vous envoie une alerte si vos conteneurs ne satisfont pas à tous les contrôles. Quand il détecte des configurations incorrectes, Security Center génère des recommandations de sécurité. Accédez à la **page des recommandations** de Security Center pour lire les suggestions et corriger les problèmes.

Lorsque des vulnérabilités sont détectées, elles sont regroupées dans une recommandation unique.

>[!NOTE]
> Ces vérifications des règles du CIS Benchmark ne sont pas effectuées sur les instances managées par AKS ni sur les machines virtuelles managées par Databricks.

## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale|
|Prix :|Nécessite [Azure Defender pour les serveurs](defender-for-servers-introduction.md)|
|Rôles et autorisations obligatoires :|**Lecteur** sur l’espace de travail auquel l’hôte se connecte|
|Clouds :|:::image type="icon" source="./media/icons/yes-icon.png"::: Clouds commerciaux<br>:::image type="icon" source="./media/icons/yes-icon.png"::: National/Souverain (Azure Government, Azure China 21Vianet)|
|||

## <a name="identify-and-remediate-security-vulnerabilities-in-your-docker-configuration"></a>Identifier et corriger les vulnérabilités de sécurité dans votre configuration Docker

1. Dans le menu de Security Center, ouvrez la page **Recommandations**.

1. Filtrez la recommandation **Les vulnérabilités dans les configurations de la sécurité des conteneurs doivent être corrigées** et sélectionnez la recommandation.

    La page de la recommandation affiche les ressources affectées (hôtes Docker). 

    :::image type="content" source="./media/monitor-container-security/docker-host-vulnerabilities-found.png" alt-text="Recommandations pour corriger les vulnérabilités dans les configurations de sécurité de conteneur.":::

    > [!NOTE]
    > Les ordinateurs qui n’exécutent pas Docker s’affichent sous l’onglet **Ressources non applicables**. Ils apparaissent dans Azure Policy comme étant conformes. 

1. Pour afficher et corriger les contrôles CIS qui ont échoué sur un hôte spécifique, sélectionnez l’hôte que vous souhaitez examiner. 

    > [!TIP]
    > Si vous avez atteint cette recommandation en partant de la page d’inventaire des ressources, sélectionnez le bouton **Entreprendre une action** sur la page de la recommandation.
    >
    > :::image type="content" source="./media/monitor-container-security/host-security-take-action-button.png" alt-text="Bouton Entreprendre une action pour lancer Log Analytics.":::

    Log Analytics s’ouvre sur une opération personnalisée prête à être exécutée. La requête personnalisée par défaut comprend une liste de toutes les règles ayant échoué qui ont été évaluées, ainsi que des instructions pour vous aider à résoudre les problèmes.

    :::image type="content" source="./media/monitor-container-security/docker-host-vulnerabilities-in-query.png" alt-text="Page Log Analytics avec la requête indiquant tous les contrôles CIS ayant échoué.":::

1. Ajustez les paramètres de requête si nécessaire.

1. Lorsque vous êtes sûr que la commande est appropriée et prête pour votre hôte, sélectionnez **Exécuter**.


## <a name="next-steps"></a>Étapes suivantes

Le renforcement Docker n’est qu’un aspect des fonctionnalités de sécurité des conteneurs de Security Center. 

En savoir plus sur la [sécurité des conteneurs dans Security Center](container-security.md).