---
title: Attribuer des rôles de ressource Azure dans Privileged Identity Management - Azure Active Directory | Microsoft Docs
description: Découvrez comment attribuer des rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 06/15/2021
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c50d62d5c8f24ed25258305411f9ed045098c7f
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/16/2021
ms.locfileid: "112232828"
---
# <a name="assign-azure-resource-roles-in-privileged-identity-management"></a>Attribuer des rôles de ressources Azure dans Privileged Identity Management

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) peut gérer les rôles de ressources intégrés d’Azure, ainsi que les rôles personnalisés, notamment (liste non exhaustive) :

- Propriétaire
- Administrateur de l'accès utilisateur
- Contributeur
- Administrateur de la sécurité
- Gestionnaire de sécurité

> [!NOTE]
> Les utilisateurs ou membres d’un groupe affecté aux rôles d’abonnement Propriétaire ou Administrateur d’accès utilisateur et les administrateurs d’Azure AD Global qui activent la gestion des abonnements dans Azure AD disposent des droits d’administrateur de ressources par défaut. Ces administrateurs peuvent assigner des rôles, configurer des paramètres de rôle et revoir les accès à l’aide de Privileged Identity Management pour Azure Resources. Un utilisateur ne peut pas gérer Privileged Identity Management pour les ressources sans autorisations d’administrateur de ressources. Affichez la liste des [rôles Azure intégrés](../../role-based-access-control/built-in-roles.md).

Privileged Identity Management prend en charge les rôles Azure personnalisés et intégrés. Pour plus d’informations sur les rôles personnalisés Azure, consultez [Rôles personnalisés Azure](../../role-based-access-control/custom-roles.md).

## <a name="role-assignment-conditions"></a>Conditions d’attribution de rôle

Vous pouvez utiliser la préversion du contrôle d’accès basé sur les attributs Azure (Azure ABAC) pour placer des conditions de ressources sur les attributions de rôles éligibles à l’aide de Privileged Identity Management (PIM). Avec PIM, vos utilisateurs finaux doivent activer une attribution de rôle éligible pour avoir l’autorisation d’effectuer certaines actions. L’utilisation de conditions Azure ABAC dans PIM vous permet non seulement de limiter les autorisations de rôle d’un utilisateur à une ressource à l’aide de conditions affinées, mais également d’utiliser PIM pour sécuriser l’attribution de rôle avec un paramètre de durée, un workflow d’approbation, une piste d’audit, etc. Pour plus d’informations, consultez version [Préversion du contrôle d’accès basé sur les attributs Azure](../../role-based-access-control/conditions-overview.md).

## <a name="assign-a-role"></a>Attribuer un rôle

Suivez ces étapes pour rendre un utilisateur éligible pour un rôle de ressource Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) avec les autorisations de rôle Propriétaire ou Administrateur de l’accès utilisateur.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Utilisez le filtre de ressources pour rechercher les ressources managées que vous recherchez.

    ![Liste des ressources Azure à gérer](./media/pim-resource-roles-assign-roles/resources-list.png)

1. Sélectionnez la ressource que vous souhaitez gérer pour ouvrir la page de présentation des ressources.

1. Sous **Gérer**, sélectionnez **Rôles** pour afficher la liste des rôles pour les ressources Azure.

    ![Rôles de ressources Azure](./media/pim-resource-roles-assign-roles/resources-roles.png)

1. Sélectionnez **Ajouter des affectations** pour ouvrir le volet **Ajouter des affectations**.

1. Choisissez **Sélectionner un rôle** pour ouvrir la page **Sélectionner un rôle**.

    ![Volet Nouvelle affectation](./media/pim-resource-roles-assign-roles/resources-select-role.png)

1. Sélectionnez un rôle que vous souhaitez attribuer et cliquez sur **Sélectionner**.

    Le volet **Sélectionner un membre ou un groupe** s’ouvre.

1. Sélectionnez un membre ou un groupe que vous souhaitez affecter au rôle, puis cliquez sur **Sélectionner**.

    ![Volet Sélectionner un membre ou un groupe](./media/pim-resource-roles-assign-roles/resources-select-member-or-group.png)

1. Dans l’onglet **Paramètres**, liste **Type d’affectation**, sélectionnez **Éligible** ou **Actif**.

    ![Volet des paramètres des appartenances](./media/pim-resource-roles-assign-roles/resources-membership-settings-type.png)

    Azure AD Privileged Identity Management pour les ressources Azure fournit deux types distincts d’attribution :

    - Les attributions **éligibles** exigent des membres qu’ils effectuent une action pour utiliser ce rôle. Il peut s’agir de procéder à une vérification de l’authentification multifacteur (MFA), de fournir une justification professionnelle ou de demander une approbation aux approbateurs désignés.

    - Les attributions **actives** n’exigent pas des membres qu’ils effectuent une action pour utiliser ce rôle. Les membres attribués comme étant actifs détiennent à tout moment les privilèges affectés au rôle.

1. Pour spécifier une durée d’attribution spécifique, modifiez les dates et heures de début et de fin.

1. Lorsque vous avez terminé, sélectionnez **Attribuer**.

1. Une fois la nouvelle attribution de rôle créée, une notification d’état s’affiche.

    ![Nouvelle affectation - Notification](./media/pim-resource-roles-assign-roles/resources-new-assignment-notification.png)

## <a name="update-or-remove-an-existing-role-assignment"></a>Mettre à jour ou supprimer une attribution de rôle existante

Suivez ces étapes pour mettre à jour ou supprimer une attribution de rôle existante.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Sélectionnez la ressource que vous souhaitez gérer pour ouvrir sa page de présentation.

1. Sous **Gérer**, sélectionnez **Rôles** pour afficher la liste des rôles pour les ressources Azure.

    ![Rôles de ressources Azure - Sélectionner un rôle](./media/pim-resource-roles-assign-roles/resources-update-select-role.png)

1. Sélectionnez le rôle que vous souhaitez mettre à jour ou supprimer.

1. Recherchez l’attribution de rôle sous les onglets **Rôles éligibles** et **Rôles actifs**.

    ![Mettre à jour ou supprimer une attribution de rôle](./media/pim-resource-roles-assign-roles/resources-update-remove.png)

1. Pour ajouter ou mettre à jour une condition pour affiner l’accès aux ressources Azure, sélectionnez **Ajouter** ou **Afficher/Modifier** dans la colonne **Condition** de l’attribution de rôle. Actuellement, les rôles Propriétaire des données blob de stockage, Lecteur de données blob de stockage et Contributeur de données blob de stockage dans Privileged Identity Management sont les seuls rôles pris en charge dans le cadre de la [préversion du contrôle d’accès basé sur les attributs Azure](../../role-based-access-control/conditions-overview.md).

    ![Mettre à jour ou supprimer des attributs pour le contrôle d’accès](./media/pim-resource-roles-assign-roles/resources-abac-update-remove.png)

1. Sélectionnez **Mettre à jour** ou **Supprimer** pour mettre à jour ou supprimer l’attribution de rôle.

    Pour obtenir des informations sur l’extension d’une attribution de rôle, consultez [Étendre ou renouveler des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-renew-extend.md).

## <a name="next-steps"></a>Étapes suivantes

- [Étendre ou renouveler des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-renew-extend.md)
- [Configurer les paramètres des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-configure-role-settings.md)
- [Attribuer des rôles dans Azure AD dans Privileged Identity Management](pim-how-to-add-role-to-user.md)
