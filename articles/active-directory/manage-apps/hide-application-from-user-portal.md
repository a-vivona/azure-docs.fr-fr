---
title: Masquer une application d’entreprise de l’expérience utilisateur dans Azure AD
description: Guide pratique pour masquer une application d’entreprise de l’expérience utilisateur dans les panneaux d’accès Azure Active Directory ou les lanceurs Microsoft 365.
services: active-directory
author: davidmu1
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 03/25/2020
ms.author: davidmu
ms.reviewer: lenalepa
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5fb04026513953ff1e75bb481a7c3a5ae0bb7c9e
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122563192"
---
# <a name="hide-enterprise-applications-from-end-users-in-azure-active-directory"></a>Masquer des applications d’entreprise aux utilisateurs finaux dans Azure Active Directory

Instructions pour masquer les applications du panneau MyApps ou du Lanceur Microsoft 365 à l’utilisateur final. Lorsqu’une application est masquée, les utilisateurs ont toujours les autorisations à celle-ci.

## <a name="prerequisites"></a>Prérequis

Les privilèges Administrateur d’application sont nécessaires pour masquer une application du panneau MyApps et du Lanceur Microsoft 365.

Les privilèges Administrateur général sont nécessaires pour masquer toutes les applications Microsoft 365.

## <a name="hide-an-application-from-the-end-user"></a>Masquer une application à l’utilisateur final

Suivez ces étapes pour masquer une application dans le panneau MyApps et dans le lanceur d’applications Microsoft 365.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général de votre annuaire.
2. Sélectionnez **Azure Active Directory**.
3. Sélectionnez **Applications d’entreprise**. Le panneau **Applications d’entreprise - Toutes les applications** s’ouvre.
4. Sous **Type d’Application**, sélectionnez **Applications d’entreprise** si cette option n’est pas encore sélectionnée.
5. Recherchez l’application que vous souhaitez masquer, puis cliquez dessus.  La vue d’ensemble de l’application apparaît.
6. Cliquez sur **Propriétés**.
7. À la question **Visible par les utilisateurs ?** , cliquez sur **Non**.
8. Cliquez sur **Enregistrer**.

> [!NOTE]
> Ces instructions s’appliquent uniquement aux applications d’entreprise.

## <a name="use-azure-ad-powershell-to-hide-an-application"></a>Utiliser Azure AD PowerShell pour masquer une application

Pour masquer une application dans le panneau MyApps, vous pouvez ajouter manuellement la balise HideApp au principal de service pour l’application. Exécutez les commandes [AzureAD PowerShell](/powershell/module/azuread/#service_principals) suivantes pour définir la propriété **Visible par les utilisateurs ?** de l’application sur **Non**.

```PowerShell
Connect-AzureAD

$objectId = "<objectId>"
$servicePrincipal = Get-AzureADServicePrincipal -ObjectId $objectId
$tags = $servicePrincipal.tags
$tags += "HideApp"
Set-AzureADServicePrincipal -ObjectId $objectId -Tags $tags
```

## <a name="hide-microsoft-365-applications-from-the-myapps-panel"></a>Masquer des applications Microsoft 365 dans le panneau MyApps

Procédez comme suit pour masquer toutes les applications Microsoft 365 dans le panneau MyApps. Les applications sont toujours visibles dans le portail Office 365.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général de votre annuaire.
2. Sélectionnez **Azure Active Directory**.
3. Sélectionnez **Utilisateurs**.
4. Sélectionnez **Paramètres utilisateur**.
5. Sous **Applications d’entreprise**, cliquez sur **Gérer la façon dont les utilisateurs finaux lancent et affichent leurs applications**.
6. Pour **Les utilisateurs peuvent voir uniquement les applications Office 365 dans le portail Office 365**, cliquez sur **Oui**.
7. Cliquez sur **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

* [Voir tous mes groupes](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Affecter un utilisateur ou un groupe à une application d’entreprise](assign-user-or-group-access-portal.md)
* [Suppression d’une affectation d’utilisateur ou de groupe à partir d’une application d’entreprise](./assign-user-or-group-access-portal.md)
* [Modifier le nom ou le logo d’une application d’entreprise dans la version préliminaire d’Azure Active Directory](./add-application-portal-configure.md)
