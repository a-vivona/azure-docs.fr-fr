---
title: Désactiver les connexions utilisateur pour une application d’entreprise dans Azure AD
description: Comment désactiver une application d’entreprise afin qu’aucun utilisateur ne puisse s’y connecter dans Azure Active Directory
services: active-directory
author: davidmu1
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 04/12/2019
ms.author: davidmu
ms.reviewer: alamaral
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7ca695fc0e84f3db3b98c3e25333d58279ad53f1
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122524714"
---
# <a name="disable-user-sign-ins-for-an-enterprise-app-in-azure-active-directory"></a>Désactiver les connexions utilisateur pour une application d’entreprise dans Azure Active Directory

Il est facile de désactiver une application d’entreprise afin qu’aucun utilisateur ne puisse s’y connecter dans Azure Active Directory (Azure AD). Vous devez disposer des autorisations appropriées pour gérer l’application d’entreprise. En outre, vous devez être administrateur général du répertoire.

## <a name="how-do-i-disable-user-sign-ins"></a>Comment désactiver les connexions des utilisateurs ?

1. Connectez-vous au [portail Azure](https://portal.azure.com) en utilisant un compte d’administrateur général pour le répertoire.
1. Sélectionnez **Tous les services**, entrez **Azure Active Directory** dans la zone de texte, puis sélectionnez **Entrée**.
1. Dans le volet **Azure Active Directory** -  **_NomRépertoire_ *_ (autrement dit, le volet Azure AD du répertoire que vous gérez), sélectionnez _* Applications d’entreprise**.
1. Dans le volet **Applications d’entreprise - Toutes les applications**, vous voyez une liste des applications que vous pouvez gérer. Sélectionnez une application.
1. Dans le volet ***appname** _ (c’est-à-dire le volet dont le nom de l’application sélectionnée se trouve dans le titre), sélectionnez _*Propriétés**.
1. Dans le volet ***appname** _ - _ volet *Propriétés**, sélectionnez **Non** pour **Activé pour la connexion des utilisateurs ?** .
1. Sélectionnez la commande **Enregistrer** .

## <a name="use-azure-ad-powershell-to-disable-an-unlisted-app"></a>Utiliser Azure AD PowerShell pour désactiver une application non répertoriée

Si vous connaissez l’AppId d’une application qui n’apparaît pas dans la liste des applications d’entreprise (par exemple, parce que vous avez supprimé l’application ou que le principal du service n’a pas encore été créé, car l’application est en cours de préautorisation par Microsoft), vous pouvez créer manuellement le principal du service pour l’application, puis le désactiver à l’aide de la [cmdlet Azure AD PowerShell](/powershell/module/azuread/New-AzureADServicePrincipal).

```PowerShell
# The AppId of the app to be disabled
$appId = "{AppId}"

# Check if a service principal already exists for the app
$servicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$appId'"
if ($servicePrincipal) {
    # Service principal exists already, disable it
    Set-AzureADServicePrincipal -ObjectId $servicePrincipal.ObjectId -AccountEnabled $false
} else {
    # Service principal does not yet exist, create it and disable it at the same time
    $servicePrincipal = New-AzureADServicePrincipal -AppId $appId -AccountEnabled $false
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Voir tous mes groupes](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Affecter un utilisateur ou un groupe à une application d’entreprise](assign-user-or-group-access-portal.md)
* [Suppression d’une affectation d’utilisateur ou de groupe à partir d’une application d’entreprise](./assign-user-or-group-access-portal.md)
* [Modifier le nom ou le logo d’une application d’entreprise dans la version préliminaire d’Azure Active Directory](./add-application-portal-configure.md)
