---
title: Colonnes standard dans les enregistrements de journaux Azure Monitor | Microsoft Docs
description: Décrit les colonnes communes à plusieurs types de données dans les journaux Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/16/2021
ms.openlocfilehash: 909c02c53f753579d6788933277bca8f75f53859
ms.sourcegitcommit: d43193fce3838215b19a54e06a4c0db3eda65d45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2021
ms.locfileid: "122535396"
---
# <a name="standard-columns-in-azure-monitor-logs"></a>Colonnes standard dans les journaux Azure Monitor
Les données des journaux Azure Monitor sont [stockées sous la forme d’un jeu d’enregistrements dans un espace de travail Log Analytics ou dans une application Application Insights](../logs/data-platform-logs.md), chacun ayant un type de données particulier associé à un ensemble unique de colonnes. De nombreux types de données comportent des colonnes standard qui sont communes à plusieurs types. Cet article décrit ces colonnes et fournit des exemples sur la façon dont vous pouvez les utiliser dans des requêtes.

Dans Application Insights, les applications basées sur un espace de travail stockent leurs données dans un espace de travail Log Analytics et utilisent les mêmes colonnes standard que les autres tables de l’espace de travail. Les applications classiques stockent leurs données séparément et présentent des colonnes standard différentes, comme spécifié dans cet article.

> [!NOTE]
> Certaines des colonnes standard n'apparaîtront pas dans la vue schéma ou IntelliSense de Log Analytics, et n'apparaîtront pas dans les résultats des requêtes, à moins de spécifier explicitement la colonne dans la sortie.
> 

## <a name="tenantid"></a>TenantId
La colonne **TenantId** contient l’ID de l’espace de travail Log Analytics.

## <a name="timegenerated"></a>TimeGenerated
La colonne **TimeGenerated** contient la date et l’heure auxquelles l’enregistrement a été créé par la source de données. Pour plus d’informations, consultez [Durée d’ingestion de données de journal dans Azure Monitor](../logs/data-ingestion-time.md).

**TimeGenerated** fournit une colonne commune à utiliser pour filtrer ou résumer par heure. Quand vous sélectionnez un intervalle de temps pour un affichage ou un tableau de bord dans le portail Azure, **TimeGenerated** est utilisée pour filtrer les résultats. 

> [!NOTE]
> Les tables prenant en charge les ressources Application Insights classiques utilisent la colonne **timestamp** au lieu de la colonne **TimeGenerated**.

### <a name="examples"></a>Exemples

La requête suivante renvoie le nombre d’événements d’erreur créés pour chaque jour de la semaine précédente.

```Kusto
Event
| where EventLevelName == "Error" 
| where TimeGenerated between(startofweek(ago(7days))..endofweek(ago(7days))) 
| summarize count() by bin(TimeGenerated, 1day) 
| sort by TimeGenerated asc 
```
## <a name="_timereceived"></a>\_TimeReceived
La colonne **\_TimeReceived** contient la date et l'heure auxquelles l'enregistrement a été reçu par le point d'ingestion Azure Monitor dans le cloud Azure. Cela peut être utile pour identifier les problèmes de latence entre la source de données et le cloud. Par exemple, un problème de mise en réseau entraîne un retard dans l’envoi des données par un agent. Pour plus d’informations, consultez [Durée d’ingestion de données de journal dans Azure Monitor](../logs/data-ingestion-time.md).

> [!NOTE]
> La colonne **\_TimeReceived** est calculée chaque fois qu’elle est utilisée. Ce processus consomme beaucoup de ressources. Affinez son utilisation pour filtrer un grand nombre d’enregistrements. L’utilisation de cette fonction en parallèle peut entraîner une augmentation de la durée d’exécution de la requête.


La requête suivante fournit la latence moyenne par heure pour les enregistrements d'événements d'un agent. Cela comprend le temps entre l’agent et le cloud et le temps total pendant lequel l'enregistrement doit être disponible pour les requêtes du journal.

```Kusto
Event
| where TimeGenerated > ago(1d) 
| project TimeGenerated, TimeReceived = _TimeReceived, IngestionTime = ingestion_time() 
| extend AgentLatency = toreal(datetime_diff('Millisecond',TimeReceived,TimeGenerated)) / 1000
| extend TotalLatency = toreal(datetime_diff('Millisecond',IngestionTime,TimeGenerated)) / 1000
| summarize avg(AgentLatency), avg(TotalLatency) by bin(TimeGenerated,1hr)
``` 

