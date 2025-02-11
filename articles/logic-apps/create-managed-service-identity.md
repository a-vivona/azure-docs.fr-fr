---
title: Authentifier les flux de travail avec des identités managées
description: Utiliser une identité managée pour authentifier les déclencheurs et les actions pour les ressources protégées Azure AD sans informations d’identification ou secrets
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: article
ms.date: 06/25/2021
ms.custom: devx-track-azurepowershell, subject-rbac-steps
ms.openlocfilehash: 76edcac6b77b70928cb2d6cd378b421b68b3d3ef
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122531778"
---
# <a name="authenticate-access-to-azure-resources-using-managed-identities-in-azure-logic-apps"></a>Authentifier l’accès aux ressources Azure avec des identités managées dans Azure Logic Apps

Pour la prise en charge, certains déclencheurs et actions pour les flux de travail d’application logique utilisent une [identité managée](../active-directory/managed-identities-azure-resources/overview.md), précédemment connue sous le nom d’*Identité de service managée (MSI)* , pour l’authentification lors de la connexion à des ressources protégées par Azure Active Directory (Azure AD). Lorsque la ressource de votre application logique a une identité managée activée et configurée, vous n’êtes pas obligé d’utiliser vos propres informations d’identification, secrets ou jetons Azure AD. Azure gère cette identité et permet de sécuriser les informations d’authentification dans la mesure où vous n’avez à gérer ni les secrets ni les jetons.

Cet article montre comment configurer les deux genres d’identité managée pour votre application logique. Pour plus d’informations, consultez la documentation suivante :

