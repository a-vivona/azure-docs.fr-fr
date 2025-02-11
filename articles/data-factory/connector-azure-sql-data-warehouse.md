---
title: Copier et transformer des données dans Azure Synapse Analytics
titleSuffix: Azure Data Factory & Azure Synapse
description: Découvrez comment copier des données depuis et vers Azure Synapse Analytics et comment transformer des données dans Azure Synapse Analytics à l’aide de Data Factory.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.subservice: data-movement
ms.custom: synapse
ms.topic: conceptual
ms.date: 08/30/2021
ms.openlocfilehash: 2131e74935ee831925dbe307a79c26909078e575
ms.sourcegitcommit: 851b75d0936bc7c2f8ada72834cb2d15779aeb69
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123313977"
---
# <a name="copy-and-transform-data-in-azure-synapse-analytics-by-using-azure-data-factory-or-synapse-pipelines"></a>Copier et transformer des données dans Azure Synapse Analytics à l’aide de pipelines Azure Data Factory ou Synapse

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
>
> - [Version1](v1/data-factory-azure-sql-data-warehouse-connector.md)
> - [Version actuelle](connector-azure-sql-data-warehouse.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment utiliser l’activité de copie dans un pipeline Azure Data Factory ou Synapse pour copier des données depuis et vers Azure Synapse Analytics et utiliser Data Flow pour transformer les données dans Azure Data Lake Storage Gen2. Pour en savoir plus sur Azure Data Factory, lisez l’[article d’introduction](introduction.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Azure Synapse Analytics est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec tableau de [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Mappage de flux de données](concepts-data-flow-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)

Pour l’activité de copie, ce connecteur Azure Synapse Analytics prend en charge les fonctions suivantes :

- Copie de données à l’aide de l’authentification SQL et de l’authentification du jeton de l’application Azure Active Directory (Azure AD) avec un principal de service ou l’identité managée pour les ressources Azure.
- En tant que source, récupération de données à l’aide d’une requête SQL ou d’une procédure stockée. Vous pouvez également choisir de copier en parallèle à partir de la source Azure Synapse Analytics. Pour plus d’informations, consultez la section [Copier en parallèle à partir d’Azure Synapse Analytics](#parallel-copy-from-azure-synapse-analytics).
- En tant que récepteur, chargez les données avec l’[instruction COPY](#use-copy-statement), [PolyBase](#use-polybase-to-load-data-into-azure-synapse-analytics) ou bulk insert. Nous vous recommandons d’utiliser PolyBase ou l’instruction COPY pour de meilleures performances de copie. Le connecteur prend également en charge la création automatique de la table de destination avec DISTRIBUTION = ROUND_ROBIN, si celle-ci n’existe pas, en fonction du schéma source.

> [!IMPORTANT]
> Si vous copiez des données à l’aide d’Azure Integration Runtime, configurez une [règle de pare-feu au niveau du serveur](../azure-sql/database/firewall-configure.md) de façon à ce que les services Azure puissent accéder au [serveur SQL logique](../azure-sql/database/logical-servers.md).
> Si vous copiez des données à l’aide d’un runtime d’intégration auto-hébergé, configurez le pare-feu pour qu’il autorise la plage IP appropriée. Cette plage inclut l’adresse IP de l’ordinateur qui est utilisé pour se connecter à Azure Synapse Analytics.
## <a name="get-started"></a>Bien démarrer

> [!TIP]
> Pour obtenir le meilleur niveau de performance possible, utilisez PolyBase ou l’instruction COPY quand vous chargez des données dans Azure Synapse Analytics. Pour plus d’informations, consultez les sections [Chargement de données dans Azure Synapse Analytics avec PolyBase](#use-polybase-to-load-data-into-azure-synapse-analytics) et [Chargement de données dans Azure Synapse Analytics avec l’instruction COPY](#use-copy-statement). Consultez [Charger 1 To dans Azure Synapse Analytics en moins de 15 minutes avec Azure Data Factory](load-azure-sql-data-warehouse.md) pour obtenir une procédure pas à pas avec un cas d’utilisation.

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

## <a name="create-an-azure-synapse-analytics-linked-service-using-ui"></a>Créer un service lié Azure Synapse Analytics à l’aide de l’IU

Pour créer un service lié Azure Synapse Analytics dans l’interface utilisateur du portail Azure, procédez comme suit.

1. Accédez à l’onglet Gérer dans votre espace de travail Azure Data Factory ou Synapse et sélectionnez Services liés, puis cliquez sur Nouveau :

    # <a name="azure-data-factory"></a>[Azure Data Factory](#tab/data-factory).

    :::image type="content" source="media/doc-common-process/new-linked-service.png" alt-text="Capture d’écran de la création d’un nouveau service lié avec l’interface utilisateur Azure Data Factory.":::

    # <a name="azure-synapse"></a>[Azure Synapse](#tab/synapse-analytics)

    :::image type="content" source="media/doc-common-process/new-linked-service-synapse.png" alt-text="Capture d’écran de la création d’un nouveau service lié avec l’interface utilisateur Azure Synapse.":::

2. Recherchez Synapse et sélectionnez le connecteur Azure Synapse Analytics.

    :::image type="content" source="media/connector-azure-sql-data-warehouse/azure-sql-data-warehouse-connector.png" alt-text="Capture d’écran montrant le connecteur Azure Synapse Analytics.":::    

1. Configurez les informations du service, testez la connexion et créez le nouveau service lié.

    :::image type="content" source="media/connector-azure-sql-data-warehouse/configure-azure-sql-data-warehouse-linked-service.png" alt-text="Capture d’écran de la configuration d’un service lié Azure Synapse Analytics.":::

## <a name="connector-configuration-details"></a>Informations de configuration des connecteurs

Les sections suivantes fournissent des informations détaillées sur les propriétés qui définissent les entités de pipeline Data Factory et Synapse propres à un connecteur Azure Synapse Analytics.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Azure Synapse Analytics sont les suivantes :

| Propriété            | Description                                                  | Obligatoire                                                     |
| :------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| type                | La propriété de type doit être définie sur **AzureSqlDW**             | Oui                                                          |
| connectionString    | Spécifiez les informations requises pour la connexion à l’instance Azure Synapse Analytics pour la propriété **connectionString**. <br/>Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité. Vous pouvez également stocker un mot de passe/une clé de principal de service dans Azure Key Vault et, en cas d'authentification SQL, extraire la configuration `password` de la chaîne de connexion. Pour plus d'informations, reportez-vous à l'exemple JSON décrit sous le tableau et à l'article [Stocker des informations d'identification dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui                                                          |
| servicePrincipalId  | Spécifiez l’ID client de l’application.                         | Oui, quand vous utilisez l’authentification Azure AD avec le principal de service. |
| servicePrincipalKey | Spécifiez la clé de l’application. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui, quand vous utilisez l’authentification Azure AD avec le principal de service. |
| tenant              | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Vous pouvez le récupérer en pointant la souris dans le coin supérieur droit du Portail Azure. | Oui, quand vous utilisez l’authentification Azure AD avec le principal de service. |
| azureCloudType | Pour l'authentification du principal de service, spécifiez le type d'environnement cloud Azure auquel votre application Azure AD est inscrite. <br/> Les valeurs autorisées sont `AzurePublic`, `AzureChina`, `AzureUsGovernment` et `AzureGermany`. Par défaut, l’environnement cloud du pipeline de fabrique de données ou Synapse est utilisé. | Non |
| connectVia          | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser Azure Integration Runtime ou un runtime d’intégration auto-hébergé (si votre magasin de données se trouve sur un réseau privé). À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. | Non                                                           |

Pour en savoir plus sur les autres types d’authentification, consultez les sections suivantes sur les prérequis et les exemples JSON, respectivement :

- [Authentification SQL](#sql-authentication)
- Authentification par jeton d’application Azure AD : [Principal du service](#service-principal-authentication)
- Authentification par jeton d’application Azure AD : [Identités managées pour les ressources Azure](#managed-identity)

>[!TIP]
>Lors de la création d’un service lié pour un pool SQL **serverless** Azure Synapse à partir de l’interface utilisateur, choisissez une « entrée manuelle » au lieu de naviguer à partir de l’abonnement.

>[!TIP]
>Si vous rencontrez une erreur avec le code d’erreur « UserErrorFailedToConnectToSqlServer » et un message tel que « La limite de session pour la base de données est XXX et a été atteinte. », ajoutez `Pooling=false` à votre chaîne de connexion, puis réessayez.

### <a name="sql-authentication"></a>Authentification SQL

#### <a name="linked-service-example-that-uses-sql-authentication"></a>Exemple de service lié qui utilise l’authentification SQL

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Mot de passe dans Azure Key Vault :**

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30",
            "password": {
                "type": "AzureKeyVaultSecret",
                "store": {
                    "referenceName": "<Azure Key Vault linked service name>",
                    "type": "LinkedServiceReference"
                },
                "secretName": "<secretName>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="service-principal-authentication"></a>Authentification d’un principal du service

Pour utiliser l’authentification du jeton d’application Azure AD basée sur le principal de service, effectuez les étapes suivantes :

1. **[Créez une application Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal)** dans le portail Azure. Prenez note du nom de l’application et des valeurs suivantes qui définissent le service lié :

   - ID de l'application
   - Clé de l'application
   - ID client

2. **[Aprovisionnez un administrateur Azure Active Directory](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-database)** pour votre serveur sur le portail Azure, si ce n’est pas déjà fait. L’administrateur Azure AD peut être un utilisateur Azure AD ou un groupe Azure AD. Si vous accordez au groupe avec identité managée un rôle d’administrateur, ignorez les étapes 3 et 4. L’administrateur a un accès complet à la base de données.

3. **[Créez des utilisateurs de base de données autonome](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities)** pour le principal de service. Connectez-vous à l’entrepôt de données vers lequel ou à partir duquel vous souhaitez copier des données à l’aide d’outils tels que SSMS, avec une identité Azure AD qui a au moins l’autorisation ALTER ANY USER. Exécutez la commande T-SQL suivante :
  
    ```sql
    CREATE USER [your_application_name] FROM EXTERNAL PROVIDER;
    ```

4. **Accordez les autorisations requises par le principal de service** comme vous le feriez d’habitude pour des utilisateurs SQL ou autres. Exécutez le code suivant, ou consultez davantage d’options [ici](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql). Si vous souhaitez utiliser PolyBase pour charger les données, consultez l’[autorisation de base de données requise](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your application name];
    ```

5. **Configurez un service lié Azure Synapse Analytics** dans un espace de travail Azure Data Factory ou Synapse.

#### <a name="linked-service-example-that-uses-service-principal-authentication"></a>Exemple de service lié qui utilise l’authentification du principal de service

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Identités managées pour authentifier les ressources Azure

Une fabrique de données ou un espace de travail Synapse peut être associé à une [identité managée pour les ressources Azure](data-factory-service-identity.md), laquelle représente la ressource. Vous pouvez utiliser cette identité managée pour l’authentification Azure Synapse Analytics. La ressource désignée peut accéder et copier des données vers ou à partir de votre entrepôt de données à l’aide de cette identité.

Pour utiliser l’authentification par identité managée, effectuez les étapes suivantes :

1. **[Aprovisionnez un administrateur Azure Active Directory](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-database)** pour votre serveur sur le portail Azure, si ce n’est pas déjà fait. L’administrateur Azure AD peut être un utilisateur Azure AD ou un groupe Azure AD. Si vous accordez au groupe avec identité managée un rôle d’administrateur, ignorez les étapes 3 et 4. L’administrateur a un accès complet à la base de données.

2. **[Créez des utilisateurs de base de données autonome](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities)** pour l’identité managée. Connectez-vous à l’entrepôt de données vers lequel ou à partir duquel vous souhaitez copier des données à l’aide d’outils tels que SSMS, avec une identité Azure AD qui a au moins l’autorisation ALTER ANY USER. Exécutez le code T-SQL suivant.
  
    ```sql
    CREATE USER [your_resource_name] FROM EXTERNAL PROVIDER;
    ```

3. **Accordez les autorisations requises par l’identité managée** comme vous le feriez d’habitude pour des utilisateurs SQL et autres. Exécutez le code suivant, ou consultez davantage d’options [ici](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql). Si vous souhaitez utiliser PolyBase pour charger les données, consultez l’[autorisation de base de données requise](#required-database-permission).

    ```sql
    EXEC sp_addrolemember db_owner, [your_resource_name];
    ```

4. **Configurez un service lié Azure Synapse Analytics**.

**Exemple :**

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;Connection Timeout=30"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md).

Les propriétés prises en charge pour le jeu de données Azure Synapse Analytics sont les suivantes :

| Propriété  | Description                                                  | Obligatoire                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | La propriété **type** du jeu de données doit être définie sur **AzureSqlDWTable**. | Oui                         |
| schéma | Nom du schéma. |Non pour Source, Oui pour Récepteur  |
| table | Nom de la table/vue. |Non pour Source, Oui pour Récepteur  |
| tableName | Nom de la table/vue avec schéma. Cette propriété est prise en charge pour la compatibilité descendante. Pour les nouvelles charges de travail, utilisez `schema` et `table`. | Non pour Source, Oui pour Récepteur |

### <a name="dataset-properties-example"></a>Exemple de propriétés du jeu de données

```json
{
    "name": "AzureSQLDWDataset",
    "properties":
    {
        "type": "AzureSqlDWTable",
        "linkedServiceName": {
            "referenceName": "<Azure Synapse Analytics linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "schema": "<schema_name>",
            "table": "<table_name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par Azure Synapse Analytics en tant que source et récepteur.

### <a name="azure-synapse-analytics-as-the-source"></a>Azure Synapse Analytics en tant que source

>[!TIP]
>Pour savoir comment charger efficacement des données à partir d’Azure Synapse Analytics à l’aide du partitionnement des données, consultez [Copier en parallèle à partir d’Azure Synapse Analytics](#parallel-copy-from-azure-synapse-analytics).

Pour copier des données depuis Azure Synapse Analytics, affectez la valeur **SqlDWSource** à la propriété **type** dans la source d’activité de copie. Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété                     | Description                                                  | Obligatoire |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | La propriété **type** de la source d’activité de copie doit être définie sur **SqlDWSource**. | Oui      |
| sqlReaderQuery               | Utiliser la requête SQL personnalisée pour lire les données. Exemple : `select * from MyTable`. | Non       |
| sqlReaderStoredProcedureName | Nom de la procédure stockée qui lit les données de la table source. La dernière instruction SQL doit être une instruction SELECT dans la procédure stockée. | Non       |
| storedProcedureParameters    | Paramètres de la procédure stockée.<br/>Les valeurs autorisées sont des paires de noms ou de valeurs. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. | Non       |
| isolationLevel | Spécifie le comportement de verrouillage des transactions pour la source SQL. Les valeurs autorisées sont les suivantes : **ReadCommitted**, **ReadUncommitted**, **RepeatableRead**, **Serializable**, **Snapshot**. S’il n’est pas spécifié, le niveau d’isolation par défaut de la base de données est utilisé. Pour plus d’informations, consultez [system.data.isolationlevel](/dotnet/api/system.data.isolationlevel). | Non |
| partitionOptions | Spécifie les options de partitionnement des données utilisées pour charger des données à partir d’Azure Synapse Analytics. <br>Les valeurs autorisées sont les suivantes : **None** (valeur par défaut), **PhysicalPartitionsOfTable** et **DynamicRange**.<br>Lorsqu’une option de partition est activée (autrement dit, pas `None`), le degré de parallélisme pour charger simultanément des données à partir d’une instance Azure Synapse Analytics est contrôlé par le paramètre [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de l’activité de copie. | Non |
| partitionSettings | Spécifiez le groupe de paramètres pour le partitionnement des données. <br>S’applique lorsque l’option de partitionnement n’est pas `None`. | Non |
| ***Sous `partitionSettings`:*** | | |
| partitionColumnName | Spécifiez le nom de la colonne source **en type entier ou date/DateHeure** (`int`, `smallint`, `bigint`, `date`, `smalldatetime`, `datetime`, `datetime2` ou `datetimeoffset`) qui sera utilisé par le partitionnement par plages de valeurs pour la copie en parallèle. S’il n’est pas spécifié, l’index ou la clé primaire de la table est détecté automatiquement et utilisé comme colonne de partition.<br>S’applique lorsque l’option de partitionnement est `DynamicRange`. Si vous utilisez une requête pour récupérer des données sources, utilisez `?AdfDynamicRangePartitionCondition ` dans la clause WHERE. Pour obtenir un exemple, consultez la section [Copier en parallèle à partir de la base de données SQL](#parallel-copy-from-azure-synapse-analytics). | Non |
| partitionUpperBound | Valeur maximale de la colonne de partition pour le fractionnement de la plage de partition. Cette valeur est utilisée pour décider du stride de la partition, et non pour filtrer les lignes de la table. Toutes les lignes de la table ou du résultat de la requête seront partitionnées et copiées. Si la valeur n’est pas spécifiée, l’activité de copie la détecte automatiquement.  <br>S’applique lorsque l’option de partitionnement est `DynamicRange`. Pour obtenir un exemple, consultez la section [Copier en parallèle à partir de la base de données SQL](#parallel-copy-from-azure-synapse-analytics). | Non |
| partitionLowerBound | Valeur minimale de la colonne de partition pour le fractionnement de la plage de partition. Cette valeur est utilisée pour décider du stride de la partition, et non pour filtrer les lignes de la table. Toutes les lignes de la table ou du résultat de la requête seront partitionnées et copiées. Si la valeur n’est pas spécifiée, l’activité de copie la détecte automatiquement.<br>S’applique lorsque l’option de partitionnement est `DynamicRange`. Pour obtenir un exemple, consultez la section [Copier en parallèle à partir de la base de données SQL](#parallel-copy-from-azure-synapse-analytics). | Non |

**Notez le point suivant :**

- Lorsque vous utilisez une procédure stockée dans la source pour récupérer des données, sachez que si votre procédure stockée est conçue pour renvoyer un schéma différent quand une valeur de paramètre différente est entrée, vous risquez d’échouer ou d’obtenir un résultat inattendu lors de l’importation d’un schéma à partir de l’interface utilisateur ou lors de la copie de données dans la base de données SQL avec création de table automatique.

#### <a name="example-using-sql-query"></a>Exemple : utilisation d’une requête SQL

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure Synapse Analytics input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlDWSource",
                "sqlReaderQuery": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="example-using-stored-procedure"></a>Exemple : utilisation d’une procédure stockée

```json
"activities":[
    {
        "name": "CopyFromAzureSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure Synapse Analytics input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlDWSource",
                "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
                "storedProcedureParameters": {
                    "stringData": { "value": "str3" },
                    "identifier": { "value": "$$Text.Format('{0:yyyy}', <datetime parameter>)", "type": "Int"}
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="sample-stored-procedure"></a>Exemple de procédure stockée :

```sql
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
    select *
    from dbo.UnitTestSrcTable
    where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="azure-synapse-analytics-as-sink"></a><a name="azure-sql-data-warehouse-as-sink"></a> Azure Synapse Analytics en tant que récepteur

Les pipelines Azure Data Factory et Synapse prennent en charge trois manières de charger des données dans Azure Synapse Analytics.

- [Utiliser l’instruction COPY](#use-copy-statement)
- [Utiliser PolyBase](#use-polybase-to-load-data-into-azure-synapse-analytics)
- Utiliser l’insertion en bloc

La méthode la plus rapide et la plus évolutive pour charger des données consiste à utiliser [l’instruction COPY](/sql/t-sql/statements/copy-into-transact-sql) ou [PolyBase](/sql/relational-databases/polybase/polybase-guide).

Pour copier des données vers Azure Synapse Analytics, définissez **SqlDWSink** comme type de récepteur dans l’activité de copie. Les propriétés suivantes sont prises en charge dans la section **sink** de l’activité de copie :

| Propriété          | Description                                                  | Obligatoire                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| type              | La propriété **type** du récepteur d’activité de copie doit être définie sur **SqlDWSink**. | Oui                                           |
| allowPolyBase     | Indique s’il faut utiliser PolyBase pour charger des données dans Azure Synapse Analytics. `allowCopyCommand` et `allowPolyBase` ne peuvent pas avoir tous les deux la valeur True. <br/><br/>Reportez-vous à la section [Utiliser PolyBase pour charger des données dans Azure Synapse Analytics](#use-polybase-to-load-data-into-azure-synapse-analytics) pour connaître les contraintes et les détails.<br/><br/>Les valeurs autorisées sont **True** et **False** (par défaut). | Non.<br/>Appliquer lors de l’utilisation de PolyBase.     |
| polyBaseSettings  | Groupe de propriétés pouvant être spécifié lorsque la propriété `allowPolybase` est définie sur **true**. | Non.<br/>Appliquer lors de l’utilisation de PolyBase. |
| allowCopyCommand | Indique s’il faut utiliser [l’instruction COPY](/sql/t-sql/statements/copy-into-transact-sql) pour charger des données dans Azure Synapse Analytics. `allowCopyCommand` et `allowPolyBase` ne peuvent pas avoir tous les deux la valeur True. <br/><br/>Reportez-vous à la section [Utiliser l’instruction COPY pour charger des données dans Azure Synapse Analytics](#use-copy-statement) pour connaître les contraintes et les détails.<br/><br/>Les valeurs autorisées sont **True** et **False** (par défaut). | Non.<br>Appliquer lors de l’utilisation de l’instruction COPY. |
| copyCommandSettings | Groupe de propriétés pouvant être spécifié lorsque la propriété `allowCopyCommand` est définie sur TRUE. | Non.<br/>Appliquer lors de l’utilisation de l’instruction COPY. |
| writeBatchSize    | Nombre de lignes à insérer dans le tableau SQL **par lot**.<br/><br/>La valeur autorisée est **integer** (nombre de lignes). Par défaut, le service détermine de façon dynamique la taille de lot appropriée selon la taille de ligne. | Non.<br/>Appliquer lors de l’utilisation de l’insertion en bloc.     |
| writeBatchTimeout | Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer.<br/><br/>La valeur autorisée est **timespan**. Exemple : « 00:30:00 » (30 minutes). | Non.<br/>Appliquer lors de l’utilisation de l’insertion en bloc.        |
| preCopyScript     | Spécifiez une requête SQL pour l’activité de copie à exécuter avant l’écriture de données dans Azure Synapse Analytics à chaque exécution. Utilisez cette propriété pour nettoyer les données préchargées. | Non                                            |
| tableOption | Spécifie si [la table du récepteur doit être créée automatiquement](copy-activity-overview.md#auto-create-sink-tables) si elle n’existe pas en fonction du schéma source. Les valeurs autorisées sont `none` (par défaut) et `autoCreate`. |Non |
| disableMetricsCollection | Le service collecte des métriques telles que les DWU Azure Synapse Analytics pour effectuer des recommandations et optimiser les performances de copie, introduisant un accès de base de données master supplémentaire. Si ce comportement vous préoccupe, spécifiez `true` pour le désactiver. | Non (la valeur par défaut est `false`) |
| maxConcurrentConnections |La limite supérieure de connexions simultanées établies au magasin de données pendant l’exécution de l’activité. Spécifiez une valeur uniquement lorsque vous souhaitez limiter les connexions simultanées.| Non |

#### <a name="azure-synapse-analytics-sink-example"></a>Exemple de récepteur Azure Synapse Analytics

```json
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true,
    "polyBaseSettings":
    {
        "rejectType": "percentage",
        "rejectValue": 10.0,
        "rejectSampleValue": 100,
        "useTypeDefault": true
    }
}
```

## <a name="parallel-copy-from-azure-synapse-analytics"></a>Copie en parallèle à partir d’Azure Synapse Analytics

Le connecteur Azure Synapse Analytics dans l’activité de copie propose un partitionnement de données intégré pour copier des données en parallèle. Vous trouverez des options de partitionnement de données dans l’onglet **Source** de l’activité de copie.

![Capture d’écran représentant les options de partition](./media/connector-sql-server/connector-sql-partition-options.png)

Lorsque vous activez la copie partitionnée, l’activité de copie exécute des requêtes en parallèle sur votre source Azure Synapse Analytics pour charger des données par partitions. Le degré de parallélisme est contrôlé via le paramètre [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) sur l’activité de copie. Par exemple, si vous définissez `parallelCopies` sur la valeur quatre, le service génère et exécute simultanément quatre requêtes selon l’option de partition et les paramètres que vous avez spécifiés, chacune récupérant des données à partir de votre instance Azure Synapse Analytics.

Il vous est recommandé d’activer la copie en parallèle avec partitionnement des données notamment lorsque vous chargez une grande quantité de données à partir de votre instance Azure Synapse Analytics. Voici quelques suggestions de configurations pour différents scénarios. Lors de la copie de données dans un magasin de données basé sur un fichier, il est recommandé d’écrire les données dans un dossier sous la forme de plusieurs fichiers (spécifiez uniquement le nom du dossier). Les performances seront meilleures qu’avec l’écriture de données dans un seul fichier.

| Scénario                                                     | Paramètres suggérés                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Chargement complet à partir d’une table volumineuse, avec des partitions physiques.        | **Option de partition** : Partitions physiques de la table. <br><br/>Pendant l’exécution, le service détecte automatiquement les partitions physiques et copie les données par partition. <br><br/>Pour vérifier si votre table possède, ou non, une partition physique, vous pouvez vous reporter à [cette requête](#sample-query-to-check-physical-partition). |
| Chargement complet d’une table volumineuse, sans partitions physiques, avec une colonne d’entiers ou DateHeure pour le partitionnement des données. | **Options de partition** : Partition dynamique par spécification de plages de valeurs.<br>**Colonne de partition** (facultatif) : Spécifiez la colonne utilisée pour partitionner les données. Si la valeur n’est pas spécifiée, la colonne de l’index ou de la clé primaire est utilisée.<br/>**Limite supérieure de partition** et **limite inférieure de partition** (facultatif) : Spécifiez si vous souhaitez déterminer le stride de la partition. Cela ne permet pas de filtrer les lignes de la table ; toutes les lignes de la table sont partitionnées et copiées. Si les valeurs ne sont pas spécifiées, l’activité de copie les détecte automatiquement.<br><br>Par exemple, si les valeurs de la colonne de partition « ID » sont comprises entre 1 et 100, et que vous définissez la limite inférieure à 20 et la limite supérieure à 80, avec la copie parallèle sur 4, le service récupère des données par 4 partitions, les ID dans la plage <=20, [21, 50], [51, 80] et >=81, respectivement. |
| Chargement d’une grande quantité de données à l’aide d’une requête personnalisée, sans partitions physiques, et avec une colonne d’entiers ou de date/DateHeure pour le partitionnement des données. | **Options de partition** : Partition dynamique par spécification de plages de valeurs.<br>**Requête**: `SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`.<br>**Colonne de partition** : Spécifiez la colonne utilisée pour partitionner les données.<br>**Limite supérieure de partition** et **limite inférieure de partition** (facultatif) : Spécifiez si vous souhaitez déterminer le stride de la partition. Cela ne permet pas de filtrer les lignes de la table ; toutes les lignes du résultat de la requête sont partitionnées et copiées. Si la valeur n’est pas spécifiée, l’activité de copie la détecte automatiquement.<br><br>Lors de l’exécution, le service remplace `?AdfRangePartitionColumnName` par le nom réel de la colonne et les plages de valeurs de chaque partition, et les envoie à Azure Synapse Analytics. <br>Par exemple, si les valeurs de la colonne de partition « ID » sont comprises entre 1 et 100, et que vous définissez la limite inférieure à 20 et la limite supérieure à 80, avec la copie parallèle sur 4, le service récupère des données par 4 partitions, les ID dans la plage <=20, [21, 50], [51, 80] et >=81, respectivement. <br><br>Voici d’autres exemples de requêtes pour différents scénarios :<br> 1. Interroger l’ensemble de la table : <br>`SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition`<br> 2. Interroger une table avec une sélection de colonnes et des filtres de la clause WHERE supplémentaires : <br>`SELECT <column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 3. Effectuer une requête avec des sous-requêtes : <br>`SELECT <column_list> FROM (<your_sub_query>) AS T WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>`<br> 4. Effectuer une requête avec une partition dans une sous-requête : <br>`SELECT <column_list> FROM (SELECT <your_sub_query_column_list> FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition) AS T`
|

Meilleures pratiques pour charger des données avec l’option de partition :

1. Choisissez une colonne distinctive comme colonne de partition (p. ex. : clé primaire ou clé unique) pour éviter l’asymétrie des données. 
2. Si la table possède une partition intégrée, utilisez l’option de partition « Partitions physiques de la table » pour obtenir de meilleures performances.
3. Si vous utilisez Azure Integration Runtime pour copier des données, vous pouvez définir des « [unités d’intégration de données (DIU)](copy-activity-performance-features.md#data-integration-units) » plus grandes (>4) pour utiliser davantage de ressources de calcul. Vérifiez les scénarios applicables ici.
4. Le « [degré de parallélisme de copie](copy-activity-performance-features.md#parallel-copy) » contrôle le nombre de partitions : un nombre trop élevé nuit parfois aux performances. Il est recommandé de définir ce nombre selon (DIU ou nombre de nœuds d'IR auto-hébergé) * (2 à 4).
5. Notez qu’Azure Synapse Analytics peut exécuter un maximum de 32 requêtes à un moment donné. La définition d’un « degré de parallélisme de copie » trop important peut entraîner un problème de limitation de Synapse.

**Exemple : chargement complet à partir d’une table volumineuse, avec des partitions physiques**

```json
"source": {
    "type": "SqlDWSource",
    "partitionOption": "PhysicalPartitionsOfTable"
}
```

**Exemple : requête avec partition dynamique par spécification de plages de valeurs**

```json
"source": {
    "type": "SqlDWSource",
    "query": "SELECT * FROM <TableName> WHERE ?AdfDynamicRangePartitionCondition AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column (optional) to decide the partition stride, not as data filter>",
        "partitionLowerBound": "<lower_value_of_partition_column (optional) to decide the partition stride, not as data filter>"
    }
}
```

### <a name="sample-query-to-check-physical-partition"></a>Exemple de requête pour vérifier une partition physique

```sql
SELECT DISTINCT s.name AS SchemaName, t.name AS TableName, c.name AS ColumnName, CASE WHEN c.name IS NULL THEN 'no' ELSE 'yes' END AS HasPartition
FROM sys.tables AS t
LEFT JOIN sys.objects AS o ON t.object_id = o.object_id
LEFT JOIN sys.schemas AS s ON o.schema_id = s.schema_id
LEFT JOIN sys.indexes AS i ON t.object_id = i.object_id
LEFT JOIN sys.index_columns AS ic ON ic.partition_ordinal > 0 AND ic.index_id = i.index_id AND ic.object_id = t.object_id
LEFT JOIN sys.columns AS c ON c.object_id = ic.object_id AND c.column_id = ic.column_id
LEFT JOIN sys.types AS y ON c.system_type_id = y.system_type_id
WHERE s.name='[your schema]' AND t.name = '[your table name]'
```

Si la table possède une partition physique, vous voyez « HasPartition » avec la valeur « yes » (oui).

## <a name="use-copy-statement-to-load-data-into-azure-synapse-analytics"></a><a name="use-copy-statement"></a> Utiliser l’instruction COPY pour charger des données dans Azure Synapse Analytics

L’utilisation de l’[instruction COPY](/sql/t-sql/statements/copy-into-transact-sql) est un moyen simple et flexible de charger des données dans Azure Synapse Analytics avec un débit élevé. Pour plus d’informations, consultez [Chargement en masse de données à l’aide de l’instruction Copy](../synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql.md)


- Si vos données sources se trouvent dans **Stockage Blob Azure ou Azure Data Lake Storage Gen2** et que **le format est compatible avec l’instruction COPY**, vous pouvez utiliser l’activité de copie pour appeler directement l’instruction COPY et permettre à Azure Synapse Analytics d’extraire les données de la source. Pour plus d’informations, consultez **[Copie directe à l’aide de l’instruction COPY](#direct-copy-by-using-copy-statement)** .
- Si votre magasin de données source et son format ne sont pas pris en charge à l’origine par l’instruction COPY, utilisez plutôt la fonctionnalité **[Copie intermédiaire avec l’instruction COPY](#staged-copy-by-using-copy-statement)** . La fonctionnalité de copie intermédiaire offre également un meilleur débit. Elle convertit automatiquement les données dans un format compatible avec l’instruction COPY, stocke les données dans Stockage Blob Azure, puis appelle l’instruction COPY pour charger les données dans Azure Synapse Analytics.

>[!TIP]
>Lorsque l’instruction COPY est utilisée avec Azure Integration Runtime, le nombre réel [d’unités d’intégration de données](copy-activity-performance-features.md#data-integration-units) est toujours de 2. La modification de ce nombre n’a aucun impact sur les performances, étant donné que le chargement de données à partir du stockage est alimenté par le moteur Azure Synapse.

### <a name="direct-copy-by-using-copy-statement"></a>Copie directe à l’aide de l’instruction COPY

L’instruction COPY Azure Synapse Analytics prend directement en charge Stockage Blob Azure, Azure Data Lake Storage Gen1 et Azure Data Lake Storage Gen2. Si vos données sources répondent aux critères décrits dans cette section, utilisez l’instruction COPY pour copier directement à partir du magasin de données source vers Azure Synapse Analytics. Sinon, utilisez la méthode [Copie intermédiaire à l’aide de l’instruction COPY](#staged-copy-by-using-copy-statement). Le service vérifie les paramètres et fait échouer l’exécution de l’activité de copie si les critères ne sont pas satisfaits.

1. Le **format et le service liés à la source** ont les types et méthodes d’authentification suivants :

    | Type de magasin de données source pris en charge                             | Format pris en charge           | Type d’authentification source pris en charge                         |
    | :----------------------------------------------------------- | -------------------------- | :----------------------------------------------------------- |
    | [Blob Azure](connector-azure-blob-storage.md)                | [Texte délimité](format-delimited-text.md)             | Authentification avec clé de compte, authentification avec signature d’accès partagé, authentification avec principal du service, authentification avec identité managée |
    | &nbsp;                                                       | [Parquet](format-parquet.md)                    | Authentification avec clé de compte, authentification avec signature d’accès partagé |
    | &nbsp;                                                       | [ORC](format-orc.md)                        | Authentification avec clé de compte, authentification avec signature d’accès partagé |
    | [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | [Texte délimité](format-delimited-text.md)<br/>[Parquet](format-parquet.md)<br/>[ORC](format-orc.md) | Authentification avec clé de compte, authentification avec principal du service, authentification avec identité managée |

    >[!IMPORTANT]
    >- Si vous utilisez l’authentification par identité managée pour votre service lié de stockage, découvrez les configurations nécessaires pour [Azure Blob](connector-azure-blob-storage.md#managed-identity) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#managed-identity).
    >- Si votre Stockage Azure est configuré avec un point de terminaison de service de type réseau virtuel, vous devez utiliser l’authentification par identité managée et activer « Autoriser le service Microsoft approuvé » sur le compte de stockage. Consultez [Impact du recours à des points de terminaison de service de type réseau virtuel avec le Stockage Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage).

2. Les paramètres du format sont comme suit :

   1. Pour **Parquet** : `compression` peut être **no compression**, **Snappy** ou **``GZip``** .
   2. Pour **ORC** : `compression` peut être **sans compression**, **```zlib```** ou **Snappy**.
   3. Pour **Texte délimité** :
      1. `rowDelimiter` est explicitement défini comme **caractère unique** ou « **\r\n** », la valeur par défaut n’est pas prise en charge.
      2. `nullValue` conserve sa valeur par défaut ou est défini sur **chaîne vide** ("").
      3. `encodingName` conserve sa valeur par défaut ou est défini sur **utf-8 ou utf-16**.
      4. `escapeChar` doit être identique à `quoteChar` et n’est pas vide.
      5. `skipLineCount` conserve sa valeur par défaut ou est défini sur 0.
      6. `compression` peut être **no compression** ou **``GZip``** .

3. Si votre source est un dossier, `recursive` dans l’activité de copie doit être défini sur true, et `wildcardFilename` doit être `*` ou `*.*`. 

4. `wildcardFolderPath` , `wildcardFilename` (autre que `*` ou `*.*`), `modifiedDateTimeStart`, `modifiedDateTimeEnd`, `prefix`, `enablePartitionDiscovery` et `additionalColumns` ne sont pas spécifiés.

Les paramètres de l’instruction COPY suivants sont pris en charge sous `allowCopyCommand` dans l’activité de copie :

| Propriété          | Description                                                  | Obligatoire                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| defaultValues | Spécifie les valeurs par défaut de chaque colonne cible dans Azure Synapse Analytics.  Les valeurs par défaut de la propriété remplacent le jeu de contraintes DEFAULT défini dans l’entrepôt de données, et la colonne d’identité ne peut pas avoir de valeur par défaut. | Non |
| additionalOptions | Options supplémentaires qui seront passées à l’instruction COPY d’Azure Synapse Analytics directement dans la clause « With » dans [l’instruction COPY](/sql/t-sql/statements/copy-into-transact-sql). Citez la valeur en fonction des besoins pour l’aligner sur les spécifications de l’instruction COPY. | Non |

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaCOPY",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "ParquetDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ParquetSource",
                "storeSettings":{
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowCopyCommand": true,
                "copyCommandSettings": {
                    "defaultValues": [
                        {
                            "columnName": "col_string",
                            "defaultValue": "DefaultStringValue"
                        }
                    ],
                    "additionalOptions": {
                        "MAXERRORS": "10000",
                        "DATEFORMAT": "'ymd'"
                    }
                }
            },
            "enableSkipIncompatibleRow": true
        }
    }
]
```

### <a name="staged-copy-by-using-copy-statement"></a>Copie intermédiaire à l’aide de l’instruction COPY

Quand vos données sources ne sont pas compatibles en mode natif avec l’instruction COPY, activez la copie des données par le biais d’un Stockage Blob Azure intermédiaire ou d’Azure Data Lake Storage Gen2 (il ne peut pas s’agir du Stockage Premium Azure). Dans ce cas, le service convertit automatiquement les données pour répondre aux exigences de format de données de l’instruction COPY. Il appelle ensuite l’instruction COPY pour charger des données dans Azure Synapse Analytics. Enfin, il nettoie vos données temporaires du stockage. Pour plus d’informations sur la copie de données par le biais d’un stockage de préproduction, consultez [Copie ](copy-activity-performance-features.md#staged-copy).

Pour utiliser cette fonctionnalité, créez un [service lié Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) ou un [service lié Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) avec **authentification par clé de compte ou identité managée par le système** qui désigne le compte de stockage Azure comme stockage temporaire.

>[!IMPORTANT]
>- Si vous utilisez l’authentification par identité managée pour votre service lié de préproduction, découvrez les configurations nécessaires pour [Azure Blob](connector-azure-blob-storage.md#managed-identity) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#managed-identity).
>- Si votre Stockage Azure de préproduction est configuré avec un point de terminaison de service de type réseau virtuel, vous devez utiliser l’authentification par identité managée et activer « Autoriser le service Microsoft approuvé » sur le compte de stockage. Consultez [Impact du recours à des points de terminaison de service de type réseau virtuel avec le Stockage Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage). 

>[!IMPORTANT]
>Si votre stockage Azure intermédiaire est configuré avec un point de terminaison privé géré et avec le pare-feu de stockage activé, vous devez utiliser l’authentification par identité managée et accorder des autorisations de lecteur de données d’objet blob de stockage à Synapse SQL Server pour qu’il puisse accéder aux fichiers intermédiaires pendant le chargement de l’instruction COPY.

```json
"activities":[
    {
        "name": "CopyFromSQLServerToSQLDataWarehouseViaCOPYstatement",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "SQLServerDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowCopyCommand": true
            },
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingStorage",
                    "type": "LinkedServiceReference"
                }
            }
        }
    }
]
```

## <a name="use-polybase-to-load-data-into-azure-synapse-analytics"></a>Utiliser PolyBase pour charger des données dans Azure Synapse Analytics

L’utilisation de [PolyBase](/sql/relational-databases/polybase/polybase-guide) est un moyen efficace de charger de grandes quantités de données dans Azure Synapse Analytics avec un débit élevé. Vous profiterez d’un gain important de débit en utilisant PolyBase au lieu du mécanisme BULKINSERT par défaut. Consultez [Charger 1 To dans Azure Synapse Analytics](v1/data-factory-load-sql-data-warehouse.md) pour obtenir une procédure pas à pas avec un cas d’utilisation.

- Si vos données sources se trouvent dans **Stockage Blob Azure, Azure Data Lake Storage Gen1 ou Azure Data Lake Storage Gen2** et que **le format est compatible avec PolyBase**, vous pouvez utiliser l’activité de copie pour appeler directement PolyBase et permettre à Azure Synapse Analytics d’extraire les données de la source. Pour plus d’informations, consultez **[Copie directe à l’aide de PolyBase](#direct-copy-by-using-polybase)** .
- Si votre magasin de données source et son format ne sont pas pris en charge à l’origine par PolyBase, utilisez plutôt la fonctionnalité **[Copie intermédiaire avec PolyBase](#staged-copy-by-using-polybase)** . La fonctionnalité de copie intermédiaire offre également un meilleur débit. Elle convertit automatiquement les données dans un format compatible avec PolyBase, stocke les données dans Stockage Blob Azure, puis appelle PolyBase pour charger les données dans Azure Synapse Analytics.

> [!TIP]
> Apprenez-en plus sur les [Bonnes pratiques d’utilisation de PolyBase](#best-practices-for-using-polybase). Lorsque PolyBase est utilisé avec Azure Integration Runtime, le nombre réel [d’unités d’intégration de données](copy-activity-performance-features.md#data-integration-units) pour le stockage direct ou de préproduction dans Synapse est toujours de 2. La modification de ce nombre n’a aucun impact sur les performances, étant donné que le chargement de données à partir du stockage est alimenté par le moteur Synapse.

Les paramètres PolyBase suivants sont pris en charge sous `polyBaseSettings` dans l’activité de copie :

| Propriété          | Description                                                  | Obligatoire                                      |
| :---------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| rejectValue       | Spécifie le nombre ou le pourcentage de lignes pouvant être rejetées avant l’échec de la requête.<br/><br/>Découvrez-en plus sur les options de rejet de PolyBase dans la section Arguments de [CREATE EXTERNAL TABLE (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql). <br/><br/>Les valeurs autorisées sont : 0 (par défaut), 1, 2, et ainsi de suite. | Non                                            |
| rejectType        | Indique si l’option **rejectValue** est une valeur littérale ou un pourcentage.<br/><br/>Les valeurs autorisées sont **Value** (par défaut) et **Percentage**. | Non                                            |
| rejectSampleValue | Détermine le nombre de lignes à extraire avant que PolyBase recalcule le pourcentage de lignes rejetées.<br/><br/>Les valeurs autorisées sont 1, 2, et ainsi de suite. | Oui, si **rejectType** est **percentage** |
| useTypeDefault    | Spécifie comment gérer les valeurs manquantes dans les fichiers texte délimité lorsque PolyBase récupère les données à partir du fichier texte.<br/><br/>Pour plus d’informations sur cette propriété, consultez la section Arguments dans [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql).<br/><br/>Les valeurs autorisées sont **True** et **False** (par défaut).<br><br> | Non                                            |

### <a name="direct-copy-by-using-polybase"></a>Copie directe à l’aide de PolyBase

Azure Synapse Analytics PolyBase prend directement en charge Stockage Blob Azure, Azure Data Lake Storage Gen1 et Azure Data Lake Storage Gen2. Si vos données sources répondent aux critères décrits dans cette section, utilisez PolyBase pour copier directement à partir du magasin de données source vers Azure Synapse Analytics. Sinon, utilisez la méthode [Copie intermédiaire à l’aide de PolyBase](#staged-copy-by-using-polybase).

> [!TIP]
> Pour copier des données efficacement vers Azure Synapse Analytics, consultez [Azure Data Factory facilite et rend plus pratique la découverte d’insights à partir de données lors de l’utilisation de Data Lake Store avec Azure Synapse Analytics](/archive/blogs/azuredatalake/azure-data-factory-makes-it-even-easier-and-convenient-to-uncover-insights-from-data-when-using-data-lake-store-with-sql-data-warehouse).

Si les critères ne sont pas remplis, le service contrôle les paramètres et rétablit automatiquement le mécanisme BULKINSERT pour le déplacement des données.

1. Le **service lié source** contient les types et méthodes d’authentification suivants :

    | Type de magasin de données source pris en charge                             | Type d’authentification source pris en charge                        |
    | :----------------------------------------------------------- | :---------------------------------------------------------- |
    | [Blob Azure](connector-azure-blob-storage.md)                | Authentification par clé de compte, l’authentification d’une identité gérée |
    | [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) | Authentification d’un principal du service                            |
    | [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | Authentification par clé de compte, l’authentification d’une identité gérée |

    >[!IMPORTANT]
    >- Si vous utilisez l’authentification par identité managée pour votre service lié de stockage, découvrez les configurations nécessaires pour [Azure Blob](connector-azure-blob-storage.md#managed-identity) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#managed-identity).
    >- Si votre Stockage Azure est configuré avec un point de terminaison de service de type réseau virtuel, vous devez utiliser l’authentification par identité managée et activer « Autoriser le service Microsoft approuvé » sur le compte de stockage. Consultez [Impact du recours à des points de terminaison de service de type réseau virtuel avec le Stockage Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage).

2. Le **format de données source** est **Parquet**, **ORC**, ou **Texte délimité**, avec les configurations suivantes :

   1. Le chemin du dossier ne contient pas de filtre de caractères génériques.
   2. Le nom de fichier est vide ou pointe vers un fichier unique. Si vous spécifiez un nom de fichier avec caractères génériques dans l’activité de copie, les seuls caractères autorisés sont `*` et `*.*`.
   3. `rowDelimiter` peut être **\n**, **\r\n** ou **\r** ou conserver sa valeur **par défaut**.
   4. `nullValue` est défini sur **une chaîne vide** («») ou conserve sa valeur par défaut, et `treatEmptyAsNull` conserve sa valeur par défaut ou est défini sur true.
   5. `encodingName` conserve sa valeur par défaut ou est défini sur **utf-8**.
   6. `quoteChar`, `escapeChar` et `skipLineCount` ne sont pas spécifiés. PolyBase est capable d’ignorer la ligne d’en-tête. Cela peut être paramétré en tant que `firstRowAsHeader`.
   7. `compression` peut être **no compression**, **``GZip``** ou **Deflate**.

3. Si votre source est un dossier, `recursive` dans l’activité de copie doit être défini sur true.

4. `wildcardFolderPath`, `wildcardFilename`, `modifiedDateTimeStart`, `modifiedDateTimeEnd`, `prefix`, `enablePartitionDiscovery` ni `additionalColumns` ne sont pas spécifiés.

>[!NOTE]
>Si votre source est un dossier, notez que PolyBase récupère les fichiers du dossier et de tous ses sous-dossiers, mais qu’il ne récupère pas les données des fichiers dont le nom commence par un trait de soulignement (_) ou un point (.), comme indiqué [ici - argument LOCATION](/sql/t-sql/statements/create-external-table-transact-sql#arguments-2).

```json
"activities":[
    {
        "name": "CopyFromAzureBlobToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "ParquetDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "ParquetSource",
                "storeSettings":{
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            }
        }
    }
]
```

### <a name="staged-copy-by-using-polybase"></a>Copie intermédiaire à l’aide de PolyBase

Quand vos données sources ne sont pas compatibles en mode natif avec PolyBase, activez la copie des données par le biais d’un Stockage Blob Azure intermédiaire ou d’Azure Data Lake Storage Gen2 (il ne peut pas s’agir du Stockage Premium Azure). Dans ce cas, le service convertit automatiquement les données pour répondre aux exigences de format de données de PolyBase. Ensuite, il appelle PolyBase pour charger les données dans Azure Synapse Analytics. Enfin, il nettoie vos données temporaires du stockage. Pour plus d’informations sur la copie de données par le biais d’un stockage de préproduction, consultez [Copie ](copy-activity-performance-features.md#staged-copy).

Pour utiliser cette fonctionnalité, créez un [service lié Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) ou un [service lié Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) avec **authentification par clé de compte ou identité managée** qui désigne le compte de stockage Azure comme stockage temporaire.

>[!IMPORTANT]
>- Si vous utilisez l’authentification par identité managée pour votre service lié de préproduction, découvrez les configurations nécessaires pour [Azure Blob](connector-azure-blob-storage.md#managed-identity) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#managed-identity).
>- Si votre Stockage Azure de préproduction est configuré avec un point de terminaison de service de type réseau virtuel, vous devez utiliser l’authentification par identité managée et activer « Autoriser le service Microsoft approuvé » sur le compte de stockage. Consultez [Impact du recours à des points de terminaison de service de type réseau virtuel avec le Stockage Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage). 

>[!IMPORTANT]
>Si votre stockage Azure intermédiaire est configuré avec un point de terminaison privé géré et avec le pare-feu de stockage activé, vous devez utiliser l’authentification d’identité managée et accorder des autorisations de lecteur de données d’objet blob de stockage à Synapse SQL Server pour qu’il puisse accéder aux fichiers intermédiaires pendant le chargement de PolyBase.

```json
"activities":[
    {
        "name": "CopyFromSQLServerToSQLDataWarehouseViaPolyBase",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "SQLServerDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "SqlSource",
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingStorage",
                    "type": "LinkedServiceReference"
                }
            }
        }
    }
]
```

### <a name="best-practices-for-using-polybase"></a>Bonnes pratiques d’utilisation de PolyBase

Les sections suivantes présentent les meilleures pratiques qui s’ajoutent à celles mentionnées dans [Meilleures pratiques pour Azure Synapse Analytics](../synapse-analytics/sql/best-practices-dedicated-sql-pool.md).

#### <a name="required-database-permission"></a>Autorisation de base de données requise

Pour utiliser PolyBase, l’utilisateur qui charge des données dans Azure Synapse Analytics doit disposer de [l’autorisation « CONTROL »](/sql/relational-databases/security/permissions-database-engine) sur la base de données cible. Vous pouvez pour cela l’ajouter en tant que membre du rôle **db_owner**. Découvrez comment procéder dans la [vue d’ensemble d’Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-manage-security.md#authorization).

#### <a name="row-size-and-data-type-limits"></a>Limitations en matière de taille de ligne et de type de données

Les chargements PolyBase sont limités aux lignes inférieures à 1 Mo. Vous ne pouvez pas l’utiliser pour charger VARCHR(MAX), NVARCHAR(MAX) ou VARBINARY(MAX). Pour plus d’informations, consultez [Limites de la capacité du service Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md#loads).

Quand votre source de données comporte des lignes supérieures à 1 Mo, vous pouvez fractionner verticalement les tables sources en plusieurs petites tables. Vérifiez que la taille maximale de chaque ligne ne dépasse pas la limite. Vous pouvez ensuite charger les tables plus petites à l’aide de PolyBase et les fusionner dans Azure Synapse Analytics.

Autrement, pour les données avec des colonnes aussi larges, vous pouvez utiliser non-PolyBase pour charger les données, en désactivant le paramètre « Autoriser PolyBase ».

#### <a name="azure-synapse-analytics-resource-class"></a>Classe de ressources Azure Synapse Analytics

Pour obtenir le meilleur débit possible, attribuez une classe de ressources plus volumineuse à l’utilisateur qui charge des données dans Azure Synapse Analytics par le biais de PolyBase.

#### <a name="polybase-troubleshooting"></a>Résolution des problèmes liés à PolyBase

#### <a name="loading-to-decimal-column"></a>Chargement de la colonne décimale

Si votre source de données est au format texte ou se trouve dans d’autres magasins non compatibles avec PolyBase (avec copie intermédiaire et PolyBase) et qu’elle contient une valeur vide à charger dans une colonne décimale d’Azure Synapse Analytics, vous risquez d’obtenir l’erreur suivante :

```output
ErrorCode=FailedDbOperation, ......HadoopSqlException: Error converting data type VARCHAR to DECIMAL.....Detailed Message=Empty string can't be converted to DECIMAL.....
```

La solution consiste à désélectionner l’option « **Utiliser l’option de type par défaut** » (false) dans Récepteur d’activité de copie -> Paramètres de PolyBase. « [USE_TYPE_DEFAULT](/sql/t-sql/statements/create-external-file-format-transact-sql#arguments) » est une configuration native PolyBase qui spécifie comment gérer les valeurs manquantes dans les fichiers texte délimités quand PolyBase extrait des données à partir du fichier texte.

#### <a name="check-the-tablename-property-in-azure-synapse-analytics"></a>Vérification de la propriété tableName dans Azure Synapse Analytics

Le tableau suivant donne des exemples montrant comment spécifier la propriété **tableName** dans le jeu de données JSON. Il montre plusieurs combinaisons de noms de schéma et de table.

| Schéma BD | Nom de la table | Propriété JSON **tableName**               |
| --------- | ---------- | ----------------------------------------- |
| dbo       | MyTable    | MyTable ou dbo.MyTable ou [dbo].[MyTable] |
| dbo1      | MyTable    | dbo1.MyTable ou [dbo1].[MyTable]          |
| dbo       | My.Table   | [My.Table] ou [dbo].[My.Table]            |
| dbo1      | My.Table   | [dbo1].[My.Table]                         |

Si vous voyez l’erreur suivante, il peut s’agir d’un problème avec la valeur spécifiée pour la propriété **tableName**. Consultez le tableau précédent pour savoir comment spécifier des valeurs pour la propriété JSON **tableName**.

```output
Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider
```

#### <a name="columns-with-default-values"></a>Colonnes avec des valeurs par défaut

Actuellement, la fonctionnalité PolyBase accepte seulement le même nombre de colonnes que dans la table cible. Par exemple, vous avez une table avec quatre colonnes dont l’une est définie avec une valeur par défaut. Les données d’entrée doivent toujours avoir quatre colonnes. Un jeu de données d’entrée de trois colonnes produit une erreur semblable au message suivant :

```output
All columns of the table must be specified in the INSERT BULK statement.
```

La valeur NULL est une forme spéciale de la valeur par défaut. Si la colonne est nullable, les données d’entrée dans l’objet Blob pour cette colonne peuvent être vides. En revanche, elles ne peuvent pas être manquantes dans le jeu de données d’entrée. PolyBase insère NULL pour les données manquantes dans Azure Synapse Analytics.

#### <a name="external-file-access-failed"></a>Échec de l’accès au fichier externe

Si vous recevez l’erreur suivante, vérifiez que vous utilisez l’authentification d’identité managée et que vous avez accordé des autorisations de lecteur de données d’objet blob de stockage à l’identité managée de l’espace de travail Azure Synapse.

```output
Job failed due to reason: at Sink '[SinkName]': shaded.msdataflow.com.microsoft.sqlserver.jdbc.SQLServerException: External file access failed due to internal error: 'Error occurred while accessing HDFS: Java exception raised on call to HdfsBridge_IsDirExist. Java exception message:\r\nHdfsBridge::isDirExist 
```

Pour plus d’informations, consultez [Octroi d’autorisations à l’identité managée après la création de l’espace de travail](../synapse-analytics/security/how-to-grant-workspace-managed-identity-permissions.md#grant-permissions-to-managed-identity-after-workspace-creation).

## <a name="mapping-data-flow-properties"></a>Propriétés du mappage de flux de données

Lors de la transformation de données dans le flux de données de mappage, vous pouvez lire et écrire dans des tables à partir d’Azure Synapse Analytics. Pour plus d’informations, consultez la [transformation de la source](data-flow-source.md) et la [transformation du récepteur](data-flow-sink.md) dans le flux de données de mappage.

### <a name="source-transformation"></a>Transformation de la source

Les paramètres spécifiques à Azure Synapse Analytics sont disponibles dans l’onglet **Options de la source** de la transformation de la source.

**Entrée** Indiquez si votre source pointe vers une table (ce qui correspond à ```Select * from <table-name>```) ou si vous souhaitez entrer une requête SQL personnalisée.

**Activer la préproduction** : Il est vivement recommandé d’utiliser cette option dans les charges de travail de production avec des sources Azure Synapse Analytics. Si vous exécutez une [activité de flux de données](control-flow-execute-data-flow-activity.md) avec des sources Azure Synapse Analytics à partir d’un pipeline, vous êtes invité à entrer un compte de stockage d’emplacement de préproduction que vous utiliserez pour le chargement des données de préproduction. Il s’agit du mécanisme le plus rapide pour charger des données à partir d’Azure Synapse Analytics.

- Si vous utilisez l’authentification par identité managée pour votre service lié de stockage, découvrez les configurations nécessaires pour [Azure Blob](connector-azure-blob-storage.md#managed-identity) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#managed-identity).
- Si votre Stockage Azure est configuré avec un point de terminaison de service de type réseau virtuel, vous devez utiliser l’authentification par identité managée et activer « Autoriser le service Microsoft approuvé » sur le compte de stockage. Consultez [Impact du recours à des points de terminaison de service de type réseau virtuel avec le Stockage Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage).
- Quand vous utilisez un pool SQL **serverless** Azure Synapse en tant que source, l’activation de la préproduction n’est pas prise en charge.

**Requête** : Si vous sélectionnez Requête dans le champ Entrée, entrez une requête SQL pour votre source. Ce paramètre remplace toute table que vous avez choisie dans le jeu de données. Les clauses **Order By** ne sont pas prises en charge ici, mais vous pouvez définir une instruction SELECT FROM complète. Vous pouvez également utiliser des fonctions de table définies par l’utilisateur. **select * from udfGetData()** est une fonction UDF dans SQL qui retourne une table. Cette requête génère une table source que vous pouvez utiliser dans votre flux de données. L’utilisation de requêtes est également un excellent moyen de réduire les lignes pour les tests ou les recherches.

Exemple SQL : ```Select * from MyTable where customerId > 1000 and customerId < 2000```

**Taille du lot** : entrez la taille de lot que doivent avoir les lectures créées à partir d’un large volume de données. Dans les flux de données, ce paramètre permet de définir la mise en cache en colonnes Spark. Il s’agit d’un champ facultatif qui utilise les valeurs par défaut de Spark s’il est laissé vide.

**Niveaux d’isolement** : La valeur par défaut pour les sources SQL dans le flux de données de mappage est Lecture non validée. Vous pouvez remplacer ici le niveau d’isolement par l’une des valeurs suivantes :

- Lecture validée
- Lecture non validée
- Lecture renouvelée
- Sérialisable
- Aucun (ignorer le niveau d’isolement)

![Niveau d’isolation](media/data-flow/isolationlevel.png)

### <a name="sink-transformation"></a>Transformation du récepteur

Les paramètres spécifiques à Azure Synapse Analytics sont disponibles dans l’onglet **Paramètres** de la transformation du récepteur.

**Méthode de mise à jour** : détermine les opérations autorisées sur la destination de votre base de données. Par défaut, seules les insertions sont autorisées. Pour mettre à jour, effectuer un upsert ou supprimer des lignes, une transformation alter-row est requise afin de baliser les lignes relatives à ces actions. Pour les mises à jour, les opérations upsert et les suppressions, une ou plusieurs colonnes clés doivent être définies pour déterminer la ligne à modifier.

**Action table :** Détermine si toutes les lignes de la table de destination doivent être recréées ou supprimées avant l’écriture.

- Aucune : Aucune action ne sera effectuée sur la table.
- Recréer : La table sera supprimée et recréée. Obligatoire en cas de création dynamique d’une nouvelle table.
- Tronquer : Toutes les lignes de la table cible seront supprimées.

**Activer le mode de préproduction :** cette option permet de charger dans des pools SQL Azure Synapse Analytics à l’aide de la commande de copie, et est recommandée pour la plupart des récepteurs Synpase. Le stockage de préproduction est configuré dans [Exécuter l’activité Flux de données](control-flow-execute-data-flow-activity.md). 

- Si vous utilisez l’authentification par identité managée pour votre service lié de stockage, découvrez les configurations nécessaires pour [Azure Blob](connector-azure-blob-storage.md#managed-identity) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#managed-identity).
- Si votre Stockage Azure est configuré avec un point de terminaison de service de type réseau virtuel, vous devez utiliser l’authentification par identité managée et activer « Autoriser le service Microsoft approuvé » sur le compte de stockage. Consultez [Impact du recours à des points de terminaison de service de type réseau virtuel avec le Stockage Azure](../azure-sql/database/vnet-service-endpoint-rule-overview.md#impact-of-using-virtual-network-service-endpoints-with-azure-storage).

**Taille du lot** : contrôle le nombre de lignes écrites dans chaque compartiment. Les plus grandes tailles de lot améliorent la compression et l’optimisation de la mémoire, mais risquent de lever des exceptions de type mémoire insuffisante lors de la mise en cache des données.

**Pré et post-scripts SQL** : Entrez des scripts SQL multilignes qui s’exécutent avant (prétraitement) et après (post-traitement) l’écriture de données dans votre base de données de réception.

![Pré et post-scripts de traitement SQL](media/data-flow/prepost1.png "Scripts de traitement SQL")

### <a name="error-row-handling"></a>Gestion des lignes d’erreur

Lors de l’écriture dans Azure Synapse Analytics, certaines lignes de données peuvent échouer en raison de contraintes définies par la destination. Quelques exemples d’erreurs courantes :

*    Les données binary ou String seront tronquées dans le tableau
*    Impossible d’insérer la valeur NULL dans la colonne
*    Échec de conversion de la valeur en type de données

Par défaut, l’exécution d’un flux de données échouera à la première erreur rencontrée. Vous pouvez choisir de **Continuer en cas d’erreur**, ce qui permet à votre flux de données de se terminer, même si des lignes individuelles comportent des erreurs. Le service offre différentes options qui vous permettent de gérer ces lignes d’erreur.

**Valider une transaction :** Indiquez si vos données sont écrites dans une seule transaction ou par lots. Une transaction unique offre de meilleures performances et aucune donnée écrite n’est visible des autres utilisateurs tant que la transaction n’est pas terminée. Les transactions par lots présentent de moins bonnes performances, mais peuvent fonctionner pour des jeux de données volumineux.

**Données de sortie rejetées :** Si cette option est activée, vous pouvez générer les lignes d’erreur dans un fichier CSV dans le stockage Blob Azure ou dans un compte Azure Data Lake Storage Gen2 de votre choix. Cela écrira les lignes d’erreur avec trois colonnes supplémentaires : l’opération SQL comme INSERT ou UPDATE, le code d’erreur de flux de données et le message d’erreur sur la ligne.

**Réussite signalée malgré l’erreur :** S’il est activé, le flux de données est marqué comme ayant réussi, même si des lignes d’erreur sont détectées. 

:::image type="content" source="media/data-flow/sql-error-row-handling.png" alt-text="Capture d’écran montrant la gestion des lignes d’erreur" border="false":::

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propriétés de l’activité GetMetadata

Pour en savoir plus sur les propriétés, consultez [Activité GetMetadata](control-flow-get-metadata-activity.md).

## <a name="data-type-mapping-for-azure-synapse-analytics"></a>Mappage de type de données pour Azure Synapse Analytics

Quand vous copiez des données depuis ou vers Azure Synapse Analytics, les mappages suivants sont utilisés entre les types de données Azure Synapse Analytics et les types de données intermédiaires Azure Data Factory. Ces mappages sont également utilisés lors de la copie de données à partir de ou vers Azure Synapse Analytics à l’aide de pipelines Synapse, puisque les pipelines implémentent également Azure Data Factory au sein d’Azure Synapse. Pour découvrir comment l’activité de copie mappe le schéma et le type de données sources au récepteur, consultez [Mappages de schémas et de types de données](copy-activity-schema-and-type-mapping.md).

>[!TIP]
>Reportez-vous à l’article [Types de tables de données dans Azure Synapse Analytics](../synapse-analytics/sql/develop-tables-data-types.md) pour découvrir les types de données pris en charge par Azure Synapse Analytics et les solutions de contournement pour les types non pris en charge.

| Type de données Azure Synapse Analytics    | Type de données intermédiaires d’Azure Data Factory |
| :------------------------------------ | :----------------------------- |
| bigint                                | Int64                          |
| binary                                | Byte[]                         |
| bit                                   | Boolean                        |
| char                                  | String, Char[]                 |
| Date                                  | DateTime                       |
| Datetime                              | DateTime                       |
| datetime2                             | DateTime                       |
| Datetimeoffset                        | DateTimeOffset                 |
| Decimal                               | Decimal                        |
| FILESTREAM attribute (varbinary(max)) | Byte[]                         |
| Float                                 | Double                         |
| image                                 | Byte[]                         |
| int                                   | Int32                          |
| money                                 | Decimal                        |
| NCHAR                                 | String, Char[]                 |
| numeric                               | Decimal                        |
| NVARCHAR                              | String, Char[]                 |
| real                                  | Unique                         |
| rowversion                            | Byte[]                         |
| smalldatetime                         | DateTime                       |
| SMALLINT                              | Int16                          |
| SMALLMONEY                            | Decimal                        |
| time                                  | TimeSpan                       |
| TINYINT                               | Byte                           |
| UNIQUEIDENTIFIER                      | Guid                           |
| varbinary                             | Byte[]                         |
| varchar                               | String, Char[]                 |

## <a name="next-steps"></a>Étapes suivantes

Consultez les [formats et magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats) pour obtenir la liste des sources et magasins de données pris en charge en tant que récepteurs par l’activité de copie.