## <a name="type"></a>Type
La colonne **Type** contient le nom de la table dans laquelle l’enregistrement a été récupéré, qui peut également être considéré comme le type d’enregistrement. Cette colonne est utile dans les requêtes qui combinent des enregistrements issus de plusieurs tableaux, comme ceux utilisés par l’opérateur `search` pour distinguer les enregistrements de différents types. **$table** peut être utilisé à la place de **Type** dans certaines requêtes.

> [!NOTE]
> Les tables prenant en charge les ressources Application Insights classiques utilisent la colonne **itemType** au lieu de la colonne **Type**.

### <a name="examples"></a>Exemples
La requête suivante renvoie le nombre d’enregistrements par type collectés au cours de la dernière heure.

```Kusto
search * 
| where TimeGenerated > ago(1h)
| summarize count() by Type

```
## <a name="_itemid"></a>\_ItemId
La colonne **\_ItemId** contient un identificateur unique pour l’enregistrement.


## <a name="_resourceid"></a>\_ResourceId
La colonne **\_ResourceId** contient un identificateur unique de la ressource à laquelle l’enregistrement est associé. Vous disposez ainsi d’une colonne standard à utiliser pour étendre votre requêtes aux enregistrements d’une ressource particulière seulement, ou pour associer des données liées entre plusieurs tables.

Pour les ressources Azure, la valeur de **_ResourceId** est l’[URL d’ID de la ressource Azure](../../azure-resource-manager/templates/template-functions-resource.md). La colonne se limite aux ressources Azure, notamment les ressources [Azure Arc](../../azure-arc/overview.md), ou aux journaux personnalisés qui indiquent l’ID de la ressource lors de l’ingestion.

> [!NOTE]
> Certains types de données ont déjà des champs qui contiennent les ID de ressource Azure ou au moins des parties de ceux-ci, comme l’ID d’abonnement. Même si ces fichiers sont conservés à des fins de compatibilité descendante, il est recommandé d’utiliser la propriété _ResourceId pour établir une corrélation, puisqu’elle est plus cohérente.

### <a name="examples"></a>Exemples
La requête suivante associe des données de performances et d’événement pour chaque ordinateur. Il montre tous les événements dont l’ID est _101_ et l’utilisation des processeurs est supérieure à 50 %.

```Kusto
Perf 
| where CounterName == "% User Time" and CounterValue  > 50 and _ResourceId != "" 
| join kind=inner (     
    Event 
    | where EventID == 101 
) on _ResourceId
```

La requête suivante associe des enregistrements _AzureActivity_ à des enregistrements _SecurityEvent_. Il montre toutes les opérations d’activité avec les utilisateurs qui se sont connectés à ces machines.

```Kusto
AzureActivity 
| where  
    OperationName in ("Restart Virtual Machine", "Create or Update Virtual Machine", "Delete Virtual Machine")  
    and ActivityStatus == "Succeeded"  
| join kind= leftouter (    
   SecurityEvent 
   | where EventID == 4624  
   | summarize LoggedOnAccounts = makeset(Account) by _ResourceId 
) on _ResourceId  
```

La requête suivante analyse **_ResourceId** et agrège les volumes de données facturés par groupe de ressources Azure.

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| parse tolower(_ResourceId) with "/subscriptions/" subscriptionId "/resourcegroups/" 
    resourceGroup "/providers/" provider "/" resourceType "/" resourceName   
