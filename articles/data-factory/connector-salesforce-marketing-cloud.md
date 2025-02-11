---
title: Copier des données à partir de Salesforce Marketing Cloud
titleSuffix: Azure Data Factory & Azure Synapse
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données de Salesforce Marketing Cloud vers des magasins de données récepteurs pris en charge.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.subservice: data-movement
ms.topic: conceptual
ms.custom: synapse
ms.date: 08/30/2021
ms.openlocfilehash: 448ef9ebc2147fa23716bda6441a0196c0c003b3
ms.sourcegitcommit: 851b75d0936bc7c2f8ada72834cb2d15779aeb69
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123307925"
---
# <a name="copy-data-from-salesforce-marketing-cloud-using-azure-data-factory"></a>Copier des données de Salesforce Marketing Cloud avec Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour copier des données de Salesforce Marketing Cloud. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Salesforce Marketing Cloud est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Il est possible de copier des données de Salesforce Marketing Cloud vers n’importe quel magasin de données récepteur pris en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Le connecteur Salesforce Marketing Cloud prend en charge l’authentification OAuth 2 ainsi que les types de packages hérités et améliorés. Le connecteur repose sur l’[API REST Salesforce Marketing Cloud](https://developer.salesforce.com/docs/atlas.en-us.mc-apis.meta/mc-apis/index-api.htm).

>[!NOTE]
>Ce connecteur ne prend pas en charge la récupération d'objets personnalisés ou d'extensions de données personnalisées.

## <a name="getting-started"></a>Prise en main

Vous pouvez créer un pipeline avec l’activité de copie à l’aide du SDK .NET, du SDK Python, d’Azure PowerShell, de l’API REST ou du modèle Azure Resource Manager. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](quickstart-create-data-factory-dot-net.md).

## <a name="create-a-linked-service-to-salesforce-marketing-cloud-using-ui"></a>Créer un service lié à Salesforce Marketing Cloud à l’aide de l’interface utilisateur

Utilisez les étapes suivantes pour créer un service lié à Salesforce Marketing Cloud dans l’interface utilisateur du portail Azure.

1. Accédez à l’onglet Gérer dans votre espace de travail Azure Data Factory ou Synapse et sélectionnez Services liés, puis cliquez sur Nouveau :

    # <a name="azure-data-factory"></a>[Azure Data Factory](#tab/data-factory).

    :::image type="content" source="media/doc-common-process/new-linked-service.png" alt-text="Créez un nouveau service lié avec l’interface utilisateur Azure Data Factory.":::

    # <a name="azure-synapse"></a>[Azure Synapse](#tab/synapse-analytics)

    :::image type="content" source="media/doc-common-process/new-linked-service-synapse.png" alt-text="Créez un nouveau service lié avec l’interface utilisateur Azure Synapse.":::

2. Recherchez Salesforce et sélectionnez le connecteur Salesforce Marketing Cloud.

   :::image type="content" source="media/connector-salesforce-marketing-cloud/salesforce-marketing-cloud-connector.png" alt-text="Sélectionnez le connecteur Salesforce Marketing Cloud.":::    


1. Configurez les informations du service, testez la connexion et créez le nouveau service lié.

   :::image type="content" source="media/connector-salesforce-marketing-cloud/configure-salesforce-marketing-cloud-linked-service.png" alt-text="Configurez un service lié à Salesforce Marketing Cloud.":::

