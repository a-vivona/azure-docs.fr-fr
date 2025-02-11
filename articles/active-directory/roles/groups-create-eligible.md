---
title: Créer un groupe pour l’attribution de rôles dans Azure Active Directory | Microsoft Docs
description: Découvrez comment créer un groupe avec attribution de rôle dans Azure AD Gérez les rôles Azure dans le Portail Azure, PowerShell ou l’API Graph.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 07/30/2021
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: c14580790891190f40dd2866aaac25ad2c286137
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122532326"
---
# <a name="create-a-role-assignable-group-in-azure-active-directory"></a>Créer un groupe avec attribution de rôle dans Azure Active Directory

Il n’est possible d’attribuer un rôle qu’à un groupe créé avec la propriété « isAssignableToRole » définie sur True ou créé sur le Portail Azure avec l’option **Des rôles Azure AD peuvent être attribués au groupe** activée. Avec cet attribut, le groupe peut être attribué à un rôle dans Azure AD (Azure Active Directory). Cet article explique comment créer ce type de groupe spécial. **Remarque :** Un groupe dont la propriété isAssignableToRole a la valeur true ne peut pas être de type d’appartenance dynamique. Pour plus d’informations, consultez [Utiliser des groupes cloud pour gérer les attributions de rôles dans Azure AD](groups-concept.md).

## <a name="prerequisites"></a>Prérequis

- Licence Azure AD Premium P1 ou P2
- Administrateur de rôle privilégié ou Administrateur général
- Module AzureAD (avec PowerShell)
- Consentement administrateur (avec l'Afficheur Graph pour l'API Microsoft Graph)

Pour plus d’informations, consultez [Prérequis pour utiliser PowerShell ou de l’Afficheur Graph](prerequisites.md).

## <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [Portail Azure](https://portal.azure.com) ou au [Centre d’administration Azure AD](https://aad.portal.azure.com).

1. Sélectionnez **Azure Active Directory** > **Groupes** > **Tous les groupes** > **Nouveau groupe**.

    [![Ouvrez Azure Active Directory et créez un groupe.](./media/groups-create-eligible/new-group.png "Ouvrez Azure Active Directory et créez un groupe.")](./media/groups-create-eligible/new-group.png#<lightbox>)

1. Sous l’onglet **Nouveau groupe**, spécifiez un type, un nom et une description pour le groupe.

1. Activez l’option **Des rôles Azure AD peuvent être attribués au groupe**. Ce commutateur est visible uniquement pour les utilisateurs autorisés à le définir, c’est-à-dire ceux qui ont le rôle Administrateur de rôle privilégié ou Administrateur général.

    [![Rendre le nouveau groupe éligible à l’attribution de rôle](./media/groups-create-eligible/eligible-switch.png "Rendre le nouveau groupe éligible à l’attribution de rôle")](./media/groups-create-eligible/eligible-switch.png#<lightbox>)

1. Sélectionnez les membres et les propriétaires du groupe. Vous avez également la possibilité d’attribuer des rôles au groupe. Dans le cas présent, ce n’est pas nécessaire.

    [![Ajoutez des membres au groupe avec attribution de rôle et attribuez des rôles.](./media/groups-create-eligible/specify-members.png "Ajoutez des membres au groupe avec attribution de rôle et attribuez des rôles.")](./media/groups-create-eligible/specify-members.png#<lightbox>)

1. Quand vous avez spécifié les membres et les propriétaires, sélectionnez **Créer**.

    [![Le bouton Créer est disponible en bas de la page.](./media/groups-create-eligible/create-button.png "Le bouton Créer est disponible en bas de la page.")](./media/groups-create-eligible/create-button.png#<lightbox>)

Le groupe est créé avec tous les rôles que vous lui avez attribués.

## <a name="powershell"></a>PowerShell

### <a name="create-a-group-that-can-be-assigned-to-role"></a>Créer un groupe auquel un rôle peut être attribué

```powershell
$group = New-AzureADMSGroup -DisplayName "Contoso_Helpdesk_Administrators" -Description "This group is assigned to Helpdesk Administrator built-in role in Azure AD." -MailEnabled $true -SecurityEnabled $true -MailNickName "contosohelpdeskadministrators" -IsAssignableToRole $true
```

Pour ce type de groupe, `isPublic` est toujours false et `isSecurityEnabled` est toujours true.

### <a name="copy-one-groups-users-and-service-principals-into-a-role-assignable-group"></a>Copier les utilisateurs et les principaux de service d’un groupe dans un groupe avec attribution de rôle

```powershell
#Basic set up
Install-Module -Name AzureAD
Import-Module -Name AzureAD
Get-Module -Name AzureAD

#Connect to Azure AD. Sign in as Privileged Role Administrator or Global Administrator. Only these two roles can create a role-assignable group.
Connect-AzureAD

#Input variabled: Existing group
$idOfExistingGroup = "14044411-d170-4cb0-99db-263ca3740a0c"

#Input variables: New role-assignable group
$groupName = "Contoso_Bellevue_Admins"
$groupDescription = "This group is assigned to Helpdesk Administrator built-in role in Azure AD."
$mailNickname = "contosobellevueadmins"

#Create new security group which is a role assignable group. For creating a Microsoft 365 group, set GroupTypes="Unified" and MailEnabled=$true
$roleAssignablegroup = New-AzureADMSGroup -DisplayName $groupName -Description $groupDescription -MailEnabled $false -MailNickname $mailNickname -SecurityEnabled $true -IsAssignableToRole $true

#Get details of existing group
$existingGroup = Get-AzureADMSGroup -Id $idOfExistingGroup
$membersOfExistingGroup = Get-AzureADGroupMember -ObjectId $existingGroup.Id

#Copy users and service principals from existing group to new group
foreach($member in $membersOfExistingGroup){
if($member.ObjectType -eq 'User' -or $member.ObjectType -eq 'ServicePrincipal'){
Add-AzureADGroupMember -ObjectId $roleAssignablegroup.Id -RefObjectId $member.ObjectId
}
}
```

## <a name="microsoft-graph-api"></a>API Microsoft Graph

### <a name="create-a-role-assignable-group-in-azure-ad"></a>Créer un groupe avec attribution de rôle dans Azure AD

```http
POST https://graph.microsoft.com/beta/groups
{
  "description": "This group is assigned to Helpdesk Administrator built-in role of Azure AD.",
  "displayName": "Contoso_Helpdesk_Administrators",
  "groupTypes": [
    "Unified"
  ],
  "isAssignableToRole": true,
  "mailEnabled": true,
  "securityEnabled": true,
  "mailNickname": "contosohelpdeskadministrators",
  "visibility" : "Private"
}
```

Pour ce type de groupe, `isPublic` est toujours false et `isSecurityEnabled` est toujours true.

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles Azure AD aux groupes](groups-assign-role.md)
- [Utiliser des groupes Azure AD pour gérer les attributions de rôles](groups-concept.md)
- [Résoudre les problèmes de rôles Azure AD attribués aux groupes](groups-faq-troubleshooting.yml)
