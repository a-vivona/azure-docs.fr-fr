---
title: Copier des données de sources OData à l’aide d’Azure Data Factory
titleSuffix: Azure Data Factory & Azure Synapse
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données de sources OData vers des banques de données réceptrices prises en charge.
author: jianleishen
ms.service: data-factory
ms.subservice: data-movement
ms.custom: synapse
ms.topic: conceptual
ms.date: 08/30/2021
ms.author: jianleishen
ms.openlocfilehash: b08493b8b055cc013fa073dbbc7c58c5c936f22a
ms.sourcegitcommit: 851b75d0936bc7c2f8ada72834cb2d15779aeb69
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123317429"
---
# <a name="copy-data-from-an-odata-source-by-using-azure-data-factory"></a>Copier des données d’une source OData à l’aide d’Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-odata-connector.md)
> * [Version actuelle](connector-odata.md)

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour copier les données d’une source OData. Il s’appuie sur l’article [Activité de copie dans Azure Data Factory](copy-activity-overview.md), qui constitue une présentation de l’activité de copie.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur OData est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier les données d’une source OData dans tout magasin de données récepteur pris en charge. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur OData prend en charge ce qui suit :

- OData version 3.0 et 4.0.
- Copie de données avec une des authentifications suivantes : **Anonyme**, **De base**, **Windows** et **Principal de service AAD**.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

## <a name="create-a-linked-service-to-an-odata-store-using-ui"></a>Créer un service lié à un magasin OData à l’aide de l’interface utilisateur

Utilisez les étapes suivantes pour créer un service lié à un magasin OData dans l’interface utilisateur du portail Azure.

1. Accédez à l’onglet Gérer dans votre espace de travail Azure Data Factory ou Synapse et sélectionnez Services liés, puis cliquez sur Nouveau :

    # <a name="azure-data-factory"></a>[Azure Data Factory](#tab/data-factory).

    :::image type="content" source="media/doc-common-process/new-linked-service.png" alt-text="Capture d’écran de la création d’un nouveau service lié avec l’interface utilisateur Azure Data Factory.":::

    # <a name="azure-synapse"></a>[Azure Synapse](#tab/synapse-analytics)

    :::image type="content" source="media/doc-common-process/new-linked-service-synapse.png" alt-text="Capture d’écran de la création d’un nouveau service lié avec l’interface utilisateur Azure Synapse.":::

2. Recherchez OData et sélectionnez le connecteur OData.

    :::image type="content" source="media/connector-odata/odata-connector.png" alt-text="Capture d’écran du connecteur OData.":::    

1. Configurez les informations du service, testez la connexion et créez le nouveau service lié.

    :::image type="content" source="media/connector-odata/configure-odata-linked-service.png" alt-text="Capture d’écran de la configuration du service lié pour un magasin OData.":::

