---
title: Identités managées pour les ressources Azure avec Service Bus
description: Cet article explique comment utiliser des identités managées pour accéder aux entités Azure Service Bus (files d’attente, rubriques et abonnements).
ms.topic: article
ms.date: 06/14/2021
ms.custom: subject-rbac-steps
ms.openlocfilehash: ed6f7d495466139a7d1a98aed7d5323f7ad4c074
ms.sourcegitcommit: 0af634af87404d6970d82fcf1e75598c8da7a044
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/15/2021
ms.locfileid: "112123210"
---
# <a name="authenticate-a-managed-identity-with-azure-active-directory-to-access-azure-service-bus-resources"></a>Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Azure Service Bus
La fonctionnalité [Identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) vous permet de créer une identité sécurisée associée au déploiement sous lequel s’exécute le code de votre application. Vous pouvez ensuite associer cette identité à des rôles de contrôle d’accès qui accordent des autorisations personnalisées pour l’accès aux ressources Azure nécessaires à votre application.

Avec les identités managées, la plateforme Azure gère cette identité d’exécution. Vous n’avez pas besoin de stocker et de protéger des clés d’accès dans le code ou la configuration de votre application, que ce soit pour l’identité elle-même ou pour les ressources auxquelles vous devez accéder. Une application cliente Service Bus exécutée dans une application Azure App Service ou dans une machine virtuelle prenant en charge les identités managées pour les ressources Azure n’a pas besoin de gérer des clés et des règles SAS, ou d’autres jetons d’accès. L’application cliente a uniquement besoin de l’adresse de point de terminaison de l’espace de noms Service Bus Messaging. Quand l’application se connecte, Service Bus lie le contexte de l’entité managée au client dans une opération illustrée plus loin dans cet article. Une fois associé à une identité managée, votre client Service Bus peut effectuer toutes les opérations autorisées. L’autorisation est accordée en associant une identité managée à des rôles Service Bus. 

## <a name="overview"></a>Vue d’ensemble
Quand un principal de sécurité (un utilisateur, un groupe ou une application) tente d’accéder à une entité Service Bus, la requête doit être autorisée. Avec Azure AD, l’accès à une ressource est un processus en deux étapes. 

 1. Pour commencer, l’identité du principal de sécurité est authentifiée, et un jeton OAuth 2.0 est renvoyé. Le nom de ressource à utiliser pour demander un jeton est `https://servicebus.azure.net`.
 1. Ensuite, ce jeton est transmis dans une requête adressée au service Service Bus pour autoriser l’accès à la ressource spécifiée.

L’étape d’authentification nécessite qu’une requête d’application contienne un jeton d’accès OAuth 2.0 au moment de l’exécution. Si une application s’exécute à partir d’une entité Azure telle qu’une machine virtuelle Azure, un groupe de machines virtuelles identiques ou une application Azure Function, elle peut utiliser une identité managée pour accéder aux ressources. 

L’étape d’autorisation exige qu’un ou plusieurs rôles Azure soient attribués au principal de sécurité. Azure Service Bus fournit des rôles Azure qui englobent des ensembles d’autorisations pour les ressources Service Bus. Les rôles qui sont attribués à un principal de sécurité déterminent les autorisations dont disposera le principal. Pour plus d’informations sur l’attribution de rôles Azure dans Azure Service Bus, consultez [Rôles Azure intégrés pour Azure Service Bus](#azure-built-in-roles-for-azure-service-bus). 

Les applications natives et applications web qui adressent des requêtes à Service Bus peuvent également autoriser l’accès avec Azure AD. Cet article explique comment demander un jeton d’accès et l’utiliser pour autoriser les demandes de ressources Service Bus. 


## <a name="assigning-azure-roles-for-access-rights"></a>Attribution de rôles Azure pour les droits d’accès
Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md). Azure Service Bus définit un ensemble de rôles Azure intégrés qui englobent les ensembles d’autorisations couramment utilisés pour accéder aux entités Service Bus. Il est également possible de définir des rôles personnalisés pour accéder aux données.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources ou de l’espace de noms Service Bus. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un groupe, à un principal de service d’application ou à une identité managée pour les ressources Azure.

