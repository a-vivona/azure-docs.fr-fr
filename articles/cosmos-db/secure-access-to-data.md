---
title: Sécuriser l’accès aux données Azure Cosmos DB
description: Découvrez les concepts liés au contrôle d’accès dans Azure Cosmos DB, notamment les clés primaires, les clés en lecture seule, les utilisateurs et les autorisations.
author: thomasweiss
ms.author: thweiss
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/30/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: b0fcfba6f72ef5e87be5c3301338a2c09598accf
ms.sourcegitcommit: 2eac9bd319fb8b3a1080518c73ee337123286fa2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123253425"
---
# <a name="secure-access-to-data-in-azure-cosmos-db"></a>Sécuriser l’accès aux données dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article fournit une vue d’ensemble du contrôle d’accès aux données dans Azure Cosmos DB.

Azure Cosmos DB offre trois moyens de contrôler l’accès à vos données.

| Type de contrôle d’accès | Caractéristiques |
|---|---|
| [Clés principales/secondaires](#primary-keys) | Secret partagé autorisant toutes les opérations de gestion ou de données. Il offre des variantes en lecture-écriture et en lecture seule. |
| [Contrôle d’accès basé sur les rôles](#rbac) | Modèle d’autorisation précis basé sur les rôles, utilisant les identités Azure Active Directory (AAD) pour l’authentification. |
| [Jetons de ressource](#resource-tokens)| Modèle d’autorisation précis basé sur les autorisations et les utilisateurs natifs d’Azure Cosmos DB. |

## <a name="primarysecondary-keys"></a><a id="primary-keys"></a>Clés principales/secondaires

Les clés principales/secondaires donnent accès à toutes les ressources administratives du compte de base de données. Chaque compte comporte deux clés : une clé primaire et une clé secondaire. L’objectif de ces paires de clés est de vous permettre de regénérer ou de permuter les clés, ce qui offre un accès permanent à votre compte et à vos données. Pour en savoir plus sur les clés primaires/secondaires, consultez l’article [Sécurité des bases de données](database-security.md#primary-keys).

### <a name="key-rotation-and-regeneration"></a><a id="key-rotation"></a> Rotation et régénération des clés

> [!NOTE]
> Suivez les instructions décrites [ici](database-security.md#key-rotation) pour faire pivoter et régénérer les clés de l’API Azure Cosmos DB pour Mongo DB, l’API Cassandra, l’API Gremlin ou l’API Table.

Le processus de rotation et de régénération des clés est simple. Tout d’abord, assurez-vous que **votre application utilise régulièrement la clé primaire ou la clé secondaire** pour accéder à votre compte Azure Cosmos DB. Puis, suivez les étapes présentées ci-dessous.

# <a name="if-your-application-is-currently-using-the-primary-key"></a>[Si votre application utilise actuellement la clé primaire](#tab/using-primary-key)

1. Dans le portail Azure, accédez à votre compte Azure Cosmos DB.

1. Sélectionnez **Clés** dans le menu gauche, puis sélectionnez **Régénérer la clé secondaire** à partir de l’ellipse sur la droite de votre clé secondaire.

    :::image type="content" source="./media/database-security/regenerate-secondary-key.png" alt-text="Capture d’écran du portail Azure montrant comment régénérer la clé secondaire" border="true":::

1. Vérifiez que la nouvelle clé secondaire fonctionne de façon cohérente par rapport à votre compte Azure Cosmos DB. Le régénération des clés peut prendre de moins d’une minute à plusieurs heures selon la taille du compte Cosmos DB.

1. Remplacez votre clé primaire par votre clé secondaire dans votre application.

1. Revenez au portail Azure et déclenchez la régénération de la clé primaire.

    :::image type="content" source="./media/database-security/regenerate-primary-key.png" alt-text="Capture d’écran du portail Azure montrant comment régénérer la clé primaire" border="true":::

# <a name="if-your-application-is-currently-using-the-secondary-key"></a>[Si votre application utilise actuellement la clé secondaire](#tab/using-secondary-key)

1. Dans le portail Azure, accédez à votre compte Azure Cosmos DB.

1. Sélectionnez **Clés** dans le menu gauche, puis sélectionnez **Régénérer la clé primaire** à partir de l’ellipse sur la droite de votre clé primaire.

    :::image type="content" source="./media/database-security/regenerate-primary-key.png" alt-text="Capture d’écran du portail Azure montrant comment régénérer la clé primaire" border="true":::

1. Vérifiez que la nouvelle clé primaire fonctionne de façon cohérente par rapport à votre compte Azure Cosmos DB. Le régénération des clés peut prendre de moins d’une minute à plusieurs heures selon la taille du compte Cosmos DB.

1. Remplacez votre clé secondaire par votre clé primaire dans votre application.

1. Revenez au portail Azure et déclenchez la régénération de la clé secondaire.

    :::image type="content" source="./media/database-security/regenerate-secondary-key.png" alt-text="Capture d’écran du portail Azure montrant comment régénérer la clé secondaire" border="true":::

---

### <a name="code-sample-to-use-a-primary-key"></a>Exemple de code utilisant une clé primaire

L’exemple de code suivant montre comment utiliser une clé primaire et un point de terminaison de compte Azure Cosmos DB pour instancier un CosmosClient :

```csharp
// Read the Azure Cosmos DB endpointUrl and authorization keys from config.
// These values are available from the Azure portal on the Azure Cosmos DB account blade under "Keys".
// Keep these values in a safe and secure location. Together they provide Administrative access to your Azure Cosmos DB account.

private static readonly string endpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
private static readonly string authorizationKey = ConfigurationManager.AppSettings["AuthorizationKey"];

CosmosClient client = new CosmosClient(endpointUrl, authorizationKey);
```

## <a name="role-based-access-control"></a><a id="rbac"></a> Contrôle d’accès en fonction du rôle

Azure Cosmos DB expose un système de contrôle d’accès en fonction du rôle (RBAC) intégré, qui vous permet :

- D’authentifier vos demandes de données avec une identité Azure Active Directory (AAD).
- D’autoriser vos demandes de données avec un modèle d’autorisation précis basé sur les rôles.

Le contrôle RBAC d’Azure Cosmos DB est la méthode de contrôle d’accès idéale dans les situations où :

- Vous ne voulez pas utiliser un secret partagé comme la clé principale et vous préférez vous appuyer sur un mécanisme d’authentification basé sur des jetons.
- Vous voulez utiliser des identités Azure AD pour authentifier vos demandes.
- Vous avez besoin d’un modèle d’autorisation précis pour limiter rigoureusement les opérations de base de données que vos identités sont autorisées à effectuer.
- Vous voulez matérialiser vos stratégies de contrôle d’accès en tant que « rôles » que vous pouvez affecter à plusieurs identités.

Pour plus d’informations sur le contrôle RBAC d’Azure Cosmos DB, consultez [Configurer le contrôle d’accès en fonction du rôle pour votre compte Azure Cosmos DB](how-to-setup-rbac.md).

## <a name="resource-tokens"></a><a id="resource-tokens"></a> Jetons de ressource

Les jetons de ressource fournissent un accès aux ressources d’application au sein d’une base de données. Jetons de ressource :

- Fournissent un accès à des conteneurs, clés de partition, documents, pièces jointes, procédures stockées, déclencheurs et fonctions définies par l’utilisateur spécifiques.
- Sont créés lorsqu’un [utilisateur](#users) dispose des [autorisations](#permissions) sur une ressource spécifique.
- Sont recréés lorsqu’une ressource d’autorisation est exécutée par un appel POST, GET ou PUT.
- Utilisent un jeton de ressource de hachage créé spécifiquement pour l’utilisateur, la ressource et les autorisations.
- Sont liés à une période de validité personnalisable. La durée de validité par défaut est d’une heure. La durée de vie du jeton peut cependant être définie de manière explicite (cinq heures maximum).
- Évitent d’avoir à divulguer la clé primaire, pour plus de sécurité.
- Permettent aux clients de lire, d’écrire et de supprimer des ressources dans le compte Azure Cosmos DB en fonction des autorisations qui leur ont été accordées.

Vous pouvez utiliser un jeton de ressource (en créant des utilisateurs et des autorisations Cosmos DB) lorsque vous voulez donner accès aux ressources dans votre compte Cosmos DB à un client qui ne peut pas être approuvé avec la clé primaire.  

Les jetons de ressource Cosmos DB permettent aux clients de lire, d’écrire et de supprimer des ressources dans votre compte Cosmos DB de manière sécurisée, en fonction des autorisations que vous avez accordées et sans clé primaire ni clé en lecture seule.

Voici un modèle de conception standard dans le cadre duquel des jetons de ressource peuvent être demandés, générés et fournis aux clients :

1. Un service de niveau intermédiaire est configuré pour servir une application mobile pour partager les photos de l'utilisateur.
2. Le service de niveau intermédiaire détient la clé primaire du compte Cosmos DB.
3. L’application photo est installée sur les appareils mobiles des utilisateurs finaux.
4. Lors de la connexion, l'application photo établit l'identité de l'utilisateur avec le service de niveau intermédiaire. Ce mécanisme d'identification dépend totalement de l'application.
5. Une fois l'identité établie, le service de niveau intermédiaire demande des autorisations en fonction de l'identité.
6. Le service de niveau intermédiaire renvoie un jeton de ressource à l'application du téléphone.
7. Cette dernière peut continuer à utiliser le jeton de ressource pour accéder directement aux ressources Azure Cosmos DB avec les autorisations définies et pendant l’intervalle autorisé.
8. À expiration du jeton de ressource, les demandes suivantes reçoivent une exception non autorisée 401.  L'application du téléphone établit alors de nouveau l'identité de l'utilisateur et demande un nouveau jeton de ressource.

    :::image type="content" source="./media/secure-access-to-data/resourcekeyworkflow.png" alt-text="Workflow de jetons de ressource Azure Cosmos DB" border="false":::

La gestion et la génération des jetons de ressource sont prises en charge par les bibliothèques clientes natives Azure Cosmos DB. Toutefois, si vous utilisez REST, vous devez créer les en-têtes de demande/d’authentification. Pour plus d’informations sur la création d’en-têtes d’authentification pour REST, consultez [Contrôle d’accès aux ressources Azure Cosmos DB](/rest/api/cosmos-db/access-control-on-cosmosdb-resources) ou le code source de notre [Kit de développement logiciel (SDK) .NET](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos/src/Authorization/AuthorizationHelper.cs) ou [Kit de développement logiciel (SDK) Node.js](https://github.com/Azure/azure-cosmos-js/blob/master/src/auth.ts).

Pour obtenir un exemple de service de niveau intermédiaire utilisé pour générer ou répartir les jetons de ressource, consultez [l’application ResourceTokenBroker](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/xamarin/UserItems/ResourceTokenBroker/ResourceTokenBroker/Controllers).

### <a name="users"></a>Utilisateurs<a id="users"></a>

Les utilisateurs d’Azure Cosmos DB sont associés à une base de données Cosmos.  Chaque base de données peut contenir zéro, un ou plusieurs utilisateurs Azure Cosmos DB. L’exemple de code suivant indique comment créer un utilisateur Cosmos DB à l’aide du [Kit de développement logiciel (SDK) Azure Cosmos DB .NET v3](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement).

```csharp
//Create a user.
Database database = benchmark.client.GetDatabase("SalesDatabase");

User user = await database.CreateUserAsync("User 1");
```

> [!NOTE]
> Chaque utilisateur Cosmos DB dispose d’une méthode ReadAsync() qui permet de récupérer la liste des [autorisations](#permissions) qui lui sont associées.

### <a name="permissions"></a>Autorisations<a id="permissions"></a>

Une ressource d’autorisation est associée à un utilisateur et affectée à une ressource spécifique. Chaque utilisateur peut contenir zéro, une ou plusieurs autorisations. Une ressource d’autorisation donne accès à un jeton de sécurité dont l’utilisateur a besoin lorsqu’il tente d’accéder à un conteneur ou à des données spécifiques dans une clé de partition spécifique. Il existe deux niveaux d’accès disponibles qui peuvent être fournis par une ressource d’autorisation :

- Tout : l’utilisateur dispose de toutes les autorisations sur la ressource.
- Lecture : L’utilisateur peut uniquement lire le contenu de la ressource, il ne peut pas procéder à des opérations d’écriture, de mise à jour ou de suppression au niveau de la ressource.

> [!NOTE]
> Pour exécuter des procédures stockées, l’utilisateur doit disposer de toutes les autorisations sur le conteneur dans lequel la procédure stockée est exécutée.

Si les [journaux de diagnostic sur les demandes de plan de données](cosmosdb-monitor-resource-logs.md) sont activés, les deux propriétés suivantes correspondant à l’autorisation sont journalisées :

* **resourceTokenPermissionId** : cette propriété indique l’ID d’autorisation du jeton de ressource que vous avez spécifié. 

* **resourceTokenPermissionMode** : cette propriété indique le mode d’autorisation que vous avez défini lors de la création du jeton de ressource. Elle peut avoir différentes valeurs, notamment « all » et « read ».

#### <a name="code-sample-to-create-permission"></a>Exemple de code pour créer une autorisation

L’exemple de code suivant indique comment créer une ressource d’autorisation, lire le jeton de ressource de la ressource d’autorisation et associer les autorisations à [l’utilisateur](#users) créé ci-dessus.

```csharp
// Create a permission on a container and specific partition key value
Container container = client.GetContainer("SalesDatabase", "OrdersContainer");
user.CreatePermissionAsync(
    new PermissionProperties(
        id: "permissionUser1Orders",
        permissionMode: PermissionMode.All,
        container: container,
        resourcePartitionKey: new PartitionKey("012345")));
```

#### <a name="code-sample-to-read-permission-for-user"></a>Exemple de code pour l’autorisation de lecture de l’utilisateur

L’extrait de code suivant indique comment récupérer l’autorisation associée à l’utilisateur créé ci-dessus et instancier un nouveau CosmosClient au nom de l’utilisateur, limité à une clé de partition unique.

```csharp
//Read a permission, create user client session.
PermissionProperties permissionProperties = await user.GetPermission("permissionUser1Orders")

CosmosClient client = new CosmosClient(accountEndpoint: "MyEndpoint", authKeyOrResourceToken: permissionProperties.Token);
```

## <a name="differences-between-rbac-and-resource-tokens"></a>Différences entre le contrôle RBAC et les jetons de ressource

| Objet | RBAC | Jetons de ressource |
|--|--|--|
| Authentification  | Avec Azure Active Directory (Azure AD). | Basé sur les utilisateurs Azure Cosmos DB natifs<br>L’intégration des jetons de ressource à Azure AD demande plus de travail pour établir un pont entre les identités Azure AD et les utilisateurs Azure Cosmos DB. |
| Autorisation | Basée sur les rôles : les définitions de rôle mappent les actions autorisées et peuvent être affectées à plusieurs identités. | Basée sur les autorisations : pour chaque utilisateur Azure Cosmos DB, vous devez affecter des autorisations d’accès aux données. |
| Étendue des jetons | Un jeton AAD contient l’identité du demandeur. Cette identité est comparée à toutes les définitions de rôles affectées pour accorder l’autorisation. | Un jeton de ressource contient l’autorisation accordée à un utilisateur Azure Cosmos DB spécifique sur une ressource Azure Cosmos DB spécifique. Les demandes d’autorisation sur différentes ressources peuvent nécessiter différents jetons. |
| Actualisation des jetons | Le jeton AAD est actualisé automatiquement par les kits SDK Azure Cosmos DB quand il expire. | L’actualisation du jeton de ressource n’est pas prise en charge. Quand un jeton de ressource expire, un nouveau jeton doit être émis. |

## <a name="add-users-and-assign-roles"></a>Ajouter des utilisateurs et attribuer des rôles

Pour ajouter l’accès en lecture aux comptes Azure Cosmos DB à votre compte d’utilisateur, demandez à un propriétaire d’abonnement d’effectuer les étapes suivantes dans le portail Azure.

1. Ouvrez le portail Azure, puis sélectionnez votre compte Azure Cosmos DB.
2. Cliquez sur l’onglet **Contrôle d’accès (IAM)** , puis cliquez sur **+ Ajouter une attribution de rôle**.
3. Dans le volet **Ajouter une attribution de rôle**, dans la zone **Rôle**, sélectionnez **Rôle de lecteur de compte Cosmos DB**.
4. Dans la zone **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou application Azure AD**.
5. Dans votre annuaire, sélectionnez l’utilisateur, le groupe ou l’application qui doit recevoir l’accès.  Dans l’annuaire, vous pouvez rechercher des noms d’affichage, des adresses e-mail et des identificateurs d’objet.
    L’utilisateur, le groupe ou l’application s’affiche alors dans la liste des membres sélectionnés.
6. Cliquez sur **Enregistrer**.

L’entité peut désormais lire les ressources Azure Cosmos DB.

## <a name="delete-or-export-user-data"></a>Supprimer ou exporter des données utilisateur

Le service de base de données Azure Cosmos DB vous permet de rechercher, de sélectionner, de modifier et de supprimer des données situées dans votre base de données ou vos conteneurs. Toutefois, il est de votre responsabilité d’utiliser les API fournies et de définir la logique requise pour rechercher et effacer les données personnelles si nécessaire. Chaque API multimodèle (SQL, MongoDB, Gremlin, Cassandra ou encore Table) fournit différents kits SDK de langage qui contiennent des méthodes permettant de rechercher et de supprimer des données en fonction de prédicats personnalisés. Vous pouvez également activer la fonctionnalité de [durée de vie (TTL)](time-to-live.md) pour supprimer automatiquement les données après une période spécifiée, sans frais supplémentaires.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la sécurité des bases de données Cosmos, consultez [Sécurité des bases de données Azure Cosmos DB](database-security.md).
- Pour savoir comment créer des jetons d’autorisation Azure Cosmos DB, consultez [Access Control on Azure Cosmos DB Resources](/rest/api/cosmos-db/access-control-on-cosmosdb-resources) (Contrôle d’accès aux ressources Azure Cosmos DB).
- Exemples de gestion des utilisateurs avec des utilisateurs et des autorisations, [Exemples de gestion des utilisateurs à l’aide du Kit de développement logiciel (SDK) .NET v3](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement/UserManagementProgram.cs)
