---
title: Gérer l’attribution d’utilisateurs pour une application dans Azure Active Directory
description: Découvrez comment attribuer et désattribuer des utilisateurs et des groupes pour une application en utilisant Azure Active Directory pour la gestion des identités.
services: active-directory
author: davidmu1
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 02/21/2020
ms.author: davidmu
ms.reviewer: alamaral
ms.openlocfilehash: 91ffbc9e71da6a1837cd77efff3f0af435c437d1
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122532148"
---
# <a name="manage-user-assignment-for-an-app-in-azure-active-directory"></a>Gérer l’attribution d’utilisateurs pour une application dans Azure Active Directory

Cet article explique comment attribuer des utilisateurs et des groupes à des applications d’entreprise dans Azure Active Directory (Azure AD) à partir du portail Azure ou à l’aide de PowerShell. Lorsque vous attribuez un utilisateur à une application, celle-ci apparaît dans le volet [Mes applications](https://myapps.microsoft.com/) de l’utilisateur pour en faciliter l’accès. Si l’application expose des rôles, vous pouvez également attribuer un rôle spécifique à l’utilisateur.

Pour un meilleur contrôle, vous pouvez configurer certains types d’applications d’entreprise pour [exiger l’affectation d’utilisateur](#configure-an-application-to-require-user-assignment).

> [!IMPORTANT]
> Lorsque vous affectez un groupe à une application, seuls les utilisateurs du groupe y ont accès. L’affectation n’est pas en cascade vers les groupes imbriqués.

> [!NOTE]
> L’attribution basée sur le groupe requiert Azure Active Directory Premium édition P1 ou P2. L’attribution basée sur le groupe est uniquement prise en charge pour les groupes de sécurité. Les appartenances aux groupes imbriqués et aux groupes Microsoft 365 ne sont pas prises en charge actuellement. Pour d’autres conditions de gestion des licences relatives aux composants traités dans le présent article, consultez la [page sur la tarification d’Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory).

## <a name="configure-an-application-to-require-user-assignment"></a>Configurer une application pour exiger une affectation d’utilisateur

Avec les types d’applications suivants, vous avez la possibilité d’exiger que des utilisateurs soient affectés à l’application avant de pouvoir y accéder :

- applications configurées pour l’authentification unique (SSO) fédérée avec l’authentification basée sur SAML ;
- applications de proxy d’application qui utilisent la pré-authentification Azure Active Directory ;
- applications créées sur la plateforme d’application Azure AD qui utilisent l’authentification OAuth 2.0/OpenID Connect après qu’un utilisateur ou administrateur a donné son consentement à l’application.

Quand une affectation d’utilisateur est requise, seuls les utilisateurs que vous affectez explicitement à l’application (via une affectation directe ou une appartenance à un groupe) peuvent se connecter. Ils peuvent accéder à l’application via leur page Mes applications ou à l’aide d’un lien direct.

Lorsque l’affectation *n’est pas obligatoire*, soit parce que vous avez défini cette option sur **Non**, soit parce que l’application utilise un autre mode d’authentification unique, n’importe quel utilisateur peut accéder à l’application s’il dispose d’un lien direct vers celle-ci ou vers l’**URL d’accès utilisateur** dans la page **Propriétés** de l’application.

Ce paramètre n’a pas d’incidence sur l’apparition ou non d’une application dans le volet Mes applications. Les applications apparaissent dans les panneaux d’accès Mes applications des utilisateurs une fois que vous avez affecté un utilisateur ou un groupe à l’application. Pour plus d’informations, consultez [Gestion de l’accès aux applications](what-is-access-management.md).

> [!NOTE]
> Lorsqu’une application exige une affectation, le consentement de l’utilisateur n’est pas autorisé pour cette application. Cela est vrai même si ce consentement aurait autrement été autorisé pour l’application en question. Veillez à [accorder le consentement administrateur à l’échelle du locataire](../manage-apps/grant-admin-consent.md) aux applications qui exigent une affectation.

Pour exiger une affectation d’utilisateur pour une application :

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte d’administrateur ou en tant que propriétaire de l’application.
2. Sélectionnez **Azure Active Directory**. Dans le menu de navigation gauche, sélectionnez **Applications d’entreprise**.
3. Sélectionnez l’application dans la liste. Si vous ne voyez pas l’application, commencez à taper son nom dans la zone de recherche. Vous pouvez également utiliser des contrôles de filtre pour sélectionner le type, l’état ou la visibilité de l’application, puis sélectionner **Appliquer**.
4. Dans le menu de navigation gauche, sélectionnez **Nouveau**.
5. Assurez-vous que le bouton bascule **Affectation de l’utilisateur requise ?** est positionné sur **Oui**.
   > [!NOTE]
   > Si le bouton bascule **Affectation de l’utilisateur requise ?** n’est pas disponible, vous pouvez utiliser PowerShell pour définir la propriété appRoleAssignmentRequired sur le principal du service.
6. Sélectionnez le bouton **Enregistrer** en haut de l’écran.

## <a name="assign-or-unassign-users-and-groups-for-an-app-using-the-azure-portal"></a>Attribuer ou désattribuer des utilisateurs et des groupes pour une application en utilisant le portail Azure

Pour savoir comment attribuer ou désattribuer un utilisateur ou un groupe à l’aide du portail Azure, consultez [Série de démarrages rapides sur la gestion des applications](add-application-portal-assign-users.md).

## <a name="assign-or-unassign-users-and-groups-for-an-app-using-the-graph-api"></a>Attribuer ou désattribuer des utilisateurs et des groupes pour une application à l’aide de l’API Graph

Vous pouvez utiliser l’API Graph pour attribuer ou désattribuer des utilisateurs et des groupes pour une application. Pour plus d’informations, consultez [Attributions de rôles d’application](/graph/api/resources/approleassignment).

## <a name="assign-users-and-groups-to-an-app-using-powershell"></a>Attribuer des utilisateurs et des groupes à une application à l’aide de PowerShell

1. Ouvrez une invite de commandes Windows PowerShell avec des privilèges élevés.
   > [!NOTE]
   > Vous devez installer le module AzureAD (utilisez la commande `Install-Module -Name AzureAD`). Si vous y êtes invité, installez un module NuGet ou le nouveau module Azure Active Directory V2 PowerShell, tapez O et appuyez sur Entrée.
2. Exécutez `Connect-AzureAD` et connectez-vous avec un compte d’utilisateur Administrateur général.
3. Pour affecter un utilisateur et un rôle à une application, utilisez le script suivant :

    ```powershell
    # Assign the values to the variables
    $username = "<Your user's UPN>"
    $app_name = "<Your App's display name>"
    $app_role_name = "<App role display name>"

    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }

    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id

For more information about how to assign a user to an application role, see the documentation for [New-AzureADUserAppRoleAssignment](/powershell/module/azuread/new-azureaduserapproleassignment).

To assign a group to an enterprise app, you must replace `Get-AzureADUser` with `Get-AzureADGroup` and replace `New-AzureADUserAppRoleAssignment` with `New-AzureADGroupAppRoleAssignment`.

For more information about how to assign a group to an application role, see the documentation for [New-AzureADGroupAppRoleAssignment](/powershell/module/azuread/new-azureadgroupapproleassignment).

### Example

This example assigns the user Britta Simon to the [Microsoft Workplace Analytics](https://products.office.com/business/workplace-analytics) application using PowerShell.

1. In PowerShell, assign the corresponding values to the variables $username, $app_name and $app_role_name.

    ```powershell
    # Assign the values to the variables
    $username = "britta.simon@contoso.com"
    $app_name = "Workplace Analytics"

2. In this example, we don't know what is the exact name of the application role we want to assign to Britta Simon. Run the following commands to get the user ($user) and the service principal ($sp) using the user UPN and the service principal display names.

    ```powershell
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"

3. Run the command `$sp.AppRoles` to display the roles available for the Workplace Analytics application. In this example, we want to assign Britta Simon the Analyst (Limited access) Role.
   ![Shows the roles available to a user using Workplace Analytics Role](./media/assign-user-or-group-access-portal/workplace-analytics-role.png)
4. Assign the role name to the `$app_role_name` variable.

    ```powershell
    # Assign the values to the variables
    $app_role_name = "Analyst (Limited access)"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }

5. Run the following command to assign the user to the app role:

    ```powershell
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```

## <a name="unassign-users-and-groups-from-an-app-using-powershell"></a>Désattribuer des utilisateurs et des groupes d’une application à l’aide de PowerShell

1. Ouvrez une invite de commandes Windows PowerShell avec des privilèges élevés.
   > [!NOTE]
   > Vous devez installer le module AzureAD (utilisez la commande `Install-Module -Name AzureAD`). Si vous y êtes invité, installez un module NuGet ou le nouveau module Azure Active Directory V2 PowerShell, tapez O et appuyez sur Entrée.
2. Exécutez `Connect-AzureAD` et connectez-vous avec un compte d’utilisateur Administrateur général.
3. Pour supprimer un utilisateur et un rôle d’une application, utilisez le script suivant :

    ```powershell
    # Store the proper parameters
    $user = get-azureaduser -ObjectId <objectId>
    $spo = Get-AzureADServicePrincipal -ObjectId <objectId>

    #Get the ID of role assignment 
    $assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $spo.ObjectId | Where {$_.PrincipalDisplayName -eq $user.DisplayName}

    #if you run the following, it will show you what is assigned what
    $assignments | Select *

    #To remove the App role assignment run the following command.
    Remove-AzureADServiceAppRoleAssignment -ObjectId $spo.ObjectId -AppRoleAssignmentId $assignments[assignment #].ObjectId
    ```

## <a name="related-articles"></a>Articles connexes

- [En savoir plus sur l’accès des utilisateurs finaux aux applications](end-user-experiences.md)
- [Planifier un déploiement de Mes applications Azure AD](my-apps-deployment-plan.md)
- [Gestion de l’accès aux applications](what-is-access-management.md)

## <a name="next-steps"></a>Étapes suivantes

- [Voir tous mes groupes](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Suppression d’une affectation d’utilisateur ou de groupe à partir d’une application d’entreprise]()
- [Désactiver les connexions utilisateur pour une application d’entreprise](disable-user-sign-in-portal.md)
- [Modifier le nom ou le logo d’une application d’entreprise dans la version préliminaire d’Azure Active Directory](./add-application-portal-configure.md)