## <a name="azure-built-in-roles-for-azure-service-bus"></a>Rôles Azure intégrés pour Azure Service Bus
Pour Azure Service Bus, la gestion des espaces de noms et de toutes les ressources associées via le Portail Azure et l’API de gestion des ressources Azure est déjà protégée à l’aide du modèle Azure RBAC. Azure fournit les rôles Azure intégrés ci-dessous pour autoriser l’accès à un espace de noms Service Bus :

- [Propriétaire de données Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner) : permet l’accès aux données de l’espace de noms Service Bus et ses entités (files d’attente, rubriques, abonnements et filtres).
- [Expéditeur de données Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-sender) : utilisez ce rôle pour autoriser l’accès en envoi à l’espace de noms Service Bus et à ses entités.
- [Récepteur de données Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-receiver) : utilisez ce rôle pour autoriser l’accès en réception à l’espace de noms Service Bus et à ses entités. 

## <a name="resource-scope"></a>Étendue des ressources 
Avant d’attribuer un rôle Azure à un principal de sécurité, déterminez l’étendue de l’accès dont doit disposer le principal de sécurité. Selon les bonnes pratiques, il est toujours préférable d’accorder la plus petite étendue possible.

La liste suivante décrit les niveaux auxquels vous pouvez étendre l’accès aux ressources Service Bus, en commençant par la plus petite étendue :