| summarize Bytes=sum(_BilledSize) by resourceGroup | sort by Bytes nulls last 
```

L’exécution d’analyses sur différents types de données étant coûteuse, utilisez ces requêtes `union withsource = tt *` avec parcimonie.

Il est toujours plus efficace d’utiliser la colonne \_SubscriptionId que de l’extraire en analysant la colonne \_ResourceId.

## <a name="_subscriptionid"></a>\_SubscriptionId
La colonne **\_SubscriptionId** contient l’ID d’abonnement de la ressource à laquelle l’enregistrement est associé. Vous disposez ainsi d’une colonne standard à utiliser pour étendre votre requête aux enregistrements d’une ressource particulière seulement, ou pour comparer différents abonnements.

Pour les ressources Azure, la valeur de **__SubscriptionId** est la partie de l’abonnement de l’[URL d’ID de la ressource Azure](../../azure-resource-manager/templates/template-functions-resource.md). La colonne se limite aux ressources Azure, notamment les ressources [Azure Arc](../../azure-arc/overview.md), ou aux journaux personnalisés qui indiquent l’ID de la ressource lors de l’ingestion.

> [!NOTE]
> Certains types de données possèdent déjà des champs qui contiennent l’ID d’abonnement Azure. Même si ces fichiers sont conservés à des fins de compatibilité descendante, il est recommandé d’utiliser la colonne \_SubscriptionId pour établir une corrélation, puisqu’elle est plus cohérente.
### <a name="examples"></a>Exemples
La requête suivante examine les données de performances pour les ordinateurs d’un abonnement spécifique. 

```Kusto
Perf 
| where TimeGenerated > ago(24h) and CounterName == "memoryAllocatableBytes"
| where _SubscriptionId == "ebb79bc0-aa86-44a7-8111-cabbe0c43993"
| summarize avgMemoryAllocatableBytes = avg(CounterValue) by Computer
```

La requête suivante analyse **_ResourceId** et agrège les volumes de données facturés par abonnement Azure.

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| summarize Bytes=sum(_BilledSize) by _SubscriptionId | sort by Bytes nulls last 
```

L’exécution d’analyses sur différents types de données étant coûteuse, utilisez ces requêtes `union withsource = tt *` avec parcimonie.


## <a name="_isbillable"></a>\_IsBillable
La colonne **\_IsBillable** spécifie si les données ingérées sont facturables. Les données dont la propriété **\_IsBillable** est égale à `false` sont collectées gratuitement et ne sont pas facturées sur votre compte Azure.

### <a name="examples"></a>Exemples
Pour obtenir la liste des ordinateurs qui envoient des types de données facturés, utilisez la requête suivante :

> [!NOTE]
> Utilisez les requêtes accompagnées de `union withsource = tt *` avec parcimonie, car l’exécution d’analyses sur différents types de données est coûteuse. 

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize TotalVolumeBytes=sum(_BilledSize) by computerName
```

Cela peut être étendu pour renvoyer le nombre d'ordinateurs par heure qui envoient des types de données facturés :

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize dcount(computerName) by bin(TimeGenerated, 1h) | sort by TimeGenerated asc
```

## <a name="_billedsize"></a>\_BilledSize
La colonne **\_BilledSize** spécifie la taille en octets des données qui seront facturées sur votre compte Azure si la propriété **\_IsBillable** est true.


### <a name="examples"></a>Exemples
Pour afficher la taille des événements facturables ingérés par ordinateur, utilisez la colonne `_BilledSize` qui fournit la taille en octets :

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| summarize Bytes=sum(_BilledSize) by  Computer | sort by Bytes nulls last 
```

Pour connaître la taille des événements facturables ingérés par abonnement, exécutez la requête suivante :

```Kusto
union withsource=table * 
| where _IsBillable == true 
| summarize Bytes=sum(_BilledSize) by  _SubscriptionId | sort by Bytes nulls last 
```

Pour connaître la taille des événements facturables ingérés par groupe de ressources, exécutez la requête suivante :

```Kusto
union withsource=table * 
| where _IsBillable == true 
| parse _ResourceId with "/subscriptions/" SubscriptionId "/resourcegroups/" ResourceGroupName "/" *
| summarize Bytes=sum(_BilledSize) by  _SubscriptionId, ResourceGroupName | sort by Bytes nulls last 

```


Pour connaître le nombre d'événements ingérés par ordinateur, utilisez la requête suivante :

```Kusto
union withsource = tt *
| summarize count() by Computer | sort by count_ nulls last
```

Pour connaître le nombre d'événements facturables ingérés par ordinateur, utilisez la requête suivante : 

```Kusto
union withsource = tt * 
| where _IsBillable == true 
| summarize count() by Computer  | sort by count_ nulls last
```

Pour connaître le nombre de types de données facturables issues d’un ordinateur spécifique, exécutez la requête suivante :

```Kusto
union withsource = tt *
| where Computer == "computer name"
| where _IsBillable == true 
| summarize count() by tt | sort by count_ nulls last 
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur le stockage des [données de journal Azure Monitor](./log-query-overview.md).
- Obtenez une leçon sur [l'écriture de requêtes de journaux](./get-started-queries.md).
- Obtenez une leçon sur l’[association de tables dans les requêtes de journaux](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#joins).
