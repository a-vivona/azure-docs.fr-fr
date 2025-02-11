---
title: Comprendre le langage de requête
description: Décrit les tables Resource Graph et les fonctions, opérateurs et types de données Kusto disponibles, utilisables avec Azure Resource Graph.
ms.date: 08/27/2021
ms.topic: conceptual
ms.openlocfilehash: 1f1eef5a70187ed883da9f535cc2f2915e3bf6d6
ms.sourcegitcommit: dcf1defb393104f8afc6b707fc748e0ff4c81830
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2021
ms.locfileid: "123104650"
---
# <a name="understanding-the-azure-resource-graph-query-language"></a>Présentation du langage de requête Azure Resource Graph

Le langage de requête pour Azure Resource Graph prend en charge un certain nombre d’opérateurs et de fonctions. Leur fonctionnement et leur utilisation s’appuient sur le [langage de requête Kusto (KQL, Kusto Query Language)](/azure/data-explorer/kusto/query/index). Pour en savoir plus sur le langage de requête utilisé par Resource Graph, commencez par suivre le [tutoriel KQL](/azure/data-explorer/kusto/query/tutorial).

Cet article traite des composants de langage pris en charge par Resource Graph :

- [Tables Resource Graph](#resource-graph-tables)
- [Éléments de langage personnalisés Resource Graph](#resource-graph-custom-language-elements)
- [Éléments du langage KQL pris en charge](#supported-kql-language-elements)
- [Étendue de la requête](#query-scope)
- [Caractères d’échappement](#escape-characters)

## <a name="resource-graph-tables"></a>Tables Resource Graph

Resource Graph fournit plusieurs tables contenant les données qu’il stocke sur les types de ressources Azure Resource Manager et leurs propriétés. Vous pouvez utiliser certaines tables avec l’opérateur `join` ou `union` pour récupérer des propriétés à partir des types de ressources associés. Voici la liste des tables disponibles dans Resource Graph :

|Table Resource Graph |Peut `join` d’autres tables ? |Description |
|---|---|---|
|Ressources |Oui |Table par défaut si aucune table n’est définie dans la requête. Elle contient la plupart des types de ressources et propriétés Resource Manager. |
|ResourceContainers |Oui |Inclut les données et les types de ressources du groupe d’administration (`Microsoft.Management/managementGroups`), de l’abonnement (`Microsoft.Resources/subscriptions`) et du groupe de ressources (`Microsoft.Resources/subscriptions/resourcegroups`). |
|AdvisorResources |Oui (préversion) |Inclut les ressources _associées_ à `Microsoft.Advisor`. |
|AlertsManagementResources |Oui (préversion) |Inclut les ressources _associées_ à `Microsoft.AlertsManagement`. |
|DesktopVirtualizationResources |Oui |Inclut les ressources _associées_ à `Microsoft.DesktopVirtualization`. |
|ExtendedLocationResources |Non |Inclut les ressources _associées_ à `Microsoft.ExtendedLocation`. |
|GuestConfigurationResources |Non |Inclut les ressources _associées_ à `Microsoft.GuestConfiguration`. |
|HealthResources|Oui |Inclut les ressources _associées_ à `Microsoft.ResourceHealth/availabilitystatuses`. |
|KubernetesConfigurationResources |Non |Inclut les ressources _associées_ à `Microsoft.KubernetesConfiguration`. |
|MaintenanceResources |Partielle, joindre _à_ uniquement. (préversion) |Inclut les ressources _associées_ à `Microsoft.Maintenance`. |
|PatchAssessmentResources|Non |Comprend des ressources _associées_ à l’évaluation des correctifs de machines virtuelles Azure. |
|PatchInstallationResources|Non |Comprend des ressources _associées_ à l’installation de correctifs de machines virtuelles Azure. |
|PolicyResources |Oui |Inclut les ressources _associées_ à `Microsoft.PolicyInsights`. |
|RecoveryServicesResources |Partielle, joindre _à_ uniquement. (préversion) |Comprend des ressources _associées_ à `Microsoft.DataProtection` et `Microsoft.RecoveryServices`. |
|SecurityResources |Oui (préversion) |Inclut les ressources _associées_ à `Microsoft.Security`. |
|ServiceHealthResources |Non (préversion) |Inclut les ressources _associées_ à `Microsoft.ResourceHealth/events`. |
|WorkloadMonitorResources |Non |Inclut les ressources _associées_ à `Microsoft.WorkloadMonitor`. |

Pour obtenir une liste complète, y compris les types de ressources, consultez [Référence : Tables et types de ressources pris en charge](../reference/supported-tables-resources.md).

> [!NOTE]
> La table _Resources_  est la table par défaut. Quand vous interrogez la table _Resources_ , il n’est pas nécessaire d’indiquer le nom de la table, sauf si vous utilisez `join` ou `union`. Toutefois, la pratique recommandée consiste à toujours inclure la table initiale dans la requête.

Utilisez l’Explorateur Resource Graph dans le portail pour découvrir les types de ressources disponibles dans chaque table. Vous pouvez également utiliser une requête telle que `<tableName> | distinct type` pour obtenir la liste des types de ressources existant dans votre environnement, pris en charge par la table Resource Graph concernée.

La requête suivante illustre un opérateur `join` simple. Le résultat de la requête fusionne les colonnes, et le suffixe **1** est ajouté à tous les noms de colonnes dupliqués de la table jointe (_ResourceContainers_ dans cet exemple). Comme la table _ResourceContainers_ possède des types tant pour les abonnements que pour les groupes de ressources, les deux types peuvent être utilisés pour la jointure à la ressource à partir de la table _Ressources_.

```kusto
Resources
| join ResourceContainers on subscriptionId
| limit 1
```

La requête suivante illustre une utilisation plus complexe de l’opérateur `join`. Tout d’abord, la requête utilise `project` pour obtenir les champs de _Ressources_ pour le type de ressource coffre Azure Key Vault. L’étape suivante utilise `join` pour fusionner les résultats avec _ResourceContainers_, où le type est un abonnement _sur_ une propriété qui se trouve à la fois dans le `project` de la première table et dans le `project` de la table jointe. Le renommage de champ évite que `join` l’ajoute en tant que _name1_, puisque la propriété est déjà projetée à partir de _Ressources_. Le résultat de la requête est un coffre de clés unique affichant le type, le nom, l’emplacement et le groupe de ressources du coffre de clés, ainsi que le nom de l’abonnement dans lequel il se trouve.

```kusto
Resources
| where type == 'microsoft.keyvault/vaults'
| project name, type, location, subscriptionId, resourceGroup
| join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| project type, name, location, resourceGroup, SubName
| limit 1
```

> [!NOTE]
> Quand vous limitez les résultats de `join` avec `project`, la propriété utilisée par `join` pour associer les deux tables (_subscriptionId_  dans l’exemple ci-dessus) doit être incluse dans `project`.

## <a name="extended-properties-preview"></a><a name="extended-properties"></a>Propriétés étendues (préversion)

En tant que fonctionnalité de _préversion_, certains types de ressources dans Resource Graph ont des propriétés supplémentaires liées au type, disponibles pour effectuer une requête au-delà des propriétés fournies par Azure Resource Manager. Cet ensemble de valeurs, connu sous le nom de _propriétés étendues_, existe sur un type de ressource pris en charge dans `properties.extended`. Pour voir quels types de ressources ont des _propriétés étendues_, utilisez la requête suivante :

```kusto
Resources
| where isnotnull(properties.extended)
| distinct type
| order by type asc
```

Exemple : Obtenir le nombre total de machines virtuelles à l’aide de `instanceView.powerState.code` :

```kusto
Resources
| where type == 'microsoft.compute/virtualmachines'
| summarize count() by tostring(properties.extended.instanceView.powerState.code)
```

## <a name="resource-graph-custom-language-elements"></a>Éléments de langage personnalisés Resource Graph

### <a name="shared-query-syntax-preview"></a><a name="shared-query-syntax"></a>Syntaxe des requêtes partagées (préversion)

La fonctionnalité d’évaluation [requête partagée](../tutorials/create-share-query.md) est accessible directement dans une requête Resource Graph. Ce scénario permet de créer des requêtes standard comme requêtes partagées et de les réutiliser. Pour appeler une requête partagée à l’intérieur d’une requête Resource Graph, utilisez la syntaxe `{{shared-query-uri}}`. L’URI de la requête partagée correspond à son _ID de ressource_ sur la page **Paramètres** correspondante. Dans cet exemple, l’URI de la requête partagée est `/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS`.
Il pointe vers l’abonnement, le groupe de ressources et le nom complet de la requête partagée à laquelle nous souhaitons faire référence dans une autre requête. Cette requête est identique à celle créée dans [Tutoriel : Création et partage d’une requête](../tutorials/create-share-query.md).

> [!NOTE]
> Il n’est pas possible d’enregistrer comme requête partagée une requête qui fait référence à une requête partagée.

Exemple 1 : Utiliser uniquement la requête partagée

Les résultats de cette requête Resource Graph sont les mêmes que ceux de la requête stockée dans la requête partagée.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
```

Exemple 2 : Inclure la requête partagée dans une requête plus large

Cette requête utilise d’abord la requête partagée, puis se sert de `limit` pour limiter encore les résultats.

```kusto
{{/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SharedQueries/providers/Microsoft.ResourceGraph/queries/Count VMs by OS}}
| where properties_storageProfile_osDisk_osType =~ 'Windows'
```

## <a name="supported-kql-language-elements"></a>Éléments du langage KQL pris en charge

Resource Graph prend en charge un sous-ensemble des [types de données](/azure/data-explorer/kusto/query/scalar-data-types/), [fonctions scalaires](/azure/data-explorer/kusto/query/scalarfunctions), [opérateurs scalaires](/azure/data-explorer/kusto/query/binoperators) et [fonctions d’agrégation](/azure/data-explorer/kusto/query/any-aggfunction) du langage KQL. Des [opérateurs tabulaires](/azure/data-explorer/kusto/query/queries) spécifiques sont pris en charge par Resource Graph, dont certains présentent différents comportements.

### <a name="supported-tabulartop-level-operators"></a>Opérateurs tabulaires/de niveau supérieur pris en charge

Voici la liste des opérateurs tabulaires KQL pris en charge par Resource Graph avec des exemples spécifiques :

|KQL |Exemple de requête Resource Graph |Notes |
|---|---|---|
|[count](/azure/data-explorer/kusto/query/countoperator) |[Compter les coffres de clés](../samples/starter.md#count-keyvaults) | |
|[distinct](/azure/data-explorer/kusto/query/distinctoperator) |[Afficher les ressources contenant storage](../samples/starter.md#show-storage) | |
|[extend](/azure/data-explorer/kusto/query/extendoperator) |[Compter les machines virtuelles par type de système d’exploitation](../samples/starter.md#count-os) | |
|[join](/azure/data-explorer/kusto/query/joinoperator) |[Coffre de clés avec nom d’abonnement](../samples/advanced.md#join) |Variantes de jointure prises en charge : [innerunique](/azure/data-explorer/kusto/query/joinoperator#default-join-flavor), [inner](/azure/data-explorer/kusto/query/joinoperator#inner-join), [leftouter](/azure/data-explorer/kusto/query/joinoperator#left-outer-join). Limite de 3 `join` dans une requête, dont une peut être une `join` entre tables. Si l’usage de `join` entre tables est partagé entre _Resource_ et _ResourceContainers_, 3 `join` entre tables sont autorisées. Les stratégies de jointure personnalisées comme la jointure de diffusion ne sont pas autorisées. Pour savoir quelles tables peuvent utiliser `join`, consultez [Tables Resource Graph](#resource-graph-tables). |
|[limit](/azure/data-explorer/kusto/query/limitoperator) |[Lister toutes les adresses IP publiques](../samples/starter.md#list-publicip) |Identique à `take`. Ne fonctionne pas avec [Ignorer](./work-with-data.md#skipping-records). |
|[mvexpand](/azure/data-explorer/kusto/query/mvexpandoperator) | | Opérateur hérité, utilisez `mv-expand` à la place. Valeur _RowLimit_ maximale de 400. La valeur par défaut est 128. |
|[mv-expand](/azure/data-explorer/kusto/query/mvexpandoperator) |[Lister Cosmos DB avec des emplacements d’écriture spécifiques](../samples/advanced.md#mvexpand-cosmosdb) |Valeur _RowLimit_ maximale de 400. La valeur par défaut est 128. Limite de 2 `mv-expand` dans une requête unique.|
|[order](/azure/data-explorer/kusto/query/orderoperator) |[Lister les ressources triées par nom](../samples/starter.md#list-resources) |Identique à `sort` |
|[parse](/azure/data-explorer/kusto/query/parseoperator) |[Obtenir les réseaux virtuels et les sous-réseaux des interfaces réseau](../samples/advanced.md#parse-subnets) |Il est préférable d’accéder directement aux propriétés, le cas échéant, au lieu d’utiliser `parse`. |
|[project](/azure/data-explorer/kusto/query/projectoperator) |[Lister les ressources triées par nom](../samples/starter.md#list-resources) | |
|[project-away](/azure/data-explorer/kusto/query/projectawayoperator) |[Supprimer des colonnes des résultats](../samples/advanced.md#remove-column) | |
|[sort](/azure/data-explorer/kusto/query/sortoperator) |[Lister les ressources triées par nom](../samples/starter.md#list-resources) |Identique à `order` |
|[summarize](/azure/data-explorer/kusto/query/summarizeoperator) |[Compter les ressources Azure](../samples/starter.md#count-resources) |Première page simplifiée uniquement |
|[take](/azure/data-explorer/kusto/query/takeoperator) |[Lister toutes les adresses IP publiques](../samples/starter.md#list-publicip) |Identique à `limit`. Ne fonctionne pas avec [Ignorer](./work-with-data.md#skipping-records). |
|[top](/azure/data-explorer/kusto/query/topoperator) |[Afficher les cinq premières machines virtuelles par nom et leur type de système d’exploitation](../samples/starter.md#show-sorted) | |
|[union](/azure/data-explorer/kusto/query/unionoperator) |[Combiner les résultats de deux requêtes en un résultat unique](../samples/advanced.md#unionresults) |Table unique autorisée : _T_ `| union` \[`kind=` `inner`\|`outer`\] \[`withsource=`_ColumnName_\] _Table_. Limite de 3 sections `union` dans une requête unique. La résolution approximative des tables avec sections `union` n’est pas autorisée. Peut être utilisé dans une table unique ou entre les tables _Resources_ et _ResourceContainers_. |
|[where](/azure/data-explorer/kusto/query/whereoperator) |[Afficher les ressources contenant storage](../samples/starter.md#show-storage) | |

Il existe une limite par défaut de trois opérateurs `join` et de trois opérateurs `mv-expand` dans une seule requête du Kit de développement logiciel (SDK) Resource Graph. Vous pouvez demander une augmentation de ces limites pour votre locataire via **Aide + support**.

Pour prendre en charge l’expérience de portail « Ouvrir une requête », l’explorateur d’Azure Resource Graph a une limite globale supérieure à celle du Kit de développement logiciel (SDK) Resource Graph.

## <a name="query-scope"></a>Étendue de requête

La portée des abonnements ou des [groupes d’administration](../../management-groups/overview.md) à partir desquels les ressources sont retournées par une requête est définie par défaut sur une liste d’abonnements en fonction du contexte de l’utilisateur autorisé. Si aucun groupe d’administration ou aucune liste d’abonnements n’est défini, l’étendue de requête correspond à l’ensemble des ressources, ce qui inclut les ressources déléguées [Azure Lighthouse](../../../lighthouse/overview.md).

La liste des abonnements ou des groupes d’administration à interroger peut être définie manuellement pour changer l’étendue des résultats. Par exemple, la propriété `managementGroups` de l’API REST prend l’ID du groupe d’administration, qui est différent du nom du groupe d’administration. Quand `managementGroups` est spécifié, les ressources des 5 000 premiers abonnements dans ou sous la hiérarchie du groupe d’administration spécifié sont incluses. `managementGroups` ne peut pas être utilisé en même temps que `subscriptions`.

Exemple : Interroger toutes les ressources dans la hiérarchie du groupe d’administration nommé « Mon groupe d’administration » avec l’ID « myMG ».

- URI de l’API REST

  ```http
  POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2021-03-01
  ```

- Corps de la requête

  ```json
  {
      "query": "Resources | summarize count()",
      "managementGroups": ["myMG"]
  }
  ```

## <a name="escape-characters"></a>Caractères d'échappement

Certains noms propres, tels que ceux qui incluent un `.` ou `$`, doivent être inclus dans un wrapper ou échappés dans la requête, sinon le nom de la propriété est interprété de façon incorrecte et les résultats attendus ne sont pas obtenus.

- `.`- Encapsulez le nom de la propriété de cette façon : `['propertyname.withaperiod']`

  Exemple de requête qui encapsule la propriété _odata.type_ :

  ```kusto
  where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.['odata.type']
  ```

- `$` - Échappez le caractère dans le nom de la propriété. Le caractère d’échappement utilisé dépend de l’interpréteur de commandes Resource Graph sur lequel il est exécuté.

  - **bash** - `\`

    Exemple de requête qui échappe la propriété _\$type_ dans Bash :

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.\$type
    ```

  - **cmd** - ne pas échapper le caractère `$`.

  - **PowerShell** - ``` ` ```

    Exemple de requête qui échappe la propriété _\$type_ dans PowerShell :

    ```kusto
    where type=~'Microsoft.Insights/alertRules' | project name, properties.condition.`$type
    ```

## <a name="next-steps"></a>Étapes suivantes

- Examinez le langage utilisé dans les [requêtes de démarrage](../samples/starter.md).
- Examinez les utilisations avancées dans les [Requêtes avancées](../samples/advanced.md).
- Découvrez plus en détails comment [explorer des ressources](explore-resources.md).