- **File d’attente**, **rubrique** ou **abonnement** : l’attribution de rôle s’applique à l’entité Service Bus spécifique. Actuellement, le Portail Azure ne prend pas en charge l’affectation d’utilisateurs, de groupes ou d’identités managées aux rôles Azure Service Bus au niveau de l’abonnement. Voici un exemple d’utilisation de la commande Azure CLI : [az-role-assignment-create](/cli/azure/role/assignment?#az_role_assignment_create) pour attribuer une identité à un rôle Azure Service Bus : 

    ```azurecli
    az role assignment create \
        --role $service_bus_role \
        --assignee $assignee_id \
        --scope /subscriptions/$subscription_id/resourceGroups/$resource_group/providers/Microsoft.ServiceBus/namespaces/$service_bus_namespace/topics/$service_bus_topic/subscriptions/$service_bus_subscription
    ```
- **Espace de noms Service Bus** : l’attribution de rôle s’étend à toute la topologie de Service Bus sous l’espace de noms et au groupe de consommateurs qui lui est associé.
- **Groupe de ressources** : l’attribution de rôle s’applique à toutes les ressources Service Bus sous le groupe de ressources.
- **Abonnement**: l’attribution de rôle s’applique à toutes les ressources Service Bus dans tous les groupes de ressources de l’abonnement.

> [!NOTE]
> Gardez à l’esprit que la propagation des attributions de rôles Azure peut prendre cinq minutes. 

Pour plus d’informations sur la définition des rôles intégrés, consultez [Comprendre les définitions de rôles](../role-based-access-control/role-definitions.md#management-and-data-operations). Pour plus d’informations sur la création de rôles personnalisés Azure, consultez [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).

## <a name="enable-managed-identities-on-a-vm"></a>Activer les identités managées sur une machine virtuelle
Avant de pouvoir utiliser les identités managées pour ressources Azure en vue d’autoriser les ressources Service Bus à partir de votre machine virtuelle, vous devez activer les identités managées pour ressources Azure sur la machine virtuelle. Pour savoir comment activer des identités managées pour ressources Azure, consultez un de ces articles :

- [Azure portal](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Modèle Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliothèques clientes Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-a-managed-identity-in-azure-ad"></a>Octroyer des autorisations à une identité managée dans Azure AD
Pour autoriser une requête auprès du service Service Bus à partir d’une identité managée dans votre application, commencez par configurer les paramètres du contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour cette identité managée. Azure Service Bus définit les rôles Azure qui englobent les autorisations d’envoi et de lecture à partir de Service Bus. Quand le rôle Azure est attribué à une identité managée, celle-ci est autorisée à accéder aux entités Service Bus au niveau d’étendue approprié.

Pour plus d’informations sur l’attribution de rôles Azure, consultez [Authentifier et autoriser avec Azure Active Directory pour accéder aux ressources Service Bus](authenticate-application.md#azure-built-in-roles-for-azure-service-bus).

## <a name="use-service-bus-with-managed-identities-for-azure-resources"></a>Utiliser Service Bus avec des identités managées pour les ressources Azure
Pour utiliser Service Bus avec des identités managées, vous devez attribuer le rôle et l’étendue appropriés à l’identité. La procédure décrite dans cette section utilise une application simple qui s’exécute sous une identité managée et accède aux ressources Service Bus.

Ici, nous utilisons un exemple d’application web hébergée dans [Azure App Service](https://azure.microsoft.com/services/app-service/). Pour obtenir des instructions pas à pas sur la création d’une application web, consultez [Créer une application web ASP.NET Core dans Azure](../app-service/quickstart-dotnetcore.md)

Une fois que vous avez créé l’application, suivez ces étapes : 

1. Accédez à **Paramètres** et sélectionnez **Identité**. 
1. Définissez **État** sur **Activé**. 
1. Sélectionnez **Enregistrer** pour enregistrer le paramètre. 

    ![Identité managée pour une application web](./media/service-bus-managed-service-identity/identity-web-app.png)

Une fois ce paramètre activé, une identité de service est créée dans votre annuaire Azure Active Directory (Azure AD) et configurée dans l’hôte App Service.

### <a name="to-assign-azure-roles-using-the-azure-portal"></a>Pour attribuer des rôles Azure à l’aide du portail Azure
Attribuez un des [rôles Service Bus](#azure-built-in-roles-for-azure-service-bus) à l’identité du service managé à l’étendue souhaitée (espace de noms Service Bus, groupe de ressources, abonnement). Pour connaître les étapes détaillées, consultez [Attribuer des rôles Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md). 

> [!NOTE]
> Pour obtenir la liste des services qui prennent en charge les identités managées, consultez [Services qui prennent en charge les identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md).

### <a name="run-the-app"></a>Exécuter l’application
À présent, modifiez la page par défaut de l’application ASP.NET que vous avez créée. Vous pouvez utiliser le code de l’application web qui se trouve sur [ce référentiel GitHub](https://github.com/Azure-Samples/app-service-msi-servicebus-dotnet).  

La page Default.aspx est votre page d’accueil. Le code se trouve dans le fichier Default.aspx.cs. Le résultat est une application web minimale avec quelques champs d’entrée et les boutons **send** (envoyer) et **receive** (recevoir) qui permettent de se connecter à Service Bus pour envoyer ou recevoir des messages.

Notez la façon dont l’objet [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) est initialisé à l’aide d’un constructeur qui prend un TokenCredential. DefaultAzureCredential dérive de TokenCredential et peut être transmis ici. Ainsi, il n’est pas nécessaire de conserver et d’utiliser des secrets. Le flux du contexte de l’identité managée vers Service Bus et l’établissement d’une liaison d’autorisation sont gérés automatiquement par les informations d’identification du jeton. C’est un modèle plus simple que l’utilisation de SAS.

Après avoir apporté ces modifications, publiez et exécutez l’application. Pour obtenir facilement les données de publication correctes, téléchargez un profil de publication, puis importez-le dans Visual Studio :

![Obtenir le profil de publication](./media/service-bus-managed-service-identity/msi3.png)
 
Pour envoyer ou recevoir des messages, saisissez le nom de l’espace de noms et le nom de l’entité que vous avez créée. Cliquez ensuite sur **envoyer** ou **recevoir**.


> [!NOTE]
> - L’identité managée ne fonctionne qu’au sein de l’environnement Azure, d’App Services, des machines virtuelles Azure et des groupes identiques. Pour les applications .NET, la bibliothèque Microsoft.Azure.Services.AppAuthentication, utilisée par le package NuGet Service Bus, représente une abstraction sur ce protocole et prend en charge une expérience de développement local. Elle vous permet également de tester votre code localement sur votre ordinateur de développement, avec votre compte d’utilisateur issu de Visual Studio, d’Azure CLI 2.0 ou de l’authentification intégrée Azure Active Directory. Pour plus d’informations sur les options de développement local avec cette bibliothèque, voir [Authentification de service à service à Azure Key Vault avec .NET](/dotnet/api/overview/azure/service-to-service-authentication).  

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
