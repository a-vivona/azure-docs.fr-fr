---
title: Comment examiner les attributions de rôles RBAC Synapse dans Synapse Studio
description: Cet article explique comment examiner les attributions de rôles RBAC Synapse à l’aide de Synapse Studio
author: meenalsri
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 12/1/2020
ms.author: mesrivas
ms.reviewer: jrasnick
ms.openlocfilehash: 11e5ed746215df4518e9bd6af2c0c92539067e47
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122525095"
---
# <a name="how-to-review-synapse-rbac-role-assignments"></a>Guide pratique pour passer en revue les attributions de rôles RBAC Synapse

Des rôles RBAC sont utilisés pour attribuer des autorisations à des utilisateurs, des groupes et autres principaux de sécurité afin d’autoriser l’accès et l’utilisation des ressources Synapse.  [En savoir plus](./synapse-workspace-synapse-rbac.md)

Cet article explique comment passer en revue les attributions de rôles actuelles pour un espace de travail.

Avec un rôle RBAC Synapse, vous pouvez répertorier les attributions de rôles RBAC Synapse pour toutes les étendues, y compris les attributions pour des objets auxquels vous n’avez pas accès. Seul un administrateur Synapse peut accorder un accès RBAC Synapse.  

>[!Note]
> Les utilisateurs invités (utilisateurs d'un autre locataire AD) peuvent également voir et gérer l'attribution des rôles après avoir obtenu le rôle d'Administrateur Synapse.    

## <a name="open-synapse-studio"></a>Ouvrir Synapse Studio  

Pour passer en revue les attributions de rôles, commencez par [ouvrir le Synapse Studio](https://web.azuresynapse.net/), puis sélectionnez votre espace de travail. Pour vous connecter à votre espace de travail, il existe deux **méthodes de sélection de compte**. L’un est de l'**abonnement Azure**, l’autre est l'**entrée manuelle**. Si vous avez le rôle de Synapse Azure ou les rôles Azure de niveau supérieur, vous pouvez utiliser les deux méthodes pour vous connecter à l’espace de travail. Si vous n’avez pas les rôles Azure associés et que vous avez été accordé en tant que rôle de contrôle de la connexion au sein de l’espace de travail, **Entrer manuellement** est le seul moyen de vous connecter à l’espace de travail.

![Se connecter à l’espace de travail](./media/common/login-workspace.png) 
 
 Une fois que vous avez ouvert votre espace de travail, sélectionnez le hub **Gérer** sur la gauche, développez la section **Sécurité**, puis sélectionnez **Contrôle d’accès**. 

 ![Sélectionner Contrôle d’accès dans la section Sécurité à gauche](./media/how-to-manage-synapse-rbac-role-assignments/left-nav-security-access-control.png)

## <a name="review-workspace-role-assignments"></a>Examiner les attributions de rôle d’espace de travail

L’écran Contrôle d’accès répertorie toutes les attributions de rôles actuelles pour l’espace de travail, regroupées par rôle. Chaque affectation comprend le nom principal, le type de principal, le rôle et son étendue.

![Écran Contrôle d’accès](./media/how-to-review-synapse-rbac-role-assignments/access-control-assignments.png)

Si un principal se voit attribuer le même rôle dans différentes étendues, vous verrez plusieurs attributions pour le principal, à raison d’une par étendue.  

Si un rôle est attribué à un groupe de sécurité, vous verrez les rôles explicitement attribués au groupe, mais pas les rôles hérités de groupes parents.  

Vous pouvez filtrer la liste par nom de principal ou par e-mail, et filtrer de manière sélective les types d’objets, les rôles et les étendues. Entrez votre nom ou votre alias d’e-mail dans le filtre Nom pour voir les rôles qui vous sont attribués. Seul un Administrateur Synapse peut modifier vos rôles.

>[!Important] 
>Si vous êtes directement ou indirectement membre d’un groupe auquel des rôles sont attribués, vous pouvez avoir des autorisations qui ne sont pas affichées.

>[!tip]
>Vous pouvez rechercher des appartenances à des groupes à l’aide d’Azure Active Directory dans le portail Azure.  

Si vous créez un espace de travail, vous et le principal du service MSI de l’espace de travail recevez automatiquement le rôle Administrateur Synapse au niveau de l’étendue de l’espace de travail.

## <a name="next-steps"></a>Étapes suivantes

Découvrez [comment gérer les attributions de rôles RBAC Synapse](./how-to-manage-synapse-rbac-role-assignments.md).

Découvrez le [rôle dont vous avez besoin pour effectuer des tâches spécifiques](./synapse-workspace-understand-what-role-you-need.md).