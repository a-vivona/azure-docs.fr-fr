---
title: Mapper les champs de sortie de compétence
titleSuffix: Azure Cognitive Search
description: Exportez le contenu enrichi créé par un ensemble de compétences en mappant ses champs de sortie aux champs d’un index de recherche.
author: LiamCavanagh
ms.author: liamca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/10/2021
ms.openlocfilehash: 0e1db5f12e83a88697db38a6ddd77edab445f6c0
ms.sourcegitcommit: 86ca8301fdd00ff300e87f04126b636bae62ca8a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/16/2021
ms.locfileid: "122533368"
---
# <a name="map-enrichment-output-to-fields-in-a-search-index"></a>Mapper la sortie d’enrichissement aux champs d’un index de recherche

![Étapes de l'indexeur](./media/cognitive-search-output-field-mapping/indexer-stages-output-field-mapping.png "étapes de l'indexeur")

Dans cet article, vous allez apprendre à mapper des champs d’entrée enrichis sur des champs de sortie dans un index pouvant faire l’objet d’une recherche. Une fois que vous avez [défini un ensemble de compétences](cognitive-search-defining-skillset.md), vous devez mapper les champs de sortie de n’importe quelle compétence qui fournit directement des valeurs sur un champ donné dans votre index de recherche.

Les mappages de champs de sortie sont nécessaires au déplacement de contenu de documents enrichis vers l’index.  Le document enrichi est en fait une arborescence d’informations, et même si l’index prend en charge les types complexes, vous pouvez parfois souhaiter transformer les informations de l’arborescence enrichie en un type plus simple (par exemple, un tableau de chaînes). Les mappages de champs de sortie vous permettent d’effectuer des transformations de formes de données en aplatissant les informations. Les mappages de champs de sortie se produisent toujours après l’exécution d’un ensemble de compétences, même s’il est possible d’exécuter cette étape sans aucun ensemble de compétences défini.

Exemples de mappages de champs de sortie :

* Dans le cadre de votre ensemble de compétences, vous avez extrait les noms des organisations mentionnées dans chacune des pages de votre document. Vous souhaitez maintenant mapper chacun de ces noms d’organisation dans un champ de votre index de type Edm.Collection(Edm.String).

* Dans le cadre de votre ensemble de compétences, vous avez créé un nouveau nœud appelé « document/translated_text ». Vous souhaitez mapper les informations de ce nœud à un champ spécifique dans votre index.

* Vous n’avez pas d’ensemble de compétences, mais vous indexez un type complexe à partir d’une base de données Cosmos DB. Vous aimeriez accéder à un nœud sur ce type complexe et le mapper à un champ de votre index.

> [!NOTE]
> Les mappages de champs de sortie s’appliquent uniquement aux index de recherche. Pour les indexeurs qui créent des [bases de connaissances](knowledge-store-concept-intro.md), les mappages de champs de sortie sont ignorés.

## <a name="use-outputfieldmappings"></a>Utiliser outputFieldMappings

Pour mapper les champs, ajoutez `outputFieldMappings` à la définition de l’indexeur comme indiqué ci-dessous :

```http
PUT https://[servicename].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
api-key: [admin key]
Content-Type: application/json
```

Le corps de la demande est structuré comme suit :

```json
{
    "name": "myIndexer",
    "dataSourceName": "myDataSource",
    "targetIndexName": "myIndex",
    "skillsetName": "myFirstSkillSet",
    "fieldMappings": [
        {
            "sourceFieldName": "metadata_storage_path",
            "targetFieldName": "id",
            "mappingFunction": {
                "name": "base64Encode"
            }
        }
    ],
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/content/organizations/*/description",
            "targetFieldName": "descriptions",
            "mappingFunction": {
                "name": "base64Decode"
            }
        },
        {
            "sourceFieldName": "/document/content/organizations",
            "targetFieldName": "orgNames"
        },
        {
            "sourceFieldName": "/document/content/sentiment",
            "targetFieldName": "sentiment"
        }
    ]
}
```

Pour chaque mappage de champ de sortie, définissez l’emplacement des données dans l’arborescence de documents enrichis (sourceFieldName) ainsi que le nom du champ tel que référencé dans l’index (targetFieldName). Affectez les [fonctions de mappage](search-indexer-field-mappings.md#field-mapping-functions) nécessaires pour transformer le contenu d’un champ avant qu’il ne soit stocké dans l’index.

## <a name="flattening-information-from-complex-types"></a>Aplatissement d’informations à partir de types complexes 

Le chemin dans un sourceFieldName peut représenter un élément ou plusieurs éléments. Dans l’exemple ci-dessus, ```/document/content/sentiment``` représente une valeur numérique unique, tandis que ```/document/content/organizations/*/description``` représente plusieurs descriptions d’organisation. 

Dans les cas où il existe plusieurs éléments, ceux-ci sont « aplatis » en un tableau qui contient chacun des éléments. 

Plus concrètement, si nous considérons l’exemple ```/document/content/organizations/*/description```, les données du champ *descriptions* ressembleraient à un tableau de descriptions aplati avant d’être indexées :

```
 ["Microsoft is a company in Seattle","LinkedIn's office is in San Francisco"]
```

S’agissant d’un principe important, nous allons examiner un autre exemple. Imaginez que vous disposez d’un tableau de types complexes parmi l’arborescence d’enrichissement. Supposons qu’un membre appelé customEntities dispose d’un tableau de types complexes comme celui décrit ci-dessous.

```json
"document/customEntities": 
[
    {
        "name": "heart failure",
        "matches": [
            {
                "text": "heart failure",
                "offset": 10,
                "length": 12,
                "matchDistance": 0.0
            }
        ]
    },
    {
        "name": "morquio",
        "matches": [
            {
                "text": "morquio",
                "offset": 25,
                "length": 7,
                "matchDistance": 0.0
            }
        ]
    }
    //...
]
```

Par ailleurs, vous possédez une index qui comporte un champ appelé « diseases » de type Collection(Edm.String) dans lequel vous souhaitez stocker chaque nom d’entité. 

Cela peut se faire facilement en utilisant le symbole « \* », comme suit :

```json
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/customEntities/*/name",
            "targetFieldName": "diseases"
        }
    ]
```

Cette opération a simplement pour effet d’« aplatir » chaque nom d’élément customEntities dans un même tableau de chaînes, comme ceci :

```json
  "diseases" : ["heart failure","morquio"]
```

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez mappé les champs enrichis sur des champs pouvant faire l’objet d’une recherche, vous pouvez définir les attributs de chacun des champs pouvant faire l’objet d’une recherche [dans le cadre de la définition de l’index](search-what-is-an-index.md).

Pour en savoir plus sur le mappage de champs, consultez [Mappages de champs dans les indexeurs de Recherche cognitive Azure](search-indexer-field-mappings.md).