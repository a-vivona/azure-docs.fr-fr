---
title: Copier des données à partir d’une source HTTP
titleSuffix: Azure Data Factory & Azure Synapse
description: Découvrez comment utiliser l’activité de copie (Copy) dans un pipeline Azure Data Factory ou Azure Synapse Analytics pour copier des données d’une source HTTP cloud ou locale vers des magasins de données récepteurs pris en charge.
author: jianleishen
ms.service: data-factory
ms.subservice: data-movement
ms.custom: synapse
ms.topic: conceptual
ms.date: 08/30/2021
ms.author: jianleishen
ms.openlocfilehash: 4463c3ffb8deaf0a88b978b39f3a52c83f3f8304
ms.sourcegitcommit: 851b75d0936bc7c2f8ada72834cb2d15779aeb69
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123303860"
---
# <a name="copy-data-from-an-http-endpoint-by-using-azure-data-factory-or-azure-synapse-analytics"></a>Copier des données d’un point de terminaison HTTP à l’aide d’Azure Data Factory ou d’Azure Synapse Analytics

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-http-connector.md)
> * [Version actuelle](connector-http.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité de copie (Copy) dans Azure Data Factory et Azure Synapse pour copier des données depuis un point de terminaison HTTP. Il s’appuie sur l’article [Activité de copie](copy-activity-overview.md), qui présente une vue d’ensemble de cette activité.

Les différences entre ce connecteur HTTP, le [connecteur REST](connector-rest.md) et le [connecteur Table Web](connector-web-table.md) sont les suivantes :

- Le **connecteur REST** prend spécifiquement en charge la copie de données à partir d’API RESTful. 
- Le **connecteur HTTP** est générique pour récupérer des données à partir de n’importe quel point de terminaison HTTP, par exemple pour télécharger un fichier. Avant que le connecteur REST soit disponible, vous utiliserez peut-être le connecteur HTTP pour copier des données à partir d’une API RESTful, ce qui est pris en charge mais moins fonctionnel comparé au connecteur REST.
- Le **connecteur Table web** extrait le contenu de tables d’une page web HTML.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur HTTP est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier des données d’une source HTTP vers toute banque de données réceptrice prise en charge. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Vous pouvez utiliser ce connecteur HTTP pour :

- Récupérer des données à partir d’un point de terminaison HTTP/S à l’aide des méthodes HTTP **GET** ou **POST**.
- Récupérer des données avec l’une des authentifications suivantes : **Anonymous**, **Basic**, **Digest**, **Windows** ou **ClientCertificate**.
- Copier la réponse HTTP en l’état ou l’analyser en utilisant les [formats de fichier et codecs de compression pris en charge](supported-file-formats-and-compression-codecs.md).

> [!TIP]
> Pour tester une requête HTTP d’extraction de données avant de configurer le connecteur HTTP, obtenez des informations à partir de la spécification d’API sur les exigences d’en-tête et de corps. Vous pouvez vous servir d’outils tels que Postman ou un navigateur web pour valider.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

## <a name="create-a-linked-service-to-an-http-source-using-ui"></a>Créer un service lié à une source HTTP à l’aide de l’interface utilisateur

Suivez les étapes suivantes pour créer un service lié à une source HTTP dans l'interface utilisateur du portail Azure.

1. Accédez à l’onglet Gérer dans votre espace de travail Azure Data Factory ou Synapse et sélectionnez Services liés, puis cliquez sur Nouveau :

    # <a name="azure-data-factory"></a>[Azure Data Factory](#tab/data-factory).

    :::image type="content" source="media/doc-common-process/new-linked-service.png" alt-text="Capture d’écran de la création d’un nouveau service lié avec l’interface utilisateur Azure Data Factory.":::

    # <a name="azure-synapse"></a>[Azure Synapse](#tab/synapse-analytics)

    :::image type="content" source="media/doc-common-process/new-linked-service-synapse.png" alt-text="Capture d’écran de la création d’un nouveau service lié avec l’interface utilisateur Azure Synapse.":::

2. Recherchez HTTP et sélectionnez le connecteur HTTP.

    :::image type="content" source="media/connector-http/http-connector.png" alt-text="Capture d’écran du connecteur HTTP.":::    

1. Configurez les informations du service, testez la connexion et créez le nouveau service lié.

    :::image type="content" source="media/connector-http/configure-http-linked-service.png" alt-text="Capture d’écran de la configuration d’un service lié HTTP.":::

## <a name="connector-configuration-details"></a>Informations de configuration des connecteurs

Les sections suivantes fournissent des informations détaillées sur les propriétés utilisables pour définir les entités propres au connecteur HTTP.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié HTTP sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** doit être définie sur **HttpServer**. | Oui |
| url | URL de base du serveur web. | Oui |
| enableServerCertificateValidation | Indique si la validation du certificat TLS/SSL de serveur est activée lors de la connexion à un point de terminaison HTTP. Si votre serveur HTTPS utilise un certificat auto-signé, affectez à cette propriété la valeur **false**. | Non<br /> (la valeur par défaut est **true**) |
| authenticationType | Spécifie le type d’authentification. Les valeurs autorisées sont : **Anonyme**, **De base**, **Digest**, **Windows** et **ClientCertificate**. L'authentification OAuth par utilisateur n'est pas prise en charge. Vous pouvez également configurer des en-têtes d’authentification dans la propriété `authHeader`. Consultez les sections à la suite de ce tableau pour accéder à d’autres propriétés et à des exemples JSON relatifs à ces types d’authentification. | Oui |
| authHeaders | En-têtes de requête HTTP supplémentaires pour l’authentification.<br/> Par exemple, pour utiliser l’authentification par clé API, vous pouvez sélectionner le type d’authentification « anonyme » et spécifier la clé API dans l’en-tête. | Non |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion au magasin de données. Pour plus d’informations, consultez la section [Conditions préalables](#prerequisites). À défaut de spécification, l’Azure Integration Runtime par défaut est utilisé. |Non |

### <a name="using-basic-digest-or-windows-authentication"></a>Utilisation de l’authentification Basic (De base), Digest ou Windows

Définissez la valeur de la propriété **authenticationType** sur **De base**, **Digest** ou **Windows**. Outre les propriétés génériques décrites dans la section précédente, spécifiez les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| userName | Nom d’utilisateur à utiliser pour accéder au point de terminaison HTTP. | Oui |
| mot de passe | Mot de passe de l’utilisateur (valeur **userName**). Vous pouvez marquer ce champ en tant que type **SecureString** pour le stocker de manière sécurisée. Vous pouvez également [référencer un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |

**Exemple**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "Basic",
            "url" : "<HTTP endpoint>",
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

### <a name="using-clientcertificate-authentication"></a>Utilisation de l’authentification ClientCertificate (Certificat client)

Pour utiliser l’authentification ClientCertificate, définissez la valeur de la propriété **authenticationType** sur **ClientCertificate**. Outre les propriétés génériques décrites dans la section précédente, spécifiez les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| embeddedCertData | Données du certificat encodé en Base64. | Spécifiez **embeddedCertData** ou **certThumbprint**. |
| certThumbprint | Empreinte numérique du certificat installé dans le magasin de certificats de votre ordinateur exécutant le runtime d’intégration auto-hébergé. S’applique uniquement quand le type auto-hébergé du runtime d’intégration est spécifié dans la propriété **connectVia**. | Spécifiez **embeddedCertData** ou **certThumbprint**. |
| mot de passe | Mot de passe associé au certificat. Vous pouvez marquer ce champ en tant que type **SecureString** pour le stocker de manière sécurisée. Vous pouvez également [référencer un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Non |

Si vous utilisez **certThumbprint** pour l’authentification et que le certificat est installé dans le magasin personnel de l’ordinateur local, accordez des autorisations de lecture au runtime d’intégration auto-hébergé :

1. Ouvrez MMC (Microsoft Management Console). Ajoutez le composant logiciel enfichable **Certificats** ciblant **Ordinateur local**.
2. Développez **Certificats** > **Personnel**, puis sélectionnez **Certificats**.
3. Cliquez avec le bouton droit sur le certificat du magasin personnel, puis sélectionnez **Toutes les tâches** > **Gérer les clés privées**.
3. Sous l’onglet **Sécurité**, ajoutez le compte d’utilisateur sous lequel le service hôte du runtime d’intégration (DIAHostService) s’exécute avec l’accès en lecture au certificat.

**Exemple 1 : utilisation de certThumbprint**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "<HTTP endpoint>",
            "certThumbprint": "<thumbprint of certificate>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Exemple 2 : utilisation d’embeddedCertData**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "<HTTP endpoint>",
            "embeddedCertData": "<Base64-encoded cert data>",
            "password": {
                "type": "SecureString",
                "value": "password of cert"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="using-authentication-headers"></a>Utilisation d’en-têtes d’authentification

En outre, vous pouvez configurer des en-têtes de demande pour l’authentification en plus des types d’authentification intégrés.

**Exemple : Utilisation de l’authentification par clé API**

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "HttpServer",
        "typeProperties": {
            "url": "<HTTP endpoint>",
            "authenticationType": "Anonymous",
            "authHeader": {
                "x-api-key": {
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

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour HTTP sous les paramètres `location` dans le jeu de données basé sur le format :

| Propriété    | Description                                                  | Obligatoire |
| ----------- | ------------------------------------------------------------ | -------- |
| type        | La propriété de type sous `location` dans le jeu de données doit être définie sur **HttpServerLocation**. | Oui      |
| relativeUrl | URL relative de la ressource qui contient les données. Le connecteur HTTP copie les données à partir de l’URL combinée : `[URL specified in linked service][relative URL specified in dataset]`.   | Non       |

> [!NOTE]
> La taille de charge utile de requête HTTP prise en charge est d’environ 500 Ko. Si la taille de charge utile que vous souhaitez passer au point de terminaison web est supérieure à 500 Ko, envisagez de la traiter en blocs plus petits.

**Exemple :**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "HttpServerLocation",
                "relativeUrl": "<relative url>"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Cette section contient la liste des propriétés prises en charge par la source HTTP.

Pour obtenir la liste complète des sections et des propriétés permettant de définir des activités, consultez [Pipelines](concepts-pipelines-activities.md). 

### <a name="http-as-source"></a>HTTP en tant que source

[!INCLUDE [data-factory-v2-file-formats](includes/data-factory-v2-file-formats.md)] 

Les propriétés suivantes sont prises en charge pour HTTP sous les paramètres `storeSettings` dans la source de la copie basée sur le format :

| Propriété                 | Description                                                  | Obligatoire |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | La propriété type sous `storeSettings` doit être définie sur **HttpReadSettings**. | Oui      |
| requestMethod            | Méthode HTTP. <br>Les valeurs autorisées sont **Get** (par défaut) et **Post**. | Non       |
| additionalHeaders         | En-têtes de requête HTTP supplémentaires.                             | Non       |
| requestBody              | Corps de la requête HTTP.                               | Non       |
| httpRequestTimeout           | Délai d’expiration (valeur **TimeSpan**) pour l’obtention d’une réponse par la requête HTTP. Cette valeur correspond au délai d’expiration pour l’obtention d’une réponse, et non au délai d’expiration pour la lecture des données de la réponse. La valeur par défaut est **00:01:40**. | Non       |
| maxConcurrentConnections |La limite supérieure de connexions simultanées établies au magasin de données pendant l’exécution de l’activité. Spécifiez une valeur uniquement lorsque vous souhaitez limiter les connexions simultanées.| Non       |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromHTTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
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
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "HttpReadSettings",
                    "requestMethod": "Post",
                    "additionalHeaders": "<header key: header value>\n<header key: header value>\n",
                    "requestBody": "<body for POST HTTP request>"
                }
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

## <a name="legacy-models"></a>Modèles hérités

>[!NOTE]
>Les Modèles suivants sont toujours pris en charge tels quels à des fins de compatibilité descendante. Il est recommandé d’utiliser le nouveau modèle mentionné dans les sections ci-dessus à partir de maintenant. L’interface utilisateur de création peut désormais générer ce nouveau modèle.

### <a name="legacy-dataset-model"></a>Modèle de jeu de données hérité

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** du jeu de données doit être définie sur **HttpFile**. | Oui |
| relativeUrl | URL relative de la ressource qui contient les données. Quand cette propriété n’est pas spécifiée, seule l’URL indiquée dans la définition du service lié est utilisée. | Non |
| requestMethod | Méthode HTTP. Les valeurs autorisées sont **Get** (par défaut) et **Post**. | Non |
| additionalHeaders | En-têtes de requête HTTP supplémentaires. | Non |
| requestBody | Corps de la requête HTTP. | Non |
| format | Si vous souhaitez récupérer des données du point de terminaison HTTP en l’état sans les analyser, puis les copier dans un magasin basé sur un fichier, ignorez la section **format** dans les définitions de jeu de données d’entrée et de sortie.<br/><br/>Si vous souhaitez analyser le contenu de la réponse HTTP pendant la copie, les types de formats de fichiers suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Sous **format**, définissez la propriété **type** sur l’une de ces valeurs. Pour plus d’informations, consultez [Format JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [Format Texte](supported-file-formats-and-compression-codecs-legacy.md#text-format), [Format Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [Format Orc](supported-file-formats-and-compression-codecs-legacy.md#orc-format) et [Format Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Pour plus d’informations, voir [Formats de fichier et de codecs de compression pris en charge](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/><br/>Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**.<br/>Niveaux pris en charge :  **Optimal** et **Fastest**. |Non |

> [!NOTE]
> La taille de charge utile de requête HTTP prise en charge est d’environ 500 Ko. Si la taille de charge utile que vous souhaitez passer au point de terminaison web est supérieure à 500 Ko, envisagez de la traiter en blocs plus petits.

**Exemple 1 : utilisation de la méthode Get (par défaut)**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "HttpFile",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        }
    }
}
```

**Exemple 2 : Utilisation de la méthode Post**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "HttpFile",
        "linkedServiceName": {
            "referenceName": "<HTTP linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "relativeUrl": "<relative url>",
            "requestMethod": "Post",
            "requestBody": "<body for POST HTTP request>"
        }
    }
}
```

### <a name="legacy-copy-activity-source-model"></a>Modèle hérité de la source d’activité de copie

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** de la source de l’activité de copie doit être définie sur **HttpSource**. | Oui |
| httpRequestTimeout | Délai d’expiration (valeur **TimeSpan**) pour l’obtention d’une réponse par la requête HTTP. Cette valeur correspond au délai d’expiration pour l’obtention d’une réponse, et non au délai d’expiration pour la lecture des données de la réponse. La valeur par défaut est **00:01:40**.  | Non |

**Exemple**

```json
"activities":[
    {
        "name": "CopyFromHTTP",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<HTTP input dataset name>",
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
                "type": "HttpSource",
                "httpRequestTimeout": "00:01:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).
