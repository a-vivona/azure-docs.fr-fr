---
title: Classeurs Azure Monitor et modèles Azure Resource Manager
description: Créez des rapports complexes en toute simplicité grâce à des classeurs Azure Monitor paramétrables prédéfinis et personnalisés déployés au moyen de modèles Azure Resource Manager.
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/30/2020
ms.openlocfilehash: d48cefcae16829bc0a58cdb4f4a43f52d78f8460
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122563303"
---
# <a name="programmatically-manage-workbooks"></a>Gérer programmatiquement des classeurs

Les propriétaires de ressources ont la possibilité de créer et de gérer programmatiquement leurs classeurs au moyen de modèles Resource Manager,

ce qui peut être utile dans différents scénarios :
* le déploiement de rapports d’analytique propres à l’organisation ou au domaine, ainsi que le déploiement de ressources (par exemple, des classeurs de performances et de défaillances propres à l’organisation pour de nouvelles applications ou machines virtuelles) ;
* le déploiement de rapports ou de tableaux de bord standard à l’aide de classeurs pour des ressources existantes.

Le classeur sera créé dans le groupe ou sous-groupe de ressources souhaité et comportera le contenu spécifié dans les modèles Resource Manager.

Il existe deux types de ressources de classeur qui peuvent être gérés par programme :
* [Modèles de classeur](#azure-resource-manager-template-for-deploying-a-workbook-template)
* [Instances de classeur](#azure-resource-manager-template-for-deploying-a-workbook-instance)

## <a name="azure-resource-manager-template-for-deploying-a-workbook-template"></a>Modèle Azure Resource Manager pour le déploiement d’un modèle de classeur

1. Ouvrez un classeur que vous voulez déployer par programme.
2. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
3. Ouvrez _l’Éditeur avancé_ avec le bouton _</>_ de la barre d’outils.
4. Vérifiez que vous êtes sous l’onglet _Modèles de galerie_.

    ![Onglet Modèle de galerie](./media/workbooks-automate/gallery-template.png)
1. Copiez le JSON figurant dans le modèle de galerie dans le presse-papiers.
2. Voici un exemple de modèle Azure Resource Manager qui déploie un modèle de classeur dans une galerie de classeurs Azure Monitor. Collez le JSON que vous avez copié à la place de `<PASTE-COPIED-WORKBOOK_TEMPLATE_HERE>`. Vous pouvez trouver [ici](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Documentation/ARM-template-for-creating-workbook-template) un modèle de référence Azure Resource Manager qui crée un modèle de classeur.

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "resourceName": {
                "type": "string",
                "defaultValue": "my-workbook-template",
                "metadata": {
                    "description": "The unique name for this workbook template instance"
                }
            }
        },
        "resources": [
            {
                "name": "[parameters('resourceName')]",
                "type": "microsoft.insights/workbooktemplates",
                "location": "[resourceGroup().location]",
                "apiVersion": "2019-10-17-preview",
                "dependsOn": [],
                "properties": {
                    "galleries": [
                        {
                            "name": "A Workbook Template",
                            "category": "Deployed Templates",
                            "order": 100,
                            "type": "workbook",
                            "resourceType": "Azure Monitor"
                        }
                    ],
                    "templateData": <PASTE-COPIED-WORKBOOK_TEMPLATE_HERE>
                }
            }
        ]
    }
    ```
1. Dans l’objet `galleries`, renseignez les clés `name` et `category` avec vos valeurs. Apprenez-en davantage sur les [paramètres](#parameters) dans la section suivante.
2. Déployez ce modèle Azure Resource Manager à l’aide du [Portail Azure](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), de l’[interface de ligne de commande](../../azure-resource-manager/templates/deploy-cli.md), de [PowerShell](../../azure-resource-manager/templates/deploy-powershell.md), etc.
3. Ouvrez le portail Azure et accédez à la galerie de classeurs choisie dans le modèle Azure Resource Manager. Dans l’exemple de modèle, accédez à la galerie de classeurs Azure Monitor :
    1. Ouvrez le portail Azure, puis accédez à Azure Monitor.
    2. Ouvrez `Workbooks` à partir de la table des matières.
    3. Recherchez votre modèle dans la galerie sous la catégorie `Deployed Templates` (un des éléments violets).

### <a name="parameters"></a>Paramètres

|Paramètres                |Explication                                                                                             |
|:-------------------------|:-------------------------------------------------------------------------------------------------------|
| `name`                   | Nom de la ressource de modèle de classeur dans Azure Resource Manager.                                  |
|`type`                    | Toujours microsoft.insights/workbooktemplates                                                            |
| `location`               | Emplacement Azure dans lequel le classeur sera créé.                                               |
| `apiVersion`             | Préversion du 17-10-2019                                                                                     |
| `type`                   | Toujours microsoft.insights/workbooktemplates                                                            |
| `galleries`              | Ensemble de galeries dans lequel afficher ce modèle de classeur.                                                |
| `gallery.name`           | Nom convivial du modèle de classeur dans la galerie.                                             |
| `gallery.category`       | Groupe dans la galerie dans lequel placer le modèle.                                                     |
| `gallery.order`          | Nombre qui détermine l’ordre d’affichage du modèle dans une catégorie de la galerie. Un ordre inférieur implique une priorité supérieure. |
| `gallery.resourceType`   | Type de ressource correspondant à la galerie. Il s’agit généralement de la chaîne de type de ressource correspondant à la ressource (par exemple, microsoft.operationalinsights/workspaces). |
|`gallery.type`            | Appelé type de classeur, il s’agit d’une clé unique qui distingue la galerie à l’intérieur d’un type de ressource. Les Application Insights, par exemple, ont des types `workbook` et `tsg` correspondant à différentes galeries de classeurs. |

### <a name="galleries"></a>Galeries

| Galerie                                        | Type de ressource                                       | Type de classeur |
|:-----------------------------------------------|:----------------------------------------------------|:--------------|
| Classeurs dans Azure Monitor                     | `Azure Monitor`                                     | `workbook`    |
| Insights de machine virtuelle dans Azure Monitor                   | `Azure Monitor`                                     | `vm-insights` |
| Classeurs dans l’espace de travail Log Analytics           | `microsoft.operationalinsights/workspaces`          | `workbook`    |
| Classeurs dans Application Insights              | `microsoft.insights/components`                     | `workbook`    |
| Guides de résolution des problèmes dans Application Insights | `microsoft.insights/components`                     | `tsg`         |
| Utilisation dans Application Insights                  | `microsoft.insights/components`                     | `usage`       |
| Classeurs dans le service Kubernetes                | `Microsoft.ContainerService/managedClusters`        | `workbook`    |
| Classeurs dans des groupes de ressources                   | `microsoft.resources/subscriptions/resourcegroups`  | `workbook`    |
| Classeurs dans Azure Active Directory            | `microsoft.aadiam/tenant`                           | `workbook`    |
| Insights de machine virtuelle dans des machines virtuelles                | `microsoft.compute/virtualmachines`                 | `insights`    |
| Insights de machine virtuelle dans des groupes de machines virtuelles identiques      | `microsoft.compute/virtualmachinescalesets`         | `insights`    |

## <a name="azure-resource-manager-template-for-deploying-a-workbook-instance"></a>Modèle Azure Resource Manager pour le déploiement d’une instance de classeur

1. Ouvrez le classeur à déployer programmatiquement.
2. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
3. Ouvrez _l’Éditeur avancé_ avec le bouton _</>_ de la barre d’outils.
4. Dans l’éditeur, basculez _Type de modèle_ sur _Modèle Resource Manager_.
5. Le modèle Resource Manager de création apparaît dans l’éditeur. Copiez le contenu et utilisez-le tel quel ou fusionnez-le avec un modèle plus vaste qui déploie également la ressource cible.

    ![Image montrant comment obtenir le modèle Resource Manager sur l’interface utilisateur du classeur](./media/workbooks-automate/programmatic-template.png)

## <a name="sample-azure-resource-manager-template"></a>Exemple de modèle Azure Resource Manager
Ce modèle montre comment déployer un classeur simple qui affiche « Hello World ! » :
```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workbookDisplayName":  {             
            "type":"string",
            "defaultValue": "My Workbook",
            "metadata": {
                "description": "The friendly name for the workbook that is used in the Gallery or Saved List. Needs to be unique in the scope of the resource group and source" 
            }
        },
        "workbookType":  {             
            "type":"string",
            "defaultValue": "tsg",
            "metadata": {
                "description": "The gallery that the workbook will been shown under. Supported values include workbook, `tsg`, Azure Monitor, etc." 
            }
        },
        "workbookSourceId":  {             
            "type":"string",
            "defaultValue": "<insert-your-resource-id-here>",
            "metadata": {
                "description": "The id of resource instance to which the workbook will be associated" 
            }
        },
        "workbookId": {
            "type":"string",
            "defaultValue": "[newGuid()]",
            "metadata": {
                "description": "The unique guid for this workbook instance" 
            }
        }
    },    
    "resources": [
        {
            "name": "[parameters('workbookId')]",
            "type": "Microsoft.Insights/workbooks",
            "location": "[resourceGroup().location]",
            "kind": "shared",
            "apiVersion": "2018-06-17-preview",
            "dependsOn": [],
            "properties": {
                "displayName": "[parameters('workbookDisplayName')]",
                "serializedData": "{\"version\":\"Notebook/1.0\",\"items\":[{\"type\":1,\"content\":\"{\\\"json\\\":\\\"Hello World!\\\"}\",\"conditionalVisibility\":null}],\"isLocked\":false}",
                "version": "1.0",
                "sourceId": "[parameters('workbookSourceId')]",
                "category": "[parameters('workbookType')]"
            }
        }
    ],
    "outputs": {
        "workbookId": {
            "type": "string",
            "value": "[resourceId( 'Microsoft.Insights/workbooks', parameters('workbookId'))]"
        }
    }
}
```

### <a name="template-parameters"></a>Paramètres de modèle

| Paramètre | Explication |
| :------------- |:-------------|
| `workbookDisplayName` | Nom convivial du classeur utilisé dans la galerie ou la liste enregistrée. Doit être unique dans l’étendue du groupe de ressources et de la source. |
| `workbookType` | Galerie dans laquelle le classeur s’affichera. Valeurs prises en charge : workbook, `tsg`, Azure Monitor, etc. |
| `workbookSourceId` | ID de l’instance de ressource à laquelle le classeur sera associé. Le nouveau classeur s’affichera en rapport avec cette instance de ressource, par exemple dans le sommaire de la ressource sous _Classeur_. Si vous souhaitez que votre classeur apparaisse dans la galerie de classeurs d’Azure Monitor, utilisez la chaîne _Azure Monitor_ au lieu d’un ID de ressource. |
| `workbookId` | GUID unique de cette instance de classeur. Utilisez _[newGuid()]_ pour créer automatiquement un nouveau GUID. |
| `kind` | Permet de spécifier si le classeur créé est partagé ou privé. Tous les nouveaux classeurs utilisent la valeur _partagé_. |
| `location` | Emplacement Azure où sera créé le classeur. Utilisez _[resourceGroup().location]_ pour le créer dans le même emplacement que le groupe de ressources. |
| `serializedData` | Contient le contenu ou la charge utile à utiliser dans le classeur. Utilisez le modèle Resource Manager de l’interface utilisateur des classeurs pour récupérer la valeur. |

### <a name="workbook-types"></a>Types de classeurs
Les types de classeurs spécifient dans quel type de galerie de classeurs la nouvelle instance de classeur s’affichera. Options disponibles :

| Type | Emplacement de la galerie |
| :------------- |:-------------|
| `workbook` | Valeur par défaut utilisée dans la plupart des rapports, qui comprend la galerie de classeurs d’Application Insights, d’Azure Monitor, etc.  |
| `tsg` | Galerie de guides de dépannage dans Application Insights |
| `usage` | Galerie _Plus_ sous _Utilisation_ dans Application Insights |

### <a name="working-with-json-formatted-workbook-data-in-the-serializeddata-template-parameter"></a>Utilisation de données de classeur au format JSON dans le paramètre de modèle serializedData

Lors de l’exportation d’un modèle Azure Resource Manager pour un classeur Azure, il existe souvent des liens de ressources fixes incorporés dans le paramètre de modèle `serializedData` exporté. Ceux-ci incluent des valeurs potentiellement sensibles, telles que l’ID d’abonnement et le nom du groupe de ressources, ainsi que d’autres types d’ID de ressource.

L’exemple ci-dessous illustre la personnalisation d’un modèle Azure Resource Manager de classeur exporté, sans recourir à la manipulation de chaînes. Le modèle présenté dans cet exemple est conçu pour utiliser les données non modifiées telles qu’elles sont exportées depuis le portail Azure. Il est également recommandé de masquer toute valeur sensible intégrée lorsque les classeurs sont gérés par programmation. Par conséquent, l’ID d’abonnement et le groupe de ressources ont été masqués ici. Aucune autre modification n’a été apportée à la valeur `serializedData` entrante brute.

```json
{
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workbookDisplayName": {
      "type": "string"
    },
    "workbookSourceId": {
      "type": "string",
      "defaultValue": "[resourceGroup().id]"
    },
    "workbookId": {
      "type": "string",
      "defaultValue": "[newGuid()]"
    }
  },
  "variables": {
    // serializedData from original exported Azure Resource Manager template
    "serializedData": "{\"version\":\"Notebook/1.0\",\"items\":[{\"type\":1,\"content\":{\"json\":\"Replace with Title\"},\"name\":\"text - 0\"},{\"type\":3,\"content\":{\"version\":\"KqlItem/1.0\",\"query\":\"{\\\"version\\\":\\\"ARMEndpoint/1.0\\\",\\\"data\\\":null,\\\"headers\\\":[],\\\"method\\\":\\\"GET\\\",\\\"path\\\":\\\"/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups\\\",\\\"urlParams\\\":[{\\\"key\\\":\\\"api-version\\\",\\\"value\\\":\\\"2019-06-01\\\"}],\\\"batchDisabled\\\":false,\\\"transformers\\\":[{\\\"type\\\":\\\"jsonpath\\\",\\\"settings\\\":{\\\"tablePath\\\":\\\"$..*\\\",\\\"columns\\\":[]}}]}\",\"size\":0,\"queryType\":12,\"visualization\":\"map\",\"tileSettings\":{\"showBorder\":false},\"graphSettings\":{\"type\":0},\"mapSettings\":{\"locInfo\":\"AzureLoc\",\"locInfoColumn\":\"location\",\"sizeSettings\":\"location\",\"sizeAggregation\":\"Count\",\"opacity\":0.5,\"legendAggregation\":\"Count\",\"itemColorSettings\":null}},\"name\":\"query - 1\"}],\"isLocked\":false,\"fallbackResourceIds\":[\"/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/XXXXXXX\"]}",

    // parse the original into a JSON object, so that it can be manipulated
    "parsedData": "[json(variables('serializedData'))]",

    // create new JSON objects that represent only the items/properties to be modified
    "updatedTitle": {
      "content":{
        "json": "[concat('Resource Group Regions in subscription \"', subscription().displayName, '\"')]"
      }
    },
    "updatedMap": {
      "content": {
        "path": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups')]"
      }
    },

    // the union function applies the updates to the original data
    "updatedItems": [
      "[union(variables('parsedData')['items'][0], variables('updatedTitle'))]",
      "[union(variables('parsedData')['items'][1], variables('updatedMap'))]"
    ],

    // copy to a new workbook object, with the updated items
    "updatedWorkbookData": {
      "version": "[variables('parsedData')['version']]",
      "items": "[variables('updatedItems')]",
      "isLocked": "[variables('parsedData')['isLocked']]",
      "fallbackResourceIds": ["[parameters('workbookSourceId')]"]
    },

    // convert back to an encoded string
    "reserializedData": "[string(variables('updatedWorkbookData'))]"
  },
  "resources": [
    {
      "name": "[parameters('workbookId')]",
      "type": "microsoft.insights/workbooks",
      "location": "[resourceGroup().location]",
      "apiVersion": "2018-06-17-preview",
      "dependsOn": [],
      "kind": "shared",
      "properties": {
        "displayName": "[parameters('workbookDisplayName')]",
        "serializedData": "[variables('reserializedData')]",
        "version": "1.0",
        "sourceId": "[parameters('workbookSourceId')]",
        "category": "workbook"
      }
    }
  ],
  "outputs": {
    "workbookId": {
      "type": "string",
      "value": "[resourceId( 'microsoft.insights/workbooks', parameters('workbookId'))]"
    }
  },
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#"
}
```

Dans cet exemple, les étapes suivantes facilitent la personnalisation d’un modèle Azure Resource Manager exporté :
1. Exportez le classeur en tant que modèle Azure Resource Manager, comme expliqué dans la section ci-dessus.
2. Dans la section `variables` du modèle :
    1. Analysez la valeur `serializedData` dans une variable objet JSON, qui crée une structure JSON comprenant un tableau d’éléments qui représentent le contenu du classeur.
    2. Créez de nouveaux objets JSON qui représentent uniquement les éléments/propriétés à modifier.
    3. Projetez un nouvel ensemble d’éléments de contenu JSON (`updatedItems`) en utilisant la fonction `union()` pour appliquer les modifications aux éléments JSON d’origine.
    4. Créez un nouvel objet de classeur, `updatedWorkbookData`, qui contient `updatedItems` les données `version`/`isLocked` des données analysées d’origine, ainsi qu’un ensemble corrigé de `fallbackResourceIds`.
    5. Sérialisez le nouveau contenu JSON dans une nouvelle variable de chaîne, `reserializedData`.
3. Utilisez la nouvelle variable `reserializedData` à la place de la propriété `serializedData` d’origine.
4. Déployez la nouvelle ressource de classeur à l’aide du modèle Azure Resource Manager mis à jour.

### <a name="limitations"></a>Limites
Pour une raison technique, ce mécanisme ne permet pas de créer des instances de classeur dans la galerie _Classeurs_ d’Application Insights. Nous mettons tout en œuvre pour corriger cette limitation. En attendant, nous vous recommandons d’utiliser la Galerie de guides de dépannage (workbookType : `tsg`) pour déployer des classeurs Application Insights.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment les classeurs sont utilisés dans la nouvelle [expérience d’insights pour le stockage](../../storage/common/storage-insights-overview.md?toc=%2fazure%2fazure-monitor%2ftoc.json).