## <a name="connector-configuration-details"></a>Informations de configuration des connecteurs

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory propres au connecteur OData.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié OData sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** doit être définie sur **OData**. |Oui |
| url | URL racine du service OData. |Oui |
| authenticationType | Type d’authentification utilisé pour se connecter à la source OData. Les valeurs autorisées sont les suivantes : **Anonyme**, **De base**, **Windows** et **Principal de service AAD**. L'authentification OAuth par utilisateur n'est pas prise en charge. Vous pouvez également configurer des en-têtes d’authentification dans la propriété `authHeader`.| Oui |
| authHeaders | En-têtes de requête HTTP supplémentaires pour l’authentification.<br/> Par exemple, pour utiliser l’authentification par clé API, vous pouvez sélectionner le type d’authentification « anonyme » et spécifier la clé API dans l’en-tête. | Non |
| userName | Si vous utilisez l’authentification De base ou Windows, spécifiez un **nom d’utilisateur**. | Non |
| mot de passe | Spécifiez le **mot de passe** associé au **nom d’utilisateur spécifié**. Vous pouvez marquer ce champ en tant que type **SecureString** pour le stocker de manière sécurisée dans Data Factory. Vous pouvez également [référencer un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Non |
| servicePrincipalId | Spécifiez l’ID de l’application Azure Active Directory. | Non |
| aadServicePrincipalCredentialType | Spécifiez le type d’informations d’identification à utiliser pour l’authentification de principal du service. Valeurs autorisées : `ServicePrincipalKey` ou `ServicePrincipalCert`. | Non |
| servicePrincipalKey | Spécifiez la clé de l’application Azure Active Directory. Marquez ce champ en tant que **SecureString** afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Non |
| servicePrincipalEmbeddedCert | Spécifiez le certificat codé en base64 de l’application inscrite dans Azure Active Directory. Marquez ce champ en tant que **SecureString** afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Non |
| servicePrincipalEmbeddedCertPassword | Spécifiez le mot de passe de votre certificat si votre certificat est sécurisé par un mot de passe. Marquez ce champ en tant que **SecureString** afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md).  | Non|
| tenant | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Récupérez-le en pointant la souris dans le coin supérieur droit du Portail Azure. | Non |
| aadResourceId | Spécifiez la ressource AAD pour laquelle vous demandez une autorisation.| Non |
| azureCloudType | Pour l’authentification du principal du service, spécifiez le type d’environnement cloud Azure auquel votre application AAD est inscrite. <br/> Les valeurs autorisées sont **AzurePublic**, **AzureChina**, **AzureUsGovernment** et **AzureGermany**. Par défaut, l’environnement cloud de la fabrique de données est utilisé. | Non |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion au magasin de données. Pour plus d’informations, consultez la section [Conditions préalables](#prerequisites). À défaut de spécification, l’Azure Integration Runtime par défaut est utilisé. |Non |

**Exemple 1 : Utilisation de l’authentification anonyme**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "https://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple 2 : Utilisation de l’authentification de base**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "Basic",
            "userName": "<user name>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple 3 : Utilisation de l’authentification Windows**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "Windows",
            "userName": "<domain>\\<user>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple 4 : Utilisation de l’authentification de la clé du principal de service**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "AadServicePrincipal",
            "servicePrincipalId": "<service principal id>",
            "aadServicePrincipalCredentialType": "ServicePrincipalKey",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource URL>"
        }
    },
    "connectVia": {
        "referenceName": "<name of Integration Runtime>",
        "type": "IntegrationRuntimeReference"
    }
}
```

**Exemple 5 : Utilisation de l’authentification du certificat du principal de service**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "AadServicePrincipal",
            "servicePrincipalId": "<service principal id>",
            "aadServicePrincipalCredentialType": "ServicePrincipalCert",
            "servicePrincipalEmbeddedCert": { 
                "type": "SecureString", 
                "value": "<base64 encoded string of (.pfx) certificate data>"
            },
            "servicePrincipalEmbeddedCertPassword": { 
                "type": "SecureString", 
                "value": "<password of your certificate>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource e.g. https://tenant.sharepoint.com>"
        }
    },
    "connectVia": {
        "referenceName": "<name of Integration Runtime>",
        "type": "IntegrationRuntimeReference"
    }
}
```