## <a name="connector-configuration-details"></a>Informations de configuration du connecteur

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir des entités Data Factory propres au connecteur Salesforce Marketing Cloud.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Salesforce Marketing Cloud sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **SalesforceMarketingCloud** | Oui |
| connectionProperties | Groupe de propriétés qui définit la façon de se connecter à Salesforce Marketing Cloud. | Oui |
| ***Sous `connectionProperties`:*** | | |
| authenticationType | spécifie la méthode d’authentification à utiliser. Valeurs autorisées : `Enhanced sts OAuth 2.0` ou `OAuth_2.0`.<br><br>Le package hérité Salesforce Marketing Cloud prend uniquement en charge `OAuth_2.0`, tandis que le package amélioré nécessite `Enhanced sts OAuth 2.0`. <br>Depuis le 1er août 2019, Salesforce Marketing Cloud a supprimé la possibilité de créer des packages hérités. Tous les nouveaux packages sont des packages améliorés. | Oui |
| host | Pour le package amélioré, l’hôte doit être votre [sous-domaine](https://developer.salesforce.com/docs/atlas.en-us.mc-apis.meta/mc-apis/your-subdomain-tenant-specific-endpoints.htm) qui est représenté par une chaîne de 28 caractères commençant par les lettres « mc », par exemple `mc563885gzs27c5t9-63k636ttgm`. <br>Pour le package hérité, spécifiez `www.exacttargetapis.com`. | Oui |
| clientId | ID client associé à l’application Salesforce Marketing Cloud.  | Oui |
| clientSecret | Clé secrète client associée à l’application Salesforce Marketing Cloud. Vous pouvez choisir de marquer ce champ comme SecureString pour le stocker en toute sécurité dans le fichier de définition d’application, ou stocker le secret dans Azure Key Vault et laisser l’activité de copie ADF en tirer (pull) les données lors de la copie. Pour plus d’informations, consultez la page [Stocker des informations d’identification dans Key Vault](store-credentials-in-key-vault.md). | Oui |
| useEncryptedEndpoints | Indique si les points de terminaison de la source de données sont chiffrés suivant le protocole HTTPS. La valeur par défaut est true.  | Non |
| useHostVerification | Indique si le nom d’hôte du certificat du serveur doit correspondre à celui du serveur en cas de connexion TLS. La valeur par défaut est true.  | Non |
| usePeerVerification | Indique s’il faut vérifier l’identité du serveur en cas de connexion TLS. La valeur par défaut est true.  | Non |

**Exemple : utilisation de l’authentification OAuth 2 du service STS amélioré pour le package amélioré** 

```json
{
    "name": "SalesforceMarketingCloudLinkedService",
    "properties": {
        "type": "SalesforceMarketingCloud",
        "typeProperties": {
            "connectionProperties": {
                "host": "<subdomain e.g. mc563885gzs27c5t9-63k636ttgm>",
                "authenticationType": "Enhanced sts OAuth 2.0",
                "clientId": "<clientId>",
                "clientSecret": {
                     "type": "SecureString",
                     "value": "<clientSecret>"
                },
                "useEncryptedEndpoints": true,
                "useHostVerification": true,
                "usePeerVerification": true
            }
        }
    }
}

```

**Exemple : utilisation de l’authentification OAuth 2 pour le package hérité** 

```json
{
    "name": "SalesforceMarketingCloudLinkedService",
    "properties": {
        "type": "SalesforceMarketingCloud",
        "typeProperties": {
            "connectionProperties": {
                "host": "www.exacttargetapis.com",
                "authenticationType": "OAuth_2.0",
                "clientId": "<clientId>",
                "clientSecret": {
                     "type": "SecureString",
                     "value": "<clientSecret>"
                },
                "useEncryptedEndpoints": true,
                "useHostVerification": true,
                "usePeerVerification": true
            }
        }
    }
}

```

Si vous utilisiez le service lié Salesforce Marketing Cloud avec la charge utile suivante, il reste pris en charge tel quel, mais nous vous suggérons d’utiliser le nouveau à l’avenir, qui ajoute la prise en charge du package amélioré.

```json
{
    "name": "SalesforceMarketingCloudLinkedService",
    "properties": {
        "type": "SalesforceMarketingCloud",
        "typeProperties": {
            "clientId": "<clientId>",
            "clientSecret": {
                 "type": "SecureString",
                 "value": "<clientSecret>"
            },
            "useEncryptedEndpoints": true,
            "useHostVerification": true,
            "usePeerVerification": true
        }
    }
}

```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les [jeux de données](concepts-datasets-linked-services.md). Cette section donne la liste des propriétés prises en charge par le jeu de données Salesforce Marketing Cloud.

Pour copier des données à partir de Salesforce Marketing Cloud, affectez la valeur **SalesforceMarketingCloudObject** à la propriété type du jeu de données. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur : **SalesforceMarketingCloudObject** | Oui |
| tableName | Nom de la table. | Non (si « query » dans la source de l’activité est spécifié) |

**Exemple**

```json
{
    "name": "SalesforceMarketingCloudDataset",
    "properties": {
        "type": "SalesforceMarketingCloudObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SalesforceMarketingCloud linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section donne la liste des propriétés prises en charge par la source Salesforce Marketing Cloud.

### <a name="salesforce-marketing-cloud-as-source"></a>Salesforce Marketing Cloud en tant que source

Pour copier des données à partir de Salesforce Marketing Cloud, affectez la valeur **SalesforceMarketingCloudSource** au type de source dans l’activité de copie. Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source d’activité de copie doit être définie sur : **SalesforceMarketingCloudSource** | Oui |
| query | Utiliser la requête SQL personnalisée pour lire les données. Par exemple : `"SELECT * FROM MyTable"`. | Non (si « tableName » est spécifié dans dataset) |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromSalesforceMarketingCloud",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SalesforceMarketingCloud input dataset name>",
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
                "type": "SalesforceMarketingCloudSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).