* [Déclencheurs et actions qui prennent en charge les identités managées](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions)
* [Limites sur les identités managées pour les applications logiques](../logic-apps/logic-apps-limits-and-config.md#managed-identity)
* [Services Azure qui prennent en charge l’authentification Azure AD avec des identités managées](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication)

<a name="triggers-actions-managed-identity"></a>

## <a name="where-to-use-managed-identities"></a>Où utiliser les identités managées ?

Azure Logic Apps prend en charge les [*identités managées* affectées par le système](../active-directory/managed-identities-azure-resources/overview.md) et les [*identités managées* affectées par l’utilisateur](../active-directory/managed-identities-azure-resources/overview.md), que vous pouvez partager dans un groupe d’applications logiques, selon l’emplacement d’exécution de vos flux de travail d’application logique :

* Une application logique basée multilocataire (plan de consommation) prend en charge l’identité affectée par le système et une identité affectée par l’utilisateur *unique*. Toutefois, au niveau de l’application logique ou au niveau de la connexion, vous ne pouvez utiliser qu’un seul type d’identité managée, car vous ne pouvez pas activer les deux à la fois.

  Une application logique à locataire unique (plan standard) prend actuellement en charge uniquement l’identité affectée par le système.

  Pour plus d’informations sur les applications multilocataire (plan de consommation) et à locataire unique (plan standard), consultez la documentation [Comparaison entre locataire unique et multilocataire et environnement de service d’intégration](single-tenant-overview-compare.md).

<a name="built-in-managed-identity"></a>
<a name="managed-connectors-managed-identity"></a>

* Seules les opérations de connecteur intégrées et managées spécifiques qui prennent en charge l’authentification ouverte Azure AD peuvent utiliser une identité managée pour l’authentification. Le tableau suivant fournit uniquement une *sélection d’exemples*. Pour obtenir une liste plus complète, consultez [Types d’authentification pour les déclencheurs et les actions qui prennent en charge l’authentification](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

  | Type d'opération | Opérations prises en charge |
  |----------------|----------------------|
  | Intégré | - Gestion des API Azure <br>- Azure App Services <br>- Azure Functions <br>- HTTP <br>- HTTP + Webhook <p><p> **Remarque** : même si les opérations HTTP peuvent authentifier les connexions aux comptes de Stockage Azure derrière des pare-feu Azure à l’aide de l’identité managée affectée par le système, elles ne prennent pas en charge l’identité managée affectée par l’utilisateur pour authentifier les mêmes connexions. |
  | Connecteur managé (**Préversion**) | - Azure Automation <br>- Azure Event Grid <br>- Azure Key Vault <br>- Azure Resource Manager <br>- HTTP avec Azure AD |
  |||

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/). L’identité managée et la ressource Azure cible à laquelle vous souhaitez accéder doivent utiliser le même abonnement Azure.

* Pour accorder à une identité managée l’accès à une ressource Azure, vous devez ajouter un rôle à la ressource cible pour cette identité. Pour ajouter des rôles, vous devez disposer d’[autorisations d’administrateur Azure AD](../active-directory/roles/permissions-reference.md) qui permettent d’attribuer des rôles à des identités dans le locataire Azure AD correspondant.

* Ressource Azure cible à laquelle vous souhaitez accéder. Dans cette ressource, vous allez ajouter un rôle pour l’identité managée, ce qui permet à l’application logique d’authentifier l’accès à la ressource cible.

* Application logique où vous souhaitez utiliser le [déclencheur ou les actions qui prennent en charge les identités managées](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

## <a name="enable-managed-identity"></a>Activer une identité managée

Pour configurer l’identité managée à utiliser, suivez le lien de cette identité :

* [Identité affectée par le système](#system-assigned)
* [Identité affectée par l’utilisateur](#user-assigned)

<a name="system-assigned"></a>

### <a name="enable-system-assigned-identity"></a>Activer l’identité attribuée par le système

À la différences des identités attribuées par l’utilisateur, vous n’avez pas besoin de créer manuellement l’identité attribuée par le système. Pour configurer l’identité affectée par le système dans le cadre de votre application logique, voici les options que vous pouvez utiliser :

* [Azure portal](#azure-portal-system-logic-app)
* [Modèle Azure Resource Manager (modèle ARM)](#template-system-logic-app)

<a name="azure-portal-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-azure-portal"></a>Activer une identité attribuée par le système sur le portail Azure

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez votre application logique dans le nouveau concepteur.

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Identité**. Sélectionnez **Affecté(e) par le système** > **Actif** > **Enregistrer**. Quand Azure vous invite à confirmer l’opération, sélectionnez **Oui**.

   ![Activer l’identité attribuée par le système](./media/create-managed-service-identity/enable-system-assigned-identity.png)

   > [!NOTE]
   > Si vous recevez une erreur indiquant que vous ne pouvez avoir qu’une seule identité managée, cela signifie que votre application logique est déjà associée à l’identité affectée par l’utilisateur. Avant de pouvoir ajouter l’identité affectée par le système, vous devez d’abord *supprimer* l’identité affectée par l’utilisateur de votre application logique.

   Votre application logique peut désormais utiliser l’identité affectée par le système qui est inscrite auprès d’Azure AD et représentée par un ID d’objet.

   ![ID d’objet pour l’identité attribuée par le système](./media/create-managed-service-identity/object-id-system-assigned-identity.png)

   | Propriété | Valeur | Description |
   |----------|-------|-------------|
   | **ID d’objet** | <*identity-resource-ID*> | GUID (identificateur global unique) qui représente l’identité affectée par le système pour votre application logique dans un locataire Azure AD |
   ||||

1. À présent, suivez les [étapes qui donnent à cette identité l’accès à la ressource](#access-other-resources), et que vous trouverez plus loin dans cette rubrique.

<a name="template-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-an-arm-template"></a>Activer une identité affectée par le système dans un modèle ARM

Pour automatiser la création et le déploiement de ressources Azure telles que des applications logiques, vous pouvez utiliser un [modèle ARM](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md). Pour activer l’identité managée attribuée par le système pour votre application logique dans le modèle, ajoutez l’objet `identity` et la propriété enfant `type` à la définition de ressource de l’application logique dans le modèle, par exemple :

```json
{
   "apiVersion": "2016-06-01",
   "type": "Microsoft.logic/workflows",
   "name": "[variables('logicappName')]",
   "location": "[resourceGroup().location]",
   "identity": {
      "type": "SystemAssigned"
   },
   "properties": {
      "definition": {
         "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
         "actions": {},
         "parameters": {},
         "triggers": {},
         "contentVersion": "1.0.0.0",
         "outputs": {}
   },
   "parameters": {},
   "dependsOn": []
}
```

Quand Azure crée la définition de ressource de votre application logique, l’`identity`objet obtient les propriétés supplémentaires suivantes :

```json
"identity": {
   "type": "SystemAssigned",
   "principalId": "<principal-ID>",
   "tenantId": "<Azure-AD-tenant-ID>"
}
```

| Propriété (JSON) | Valeur | Description |
|-----------------|-------|-------------|
| `principalId` | <*principal-ID*> | Identificateur global unique (GUID) de l’objet du principal de service pour l’identité managée qui représente votre application logique dans le locataire Azure AD. Ce GUID apparaît parfois sous la forme d’un « ID d’objet » ou `objectID`. |
| `tenantId` | <*Azure-AD-tenant-ID*> | Identificateur global unique (GUID) représentant le locataire Azure AD où l’application logique est maintenant membre. À l’intérieur du locataire Azure AD, le principal du service a le même nom que l’instance d’application logique. |
||||

<a name="user-assigned"></a>

### <a name="enable-user-assigned-identity"></a>Activer l’identité affectée par l’utilisateur

Pour configurer une identité managée affectée par l’utilisateur dans le cadre de votre application logique, vous devez d’abord créer cette identité en tant que ressource Azure autonome distincte. Voici les options que vous pouvez utiliser :

* [Azure portal](#azure-portal-user-identity)
* [Modèle ARM](#template-user-identity)
* Azure PowerShell
  * [Créer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
  * [Ajouter une attribution de rôle](../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md)
* Azure CLI
  * [Créer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
  * [Ajouter une attribution de rôle](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md)
* API REST Azure
  * [Créer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)
  * [Ajouter une attribution de rôle](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-user-identity"></a>

#### <a name="create-user-assigned-identity-in-the-azure-portal"></a>Créer une identité affectée par l’utilisateur dans le portail Azure

1. Dans le [portail Azure](https://portal.azure.com), dans la zone de recherche de n’importe quelle page, entrez `managed identities`, puis sélectionnez **Identités managées**.

   ![Capture d’écran montrant le portail avec l’option « Identités managées » sélectionnée.](./media/create-managed-service-identity/find-select-managed-identities.png)

1. Sous **Identités managées**, sélectionnez **Ajouter**.

   ![Ajouter une nouvelle identité managée](./media/create-managed-service-identity/add-user-assigned-identity.png)

1. Fournissez des informations sur votre identité managée, puis sélectionnez **Vérifier + créer**, par exemple :

   ![Créer une identité managée affectée par l’utilisateur](./media/create-managed-service-identity/create-user-assigned-identity.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Abonnement** | Oui | <*Azure-subscription-name*> | Nom de l’abonnement Azure à utiliser. |
   | **Groupe de ressources** | Oui | <*nom-groupe-de-ressources-Azure*> | Nom du groupe de ressources à utiliser. Créez un groupe, ou sélectionnez un groupe existant. Cet exemple crée un groupe nommé `fabrikam-managed-identities-RG`. |
   | **Région** | Oui | <*Azure-region*> | Région Azure où stocker les informations relatives à votre ressource. Cet exemple utilise la région « USA Ouest ». |
   | **Nom** | Oui | <*nom-identité-affectée-par-utilisateur*> | Nom à donner à votre identité affectée par l’utilisateur. Cet exemple utilise `Fabrikam-user-assigned-identity`. |
   |||||

   Une fois ces détails validés, Azure crée votre identité managée. À présent, vous pouvez ajouter l’identité affectée par l’utilisateur à votre application logique. Vous ne pouvez pas ajouter plusieurs identités affectées par l’utilisateur à votre application logique.

1. Dans le Portail Azure, ouvrez votre application logique dans le nouveau concepteur.

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Identité**, puis **Affecté(e) par l’utilisateur** > **Ajouter**.

   ![Ajouter une identité managée affectée par l’utilisateur](./media/create-managed-service-identity/add-user-assigned-identity-logic-app.png)

1. Dans le volet **Ajouter une identité managée affectée par l’utilisateur**, dans la liste **Abonnement**, sélectionnez votre abonnement Azure, si cela n’est pas déjà fait. Dans la liste qui montre *toutes* les identités managées de cet abonnement, sélectionnez l’identité affectée par l’utilisateur qui vous intéresse. Pour filtrer la liste, dans la zone de recherche **Identités managées affectées par l’utilisateur**, entrez le nom de l’identité ou du groupe de ressources. Une fois que vous avez terminé, sélectionnez **Ajouter**.

   ![Sélectionner l’identité affectée par l’utilisateur à employer](./media/create-managed-service-identity/select-user-assigned-identity.png)

   > [!NOTE]
   > Si vous recevez une erreur indiquant que vous ne pouvez avoir qu’une seule identité managée, cela signifie que votre application logique est déjà associée à l’identité affectée par le système. Avant de pouvoir ajouter l’identité affectée par l’utilisateur, vous devez d’abord désactiver l’identité affectée par le système sur votre application logique.

   Votre application logique est désormais associée à l’identité managée affectée par l’utilisateur.

   ![Association à l’identité affectée par l’utilisateur](./media/create-managed-service-identity/added-user-assigned-identity.png)

1. À présent, suivez les [étapes qui donnent à cette identité l’accès à la ressource](#access-other-resources), et que vous trouverez plus loin dans cette rubrique.

<a name="template-user-identity"></a>

#### <a name="create-user-assigned-identity-in-an-arm-template"></a>Créer une identité affectée par l’utilisateur dans un modèle ARM

Pour automatiser la création et le déploiement de ressources Azure telles que les applications logiques, vous pouvez utiliser un [modèle ARM](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), lequel prend en charge les [identités affectées par l’utilisateur pour l’authentification](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md). Dans la section `resources` de votre modèle, la définition de ressource de votre application logique nécessite les éléments suivants :

* Objet `identity` dont la propriété `type` a la valeur `UserAssigned`

* Objet `userAssignedIdentities` enfant qui spécifie la ressource et le nom affectés par l’utilisateur

Cet exemple montre une définition de ressource d’application logique pour une requête HTTP PUT, et comprend un objet `identity` non paramétré. La réponse à la requête PUT et à l’opération GET qui suit comporte également cet objet `identity` :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicappName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user-assigned-identity-name>": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": []
      },
   ],
   "outputs": {}
}
```

Si votre modèle inclut également la définition de ressource de l’identité managée, vous pouvez paramétrer l’objet `identity`. Cet exemple montre comment l’objet `userAssignedIdentities` enfant référence une variable `userAssignedIdentity` que vous définissez dans la section `variables` de votre modèle. Cette variable référence l’ID de ressource de l’identité affectée par l’utilisateur.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "Template_LogicAppName": {
         "type": "string"
      },
      "Template_UserAssignedIdentityName": {
         "type": "securestring"
      }
   },
   "variables": {
      "logicAppName": "[parameters(`Template_LogicAppName')]",
      "userAssignedIdentityName": "[parameters('Template_UserAssignedIdentityName')]"
   },
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicAppName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": [
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]"
         ]
      },
      {
         "apiVersion": "2018-11-30",
         "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
         "name": "[parameters('Template_UserAssignedIdentityName')]",
         "location": "[resourceGroup().location]",
         "properties": {}
      }
  ]
}
```

<a name="access-other-resources"></a>

## <a name="give-identity-access-to-resources"></a>Accorder à une identité l’accès aux ressources

Avant de pouvoir utiliser l’identité gérée de votre application logique pour l’authentification, sur la ressource Azure dans laquelle vous souhaitez utiliser l’identité, vous devez configurer l’accès à votre identité à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC).

Pour effectuer cette tâche, attribuez le rôle approprié à cette identité sur la ressource Azure via l’une des options suivantes :

* [Azure portal](#azure-portal-assign-access)
* [Modèle ARM](../role-based-access-control/role-assignments-template.md)
* [Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)
* [Azure CLI](../role-based-access-control/role-assignments-cli.md)
* [API REST Azure](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-assign-access"></a>

### <a name="assign-managed-identity-role-based-access-in-the-azure-portal"></a>Attribuer un accès d’identité managée basée sur le rôle dans le Portail Azure

Sur la ressource Azure dans laquelle vous souhaitez utiliser l’identité managée, vous devez affecter votre identité à un rôle qui peut accéder à la ressource cible. Pour plus d’informations générales sur cette tâche, consultez [Attribuer un accès d’identité managée à une autre ressource à l’aide du RBAC Azure](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md).

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez la ressource où vous souhaitez utiliser l’identité.

1. Dans le menu de la ressource, sélectionnez **Contrôle d’accès (IAM)**  > **Ajouter** > **Ajouter une attribution de rôle**.

   > [!NOTE]
   > Si l’option **Ajouter une attribution de rôle** est désactivée, vous n’avez pas les autorisations pour attribuer les rôles. Pour plus d’informations, consultez [Rôles intégrés Azure AD](../active-directory/roles/permissions-reference.md).

1. À présent, attribuez le rôle nécessaire à votre identité managée. Dans l’onglet **Rôle**, attribuez un rôle à votre identité qui lui permette d’accéder à la ressource actuelle.

   Pour cet exemple, attribuez le rôle nommé **Contributeur aux données Blob de stockage**, qui comprend un accès en écriture aux objets blob dans un conteneur de Stockage Azure. Pour plus d’informations sur les rôles de conteneur de stockage spécifiques, passez en revue les [Rôles qui peuvent accéder aux objets blob dans un conteneur Stockage Azure](../storage/blobs/authorize-access-azure-active-directory.md#assign-azure-roles-for-access-rights).

1. Ensuite, choisissez l’identité managée dans laquelle vous souhaitez affecter le rôle. Sous **Attribuer l’accès à**, sélectionnez **Identité managée** > **Ajouter des membres**.

1. En fonction du type de votre identité managée, sélectionnez ou fournissez les valeurs suivantes :

   | Type | Instance de service Azure | Abonnement | Membre |
   |------|------------------------|--------------|--------|
   | **Attribué par le système** | **Application logique** | <*Azure-subscription-name*> | <*your-logic-app-name*> |
   | **Affecté par l’utilisateur** | Non applicable | <*Azure-subscription-name*> | <*your-user-assigned-identity-name*> |
   |||||

   Pour plus d’informations sur l’attribution de rôles, consultez la documentation,[Attribution de rôles à l’aide du Portail Azure](../role-based-access-control/role-assignments-portal.md).

1. Une fois que vous avez fini de configurer l’accès pour l’identité, vous pouvez utiliser l’identité pour [authentifier l’accès aux déclencheurs et aux actions qui prennent en charge les identités managées](#authenticate-access-with-identity).

<a name="authenticate-access-with-identity"></a>

## <a name="authenticate-access-with-managed-identity"></a>Authentifier l’accès avec des identités managées

Une fois que vous avez [activé l’identité managée pour votre application logique](#azure-portal-system-logic-app) et [accordé à cette identité l’accès à la ressource ou entité cibles](#access-other-resources), vous pouvez utiliser cette identité dans des [déclencheurs et actions prenant en charge les identités managées](logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

> [!IMPORTANT]
> Si vous avez une fonction Azure dans laquelle vous voulez utiliser l’identité attribuée par le système, commencez par [activer l’authentification pour les fonctions Azure](../logic-apps/logic-apps-azure-functions.md#enable-authentication-for-functions).

Ces étapes montrent comment utiliser l’identité managée avec un déclencheur ou une action via le portail Azure. Pour spécifier l’identité managée dans la définition JSON sous-jacente d’un déclencheur ou d’une action, voir [Authentification d’identité managée](../logic-apps/logic-apps-securing-a-logic-app.md#managed-identity-authentication).

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez votre application logique dans le nouveau concepteur.

1. Si ce n’est encore fait, ajoutez [le déclencheur ou l’action prenant en charge les identités managées](logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

   > [!NOTE]
   > Tous les déclencheurs et toutes les actions ne vous permettent pas d’ajouter un type d’authentification. Pour plus d’informations, consultez [Types d’authentification pour les déclencheurs et les actions qui prennent en charge l’authentification](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

1. Sur le déclencheur ou l’action que vous avez ajouté, procédez comme suit :

   * **Déclencheurs et actions intégrés qui prennent en charge l’utilisation d’une identité managée**

     1. Ajoutez la propriété **Authentification** si la propriété n’apparaît pas déjà.

     1. Sous **Type d’authentification**, sélectionnez **Identité managée**.

     Pour plus d’informations, consultez [Exemple : Authentifier le déclencheur ou l’action intégré avec une identité managée](#authenticate-built-in-managed-identity).
 
   * **Déclencheurs et actions de connecteur géré qui prennent en charge l’utilisation d’une identité managée**

     1. Sur la page de sélection du locataire, sélectionnez **Se connecter avec une identité managée**.

     1. Sur la page suivante, indiquez un nom de connexion.

        Par défaut, la liste d’identités managées affiche uniquement l’identité managée actuellement activée, car une application logique permet d’activer une seule identité managée à la fois, par exemple :

        ![Capture d’écran montrant la page Nom de la connexion et l’identité managée sélectionnée.](./media/create-managed-service-identity/system-assigned-managed-identity.png)

     Pour plus d’informations, consultez [Exemple : Authentifier un déclencheur ou une action de connecteur géré avec une identité managée](#authenticate-managed-connector-managed-identity).

<a name="authenticate-built-in-managed-identity"></a>

#### <a name="example-authenticate-built-in-trigger-or-action-with-a-managed-identity"></a>Exemple : Authentifier le déclencheur ou l’action intégré avec une identité managée

Le déclencheur ou l’action HTTP peuvent utiliser l’identité affectée par le système que vous avez activée pour votre application logique. En général, le déclencheur ou l’action HTTP utilisent ces propriétés pour spécifier la ressource ou l’entité auxquelles vous souhaitez accéder :

| Propriété | Obligatoire | Description |
|----------|----------|-------------|
| **Méthode** | Oui | Méthode HTTP utilisée par l’opération que vous souhaitez exécuter |
| **URI** | Oui | URL de point de terminaison pour accéder à la ressource ou entité Azure cibles. La syntaxe de l’URI comprend généralement l’[ID de ressource](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) pour la ressource ou le service Azure. |
| **En-têtes** | Non | Toute valeur d’en-tête que vous devez ou souhaitez inclure dans la demande sortante, telle que le type de contenu |
| **Requêtes** | Non | Tout paramètre de requête que vous devez ou souhaitez inclure dans la demande, tel que le paramètre pour une opération spécifique ou la version de l’API pour l’opération que vous souhaitez exécuter |
| **Authentification** | Oui | Type d’authentification à utiliser pour authentifier l’accès à la ressource ou entité cibles |
||||

À titre d’exemple, supposons que vous souhaitez exécuter l’[opération de capture instantanée d’objet blob](/rest/api/storageservices/snapshot-blob) sur un blob dans le compte de Stockage Azure où vous avez précédemment configuré l’accès pour votre identité. Toutefois, le [connecteur de Stockage Blob Azure](/connectors/azureblob/) ne propose pas cette opération actuellement. Au lieu de cela, vous pouvez l’exécuter à l’aide de l’[action HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action) ou d’une autre [opération de l’API REST du service BLOB](/rest/api/storageservices/operations-on-blobs).

> [!IMPORTANT]
> Pour accéder aux comptes de stockage Azure derrière des pare-feu en utilisant des requêtes HTTP et des identités managées, veillez à configurer également votre compte de stockage avec l’[exception qui autorise l’accès de services Microsoft approuvés](../connectors/connectors-create-api-azureblobstorage.md#access-blob-storage-with-managed-identities).

Pour exécuter l’[opération de capture instantanée d’objet blob](/rest/api/storageservices/snapshot-blob), l’action HTTP spécifie les propriétés suivantes :

| Propriété | Obligatoire | Valeur d'exemple | Description |
|----------|----------|---------------|-------------|
| **Méthode** | Oui | `PUT`| Méthode HTTP utilisée par l’opération de capture instantanée d’objet blob |
| **URI** | Oui | `https://{storage-account-name}.blob.core.windows.net/{blob-container-name}/{folder-name-if-any}/{blob-file-name-with-extension}` | ID de ressource d’un fichier de Stockage Blob Azure dans l’environnement global (public) Azure qui utilise cette syntaxe |
| **En-têtes** | Pour Stockage Azure | `x-ms-blob-type` = `BlockBlob` <p>`x-ms-version` = `2019-02-02` <p>`x-ms-date` = `@{formatDateTime(utcNow(),'r')}` | Les valeurs d’en-tête `x-ms-blob-type`, `x-ms-version` et `x-ms-date` sont requises pour les opérations du service Stockage Azure. <p><p>**Important !** Dans le déclencheur HTTP sortant et les demandes d’action pour Stockage Azure, l’en-tête requiert la propriété `x-ms-version` et la version de l’API pour l’opération que vous souhaitez exécuter. La valeur `x-ms-date` doit être la date actuelle. Dans le cas contraire, votre application logique échoue avec une erreur `403 FORBIDDEN`. Pour obtenir la date actuelle au format requis, vous pouvez utiliser l’expression dans l’exemple de valeur. <p>Pour plus d’informations, consultez les rubriques suivantes : <p><p>- [En-têtes de demande – Capture instantanée d’objet blob](/rest/api/storageservices/snapshot-blob#request) <br>- [Contrôle de version pour les services Stockage Azure](/rest/api/storageservices/versioning-for-the-azure-storage-services#specifying-service-versions-in-requests) |
| **Requêtes** | Uniquement pour l’opération de capture instantanée de blob | `comp` = `snapshot` | Nom et valeur du paramètre de requête pour l’opération. |
|||||

Voici l’exemple d’action HTTP qui affiche toutes ces valeurs de propriété :

![Ajouter une action HTTP pour accéder à une ressource Azure](./media/create-managed-service-identity/http-action-example.png)

1. Après avoir ajouté l’action HTTP, ajoutez la propriété **Authentification** à l’action HTTP. Dans la liste **Ajouter un nouveau paramètre**, sélectionnez **Authentification**.

   ![Ajouter la propriété « Authentification » à l’action HTTP](./media/create-managed-service-identity/add-authentication-property.png)

   > [!NOTE]
   > Tous les déclencheurs et toutes les actions ne vous permettent pas d’ajouter un type d’authentification. Pour plus d’informations, consultez [Types d’authentification pour les déclencheurs et les actions qui prennent en charge l’authentification](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

1. Dans la liste **Type d’authentification**, sélectionnez **Identité managée**.

   ![Pour « Authentification », sélectionner « Identité managée »](./media/create-managed-service-identity/select-managed-identity.png)

1. Dans la liste des identités managées, sélectionnez les options disponibles en fonction de votre scénario.

   * Si vous configurez l’identité affectée par le système, sélectionnez **Identité managée affectée par le système**, si cela n’est pas déjà fait.

     ![Sélectionner « Identité managée affectée par le système »](./media/create-managed-service-identity/select-system-assigned-identity-for-action.png)

   * Si vous configurez une identité affectée par l’utilisateur, sélectionnez cette identité, si cela n’est pas déjà fait.

     ![Sélectionner l’identité affectée par l’utilisateur](./media/create-managed-service-identity/select-user-assigned-identity-for-action.png)

   Cet exemple se poursuit avec **Identité managée affectée par le système**.

1. Sur certains déclencheurs et actions, la propriété **Audience** apparaît également pour vous permettre de définir l’ID de ressource cible. Affectez à la propriété **Audience** la valeur de l’[ID de ressource de la ressource ou du service cible](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication). Autrement, par défaut, la propriété **Audience** utilise l’ID de ressource `https://management.azure.com/`, qui est l’ID de ressource pour Azure Resource Manager.
  
    Par exemple, si vous souhaitez authentifier l’accès à une [ressource de Coffre de clés dans le cloud global Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-key-vault), vous devez définir la propriété **Audience** *précisément* sur l’ID de ressource suivante : `https://vault.azure.net`. Veuillez noter que cet ID de ressource spécifique ne possède *pas* de barres obliques de fin. En réalité, l’inclusion d’une barre oblique de fin pourrait produire soit une erreur `400 Bad Request` soit une erreur `401 Unauthorized`.

   > [!IMPORTANT]
   > Vérifiez que l’ID de ressource cible *correspond exactement* à la valeur qu’attend Azure Active Directory, y compris les barres obliques de fin obligatoires. Par exemple, l’ID de ressource pour tous les comptes de Stockage Blob Azure requiert une barre oblique finale. Toutefois, l’ID de ressource pour un compte de stockage spécifique ne requiert pas de barre oblique finale. Vérifiez les [ID de ressource des services Azure qui prennent en charge Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

   Cet exemple définit la propriété **Audience** sur `https://storage.azure.com/` afin que les jetons d’accès utilisés pour l’authentification soient valides pour tous les comptes de stockage. Toutefois, vous pouvez également spécifier l’URL du service racine, `https://fabrikamstorageaccount.blob.core.windows.net`, pour un compte de stockage spécifique.

   ![Affecter à la propriété « Audience » l’ID de ressource cible](./media/create-managed-service-identity/specify-audience-url-target-resource.png)

   Pour plus d’informations sur l’autorisation de l’accès avec Azure AD pour Azure Storage, consultez les rubriques suivantes :

   * [Autoriser l’accès aux objets blob et aux files d’attente Azure avec Azure Active Directory](../storage/blobs/authorize-access-azure-active-directory.md)
   * [Autoriser l’accès au Stockage Azure avec Azure Active Directory](/rest/api/storageservices/authorize-with-azure-active-directory#use-oauth-access-tokens-for-authentication)

1. Continuez à créer l’application logique comme vous le souhaitez.

<a name="authenticate-managed-connector-managed-identity"></a>

#### <a name="example-authenticate-managed-connector-trigger-or-action-with-a-managed-identity"></a>Exemple : Authentifier un déclencheur ou une action de connecteur géré avec une identité managée

L’action Azure Resource Manager, **Lire une ressource**, peut utiliser l’identité managée que vous avez activée pour votre application logique. Cet exemple montre comment utiliser l’identité managée affectée par le système.

1. Une fois que vous avez ajouté l’action à votre workflow, dans la page de sélection du locataire, sélectionnez **Se connecter avec une identité managée**.

   ![Capture d’écran qui montre l’action Azure Resource Manager et l’option « Se connecter avec une identité managée » sélectionnée.](./media/create-managed-service-identity/select-connect-managed-identity.png)

   L’action affiche maintenant la page Nom de la connexion avec la liste des identités managées, qui comprend le type d’identité managée qui est actuellement activé sur l’application logique.

1. Dans la page Nom de la connexion, indiquez un nom pour la connexion. Dans la liste d’identités managées, sélectionnez l’identité managée (**Identité managée affectée par le système** dans cet exemple), puis sélectionnez **Créer**. Si vous avez activé une identité managée affectée par l’utilisateur, sélectionnez cette identité à la place.

   ![Capture d’écran qui montre l’action Azure Resource Manager avec le nom de connexion entré et l’option « Identité managée affectée par le système » sélectionnée.](./media/create-managed-service-identity/system-assigned-managed-identity.png)

   Si l’identité managée n’est pas activée, l’erreur suivante s’affiche lorsque vous essayez de créer la connexion :

   *Vous devez activer l’identité managée pour votre application logique, puis accorder l’accès requis à l’identité dans la ressource cible.*

   ![Capture d’écran qui montre l’action Azure Resource Manager avec une erreur quand aucune identité managée n’est activée.](./media/create-managed-service-identity/system-assigned-managed-identity-disabled.png)

1. Une fois la connexion créée, le concepteur peut extraire les valeurs, le contenu ou le schéma dynamiques à l’aide de l’authentification par identité managée.

1. Continuez à créer l’application logique comme vous le souhaitez.

<a name="logic-app-resource-definition-connection-managed-identity"></a>

### <a name="logic-app-resource-definition-and-connections-that-use-a-managed-identity"></a>Définition et connexions de ressources d’application logique qui utilisent une identité managée

Une connexion qui active et utilise une identité managée est un type particulier de connexion qui fonctionne uniquement avec une identité managée. Au moment de l’exécution, la connexion utilise l’identité managée qui est activée sur l’application logique. Cette configuration est enregistrée dans l’objet `parameters` de la définition de ressource de l’application logique, qui contient l’objet `$connections` qui inclut des pointeurs vers l’ID de ressource de la connexion ainsi que l’ID de ressource de l’identité, si l’identité affectée par l’utilisateur est activée.

Cet exemple montre à quoi ressemble la configuration lorsque l’application logique active l’identité managée affectée par le système :

```json
"parameters": {
   "$connections": {
      "value": {
         "<action-name>": {
            "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/connections/{connection-name}",
            "connectionName": "{connection-name}",
            "connectionProperties": {
               "authentication": {
                  "type": "ManagedServiceIdentity"
               }
            },
            "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{managed-connector-type}"
         }
      }
   }
}
```

Cet exemple montre à quoi ressemble la configuration lorsque l’application logique active l’identité managée affectée par l’utilisateur :

```json
"parameters": {
   "$connections": {
      "value": {
         "<action-name>": {
            "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/connections/{connection-name}",
            "connectionName": "{connection-name}",
            "connectionProperties": {
               "authentication": {
                  "identity": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/microsoft.managedidentity/userassignedidentities/{managed-identity-name}",
                  "type": "ManagedServiceIdentity"
               }
            },
            "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{managed-connector-type}"
         }
      }
   }
}
```

Pendant l’exécution, le service Logic Apps vérifie si un déclencheur et des actions de connecteur géré dans l’application logique sont configurés pour utiliser l’identité managée et que toutes les autorisations requises sont configurées pour utiliser l’identité managée pour accéder aux ressources cibles qui sont spécifiées par le déclencheur et les actions. En cas de réussite, le service Logic Apps récupère le jeton Azure AD associé à l’identité managée et utilise cette identité pour authentifier l’accès à la ressource cible et effectuer l’opération configurée dans le déclencheur et les actions.

<a name="arm-templates-connection-resource-managed-identity"></a>

## <a name="arm-template-for-managed-connections-and-managed-identities"></a>Modèle ARM pour les connexions managées et les identités managées

Si vous automatisez le déploiement avec un modèle ARM et si votre application logique inclut un déclencheur ou une action de connecteur managé qui utilise une identité managée, vérifiez que la définition de ressource de la connexion sous-jacente inclut la propriété `parameterValueType` avec `Alternative` comme valeur de propriété. Sinon, votre déploiement ARM ne pourra pas configurer la connexion pour utiliser l’identité managée à des fins d’authentification, et la connexion ne fonctionnera pas dans le workflow de votre application logique. Cet impératif s’applique uniquement aux [déclencheurs et actions de connecteur managé spécifiques](#managed-connectors-managed-identity) pour lesquels vous avez sélectionné l’[option **Se connecter avec une identité managée**](#authenticate-managed-connector-managed-identity).

Par exemple, voici la définition de ressource de connexion sous-jacente pour une action Azure Automation qui utilise une identité managée où la définition inclut la propriété `parameterValueType`, dont la valeur de propriété est `Alternative` :

```json
{
    "type": "Microsoft.Web/connections",
    "name": "[variables('automationAccountApiConnectionName')]",
    "apiVersion": "2016-06-01",
    "location": "[parameters('location')]",
    "kind": "V1",
    "properties": {
        "api": {
            "id": "[subscriptionResourceId('Microsoft.Web/locations/managedApis', parameters('location'), 'azureautomation')]"
        },
        "customParameterValues": {},
        "displayName": "[variables('automationAccountApiConnectionName')]",
        "parameterValueType": "Alternative"
    }
},
```

<a name="remove-identity"></a>

## <a name="disable-managed-identity"></a>Désactiver l’identité managée

Pour arrêter d’utiliser une identité managée pour votre application logique, vous disposez des options suivantes :

* [Azure portal](#azure-portal-disable)
* [Modèle ARM](#template-disable)
* Azure PowerShell
  * [Supprimer une attribution de rôle](../role-based-access-control/role-assignments-powershell.md)
  * [Supprimer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
* Azure CLI
  * [Supprimer une attribution de rôle](../role-based-access-control/role-assignments-cli.md)
  * [Supprimer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
* API REST Azure
  * [Supprimer une attribution de rôle](../role-based-access-control/role-assignments-rest.md)
  * [Supprimer une identité affectée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)

Si vous supprimez votre application logique, Azure supprime automatiquement d’Azure AD l’identité managée.

<a name="azure-portal-disable"></a>

### <a name="disable-managed-identity-in-the-azure-portal"></a>Désactiver l’identité managée dans le portail Azure

Dans le portail Azure, supprimez d’abord l’accès de l’identité à [votre ressource cible](#disable-identity-target-resource). Désactivez ensuite l’identité affectée par le système, ou supprimez l’identité affectée par l’utilisateur de [votre application logique](#disable-identity-logic-app).

<a name="disable-identity-target-resource"></a>

#### <a name="remove-identity-access-from-resources"></a>Supprimer l’accès d’une identité à des ressources

1. Dans le [portail Azure](https://portal.azure.com), accédez à la ressource Azure cible dont vous souhaitez supprimer l’accès à l’identité managée.

1. Dans le menu de la ressource cible, sélectionnez **Contrôle d’accès (IAM)** . Sous la barre d’outils, sélectionnez **Attributions de rôle**.

1. Dans la liste des rôles, sélectionnez les identités managées que vous souhaitez supprimer. Dans la barre d’outils, sélectionnez **Supprimer**.

   > [!TIP]
   > Si l’option **Supprimer** est désactivée, vous n’avez probablement pas les autorisations appropriées. Pour plus d’informations sur les autorisations permettant de gérer des rôles pour des ressources, voir [Autorisations des rôles d’administrateur dans Azure Active Directory](../active-directory/roles/permissions-reference.md).

L’identité managée est maintenant supprimée et n’a plus accès à la ressource cible.

<a name="disable-identity-logic-app"></a>

#### <a name="disable-managed-identity-on-logic-app"></a>Désactiver l’identité managée sur l’application logique

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez votre application logique dans le nouveau concepteur.

1. Dans le menu de l’application logique, sous **Paramètres**, sélectionnez **Identité**, puis suivez les étapes correspondant à votre identité :

   * Sélectionnez **Affecté(e) par le système** > **Actif** > **Enregistrer**. Quand Azure vous invite à confirmer l’opération, sélectionnez **Oui**.

     ![Désactiver l’identité affectée par le système](./media/create-managed-service-identity/disable-system-assigned-identity.png)

   * Sélectionnez **Affecté(e) par l’utilisateur** et l’identité managée, puis sélectionnez **Supprimer**. Quand Azure vous invite à confirmer l’opération, sélectionnez **Oui**.

     ![Supprimer l’identité affectée par l’utilisateur](./media/create-managed-service-identity/remove-user-assigned-identity.png)

L’identité managée est désormais désactivée sur votre application logique.

<a name="template-disable"></a>

### <a name="disable-managed-identity-in-an-arm-template"></a>Désactiver l’identité managée dans un modèle ARM

Si vous avez créé l’identité managée de l’application logique à l’aide d’un modèle ARM, définissez la propriété enfant de l’`identity`objet `type` sur `None`.

```json
"identity": {
   "type": "None"
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser l’accès et les données dans Azure Logic Apps](../logic-apps/logic-apps-securing-a-logic-app.md)