**Exemple 6 : Utilisation de l’authentification avec une clé API**

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of OData source>",
            "authenticationType": "Anonymous",
            "authHeader": {
                "APIKey": {
                    "type": "SecureString",
                    "value": "<API key>"
                }
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Cette section contient la liste des propriétés prises en charge par le jeu de données OData.

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez [Jeux de données et services liés](concepts-datasets-linked-services.md). 

Pour copier des données à partir d’OData, définissez la propriété **type** du jeu de données sur **ODataResource**. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** du jeu de données doit être définie sur **ODataResource**. | Oui |
| path | Chemin de la ressource OData. | Oui |

**Exemple**

```json
{
    "name": "ODataDataset",
    "properties":
    {
        "type": "ODataResource",
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<OData linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties":
        {
            "path": "Products"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Cette section contient la liste des propriétés prises en charge par la source OData.

Pour obtenir la liste complète des sections et des propriétés permettant de définir des activités, consultez [Pipelines](concepts-pipelines-activities.md). 

### <a name="odata-as-source"></a>OData en tant que source

Pour copier des données à partir d’OData, les propriétés prises en charge dans la section **source** de l'activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété de **type** de la source d’activité de copie doit être définie sur **ODataSource**. | Oui |
| query | Options de requête OData pour filtrer les données. Exemple : `"$select=Name,Description&$top=5"`.<br/><br/>**Remarque** : Le connecteur OData copie des données à partir de l’URL combinée `[URL specified in linked service]/[path specified in dataset]?[query specified in copy activity source]`. Pour plus d’informations, consultez [OData URL components](https://www.odata.org/documentation/odata-version-3-0/url-conventions/). | Non |
| httpRequestTimeout | Délai d’expiration (valeur **TimeSpan**) pour l’obtention d’une réponse par la requête HTTP. Cette valeur correspond au délai d’expiration pour l’obtention d’une réponse, et non au délai d’expiration pour la lecture des données de la réponse. Si elle n’est pas spécifiée, la valeur par défaut est **00:30:00** (30 minutes). | Non |

**Exemple**

```json
"activities":[
    {
        "name": "CopyFromOData",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<OData input dataset name>",
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
                "type": "ODataSource",
                "query": "$select=Name,Description&$top=5"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Si vous utilisiez une source de données typée `RelationalSource`, elle reste prise en charge telle quelle, mais nous vous suggérons d’utiliser désormais la nouvelle source.

## <a name="data-type-mapping-for-odata"></a>Mappage de type de données pour OData

Lorsque vous copiez des données à partir d’OData, les mappages suivants sont utilisés entre les types de données OData et les types de données intermédiaires d’Azure Data Factory. Pour découvrir comment l’activité de copie mappe le schéma et le type de données sources au récepteur, consultez [Mappages de schémas et de types de données](copy-activity-schema-and-type-mapping.md).

| Type de données OData | Type de données intermédiaires d’Azure Data Factory |
|:--- |:--- |
| Edm.Binary | Byte[] |
| Edm.Boolean | Bool |
| Edm.Byte | Byte[] |
| Edm.DateTime | DateTime |
| Edm.Decimal | Decimal |
| Edm.Double | Double |
| Edm.Single | Unique |
| Edm.Guid | Guid |
| Edm.Int16 | Int16 |
| Edm.Int32 | Int32 |
| Edm.Int64 | Int64 |
| Edm.SByte | Int16 |
| Edm.String | String |
| Edm.Time | TimeSpan |
| Edm.DateTimeOffset | DateTimeOffset |

> [!NOTE]
> Les types de données complexes OData (par exemple, **Object**), ne sont pas pris en charge.

## <a name="copy-data-from-project-online"></a>Copier des données à partir de Project Online

Pour copier des données à partir de Project Online, vous pouvez utiliser le connecteur OData et un jeton d’accès obtenu à partir d’outils tels que Postman.

> [!CAUTION]
> Le jeton d’accès expire dans une heure par défaut : vous devez obtenir un nouveau jeton d’accès lorsqu’il expire.

1. Utilisez **Postman** pour recevoir le jeton d’accès :

   1. Accédez à l’onglet **Authorization** (Autorisation) sur le site web de Postman.
   1. Dans la zone **Type**, sélectionnez **OAuth 2.0**, puis, dans la zone **Add authorization data to** (Ajouter des données d’autorisation à), sélectionnez **Request Headers** (En-têtes de demande).
   1. Renseignez les informations suivantes dans la page **Configure New Token** (Configurer un nouveau jeton) pour obtenir un nouveau jeton d’accès : 
      - **Grant type** (Type d’autorisation) : Sélectionnez **Authorization Code** (Code d’autorisation).
      - **Callback URL** (URL de rappel) : Entrez `https://www.localhost.com/`. 
      - **Auth URL** (URL d’authentification) : Entrez `https://login.microsoftonline.com/common/oauth2/authorize?resource=https://<your tenant name>.sharepoint.com`. Remplacez `<your tenant name>` par votre propre nom de locataire. 
      - **Access Token URL** (URL de jeton d’accès) : Entrez `https://login.microsoftonline.com/common/oauth2/token`.
      - **Client ID** (ID client) : Entrez l’ID de votre principal de service AAD.
      - **Client Secret** (Clé secrète client) : Entrez le secret de votre principal de service.
      - **Client Authentication** (Authentification du client) : Sélectionnez **Send as Basic Auth header** (Envoyer en tant qu’en-tête d’authentification de base).
     
   1. Vous serez invité à vous connecter avec votre nom d’utilisateur et votre mot de passe.
   1. Une fois que vous avez obtenu votre jeton d’accès, copiez-le et enregistrez-le pour l’étape suivante.
   
    [![Utiliser Postman pour recevoir le jeton d’accès](./media/connector-odata/odata-project-online-postman-access-token-inline.png)](./media/connector-odata/odata-project-online-postman-access-token-expanded.png#lightbox)

1. Créez le service lié OData :
    - **Service URL** (URL du service) : Entrez `https://<your tenant name>.sharepoint.com/sites/pwa/_api/Projectdata`. Remplacez `<your tenant name>` par votre propre nom de locataire. 
    - **Authentication type** (Type d’authentification) : Sélectionnez **Anonymous** (Anonyme).
    - **Auth headers** (En-têtes d’authentification) :
        - **Property name** (Nom de propriété) : Choisissez **Authorization** (Autorisation).
        - **Valeur** : Entrez `Bearer <access token from step 1>`.
    - Testez le service lié.

    ![Créer le service lié OData](./media/connector-odata/odata-project-online-linked-service.png)

1. Créez le jeu de données OData :
    1. Créez le jeu de données avec le service lié OData créé à l’étape 2.
    1. Prévisualisez les données.
 
    ![Aperçu des données](./media/connector-odata/odata-project-online-preview-data.png)
 


## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).