---
title: Performances et mise à l’échelle dans Fonctions durables - Azure
description: Découvrez les caractéristiques de mise à l’échelle uniques de l’extension de Durable Functions pour Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 05/13/2021
ms.author: azfuncdf
ms.openlocfilehash: 179eb4c5c380a65374143cf24b2f960458fe62b5
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/16/2021
ms.locfileid: "112240438"
---
# <a name="performance-and-scale-in-durable-functions-azure-functions"></a>Performances et mise à l’échelle dans Fonctions durables (Azure Functions)

Pour optimiser les performances et l’évolutivité, il est important de comprendre les caractéristiques uniques de mise à l’échelle de [Fonctions durables](durable-functions-overview.md).

## <a name="azure-storage-provider"></a>Fournisseur Stockage Azure

La configuration par défaut pour Durable Functions stocke cet état d’exécution dans un compte Stockage Azure (classique). Toutes les exécutions de fonction sont engendrées par des files d’attente Stockage Azure. L’historique et l’état de l’entité et de l’orchestration sont stockés dans des tables Azure. Les objets blob Azure et les baux d’objets blob sont utilisés pour distribuer des instances d’orchestration et des entités entre plusieurs instances d’application (également appelées *Workers* ou simplement *machines virtuelles*). Cette section décrit plus en détail les différents artefacts de Stockage Azure et leur impact sur les performances et l’évolutivité.

> [!NOTE]
> Ce document se concentre principalement sur les caractéristiques de performances et d’évolutivité de Durable Functions à l’aide du fournisseur Stockage Azure par défaut. Toutefois, d’autres fournisseurs de stockage sont également disponibles. Pour plus d’informations sur les options de fournisseur de stockage prises en charge pour Durable Functions et leur comparaison, consultez la documentation sur les [fournisseurs de stockage Durable Functions](durable-functions-storage-providers.md).

### <a name="history-table"></a>Table d’historique

La table d’**historique** est une table de stockage Azure qui contient les événements d’historique de toutes les instances d’orchestration au sein d’un hub de tâches. Le nom de la table apparaît sous la forme *TaskHubName* History. Au fur et à mesure que des instances sont exécutées, de nouvelles lignes sont ajoutées à cette table. La clé de partition de la table provient de l’ID d’instance de l’orchestration. Par défaut, les ID d’instance sont aléatoires, ce qui garantit une distribution optimale des partitions internes dans Stockage Azure. La clé de ligne de ce tableau est un numéro de séquence utilisé pour classer les événements d’historique.

Lorsqu’une instance d’orchestration doit être exécutée, les lignes correspondantes de la table d’historique sont chargées en mémoire à l’aide d’une requête de plage dans une partition de table unique. Ces *événements d’historique* sont ensuite relus dans le code de fonction d’orchestrateur pour revenir à l’état contrôlé précédemment. L’utilisation de l’historique d’exécution permettant de régénérer l’état de cette façon est influencée par le [modèle d’approvisionnement en événements](/azure/architecture/patterns/event-sourcing).

> [!TIP]
> Les données d’orchestration stockées dans la table d’historique incluent les charges utiles de sortie des fonctions d’activité ou orchestrator secondaire. Les charges utiles des événements externes sont également stockées dans la table d’historique. Étant donné que l’historique complet est chargé en mémoire chaque fois qu’un orchestrateur doit s’exécuter, un historique suffisamment volumineux peut entraîner une sollicitation importante de la mémoire sur une machine virtuelle donnée. La longueur et la taille de l’historique de l’orchestration peuvent être réduites en fractionnant les grandes orchestrations en plusieurs sous-orchestrations ou en réduisant la taille des sorties retournées par l’activité et les fonctions orchestrator secondaires qu’elle appelle. Vous pouvez également réduire l’utilisation de la mémoire en diminuant [limitations d’accès concurrentiel](#concurrency-throttles) par machine virtuelle pour limiter le nombre d’orchestrations chargées en mémoire simultanément.

### <a name="instances-table"></a>Table d’instances

La table d’**instances** contient les états de toutes les instances d’orchestration et d’entité au sein d’un hub de tâches. Au fur et à mesure que des instances sont créées, des lignes sont ajoutées à cette table. La clé de partition de cette table est l’ID d’instance de l’orchestration ou la clé d’entité, et la clé de ligne est une chaîne vide. Il y a une ligne par instance d’orchestration ou d’entité.

Cette table est utilisée pour répondre aux [demandes de requête d’instance provenant du code](durable-functions-instance-management.md#query-instances) et aux appels de l’[API HTTP de requête d’état](durable-functions-http-api.md#get-instance-status). Sa cohérence avec le contenu de la table d’**historique** mentionnée précédemment est conservée. L’utilisation d’une table de stockage Azure distincte pour satisfaire les opérations de requête d’instance de cette façon est influencée par le [modèle de séparation des responsabilités en matière de commande et de requête (CQRS)](/azure/architecture/patterns/cqrs).

> [!TIP]
> Le partitionnement de la table *Instances* permet au service informatique de stocker des millions d’instances d’orchestration sans aucun impact notable sur les performances ou la mise à l’échelle du runtime. Toutefois, le nombre d’instances peut avoir un impact significatif sur les performances des [requêtes sur plusieurs instances](durable-functions-instance-management.md#query-all-instances). Pour contrôler la quantité de données stockées dans ces tables, pensez à [purger régulièrement les anciennes données d’instance](durable-functions-instance-management.md#purge-instance-history).

### <a name="internal-queue-triggers"></a>Déclencheurs de file d’attente interne

Les fonctions d’orchestrateur, d’entité et d’activité sont toutes déclenchées par des files d’attente internes dans le hub de tâches de l’application de fonction. Cette utilisation des files d’attente apporte des garanties de livraison de messages « au moins une fois » fiables. Il existe deux types de file d’attente dans Fonctions durables : la **file d’attente de contrôle** et la **file d’attente des éléments de travail**.

#### <a name="the-work-item-queue"></a>File d’attente des éléments de travail

Il existe une file d’attente des éléments de travail par hub de tâches dans Fonctions durables. Il s’agit d’une file d’attente de base qui se comporte comme toute autre file d’attente `queueTrigger` dans Azure Functions. Cette file d’attente permet de déclencher des *fonctions d’activité* sans état en enlevant un message de la file d’attente à la fois. Chacun de ces messages contient des entrées de fonction d’activité et des métadonnées supplémentaires, par exemple la fonction à exécuter. Quand une application Durable Functions est étendue à plusieurs machines virtuelles, ces dernières rivalisent entre elles pour acquérir les tâches de la file d’attente des éléments de travail.

#### <a name="control-queues"></a>File(s) d’attente de contrôle

Il existe plusieurs *files d’attente de contrôle* par hub de tâches dans Fonctions durables. Une *file d’attente de contrôle* est plus sophistiquée et complexe que la file d’attente des éléments de travail. Les files d’attente de contrôle servent à déclencher les fonctions d’orchestrateur ou d’entité avec état. Dans la mesure où les instances de fonction d’orchestrateur et d’entité sont des singletons avec état, il est important que chaque orchestration ou entité soit traitée uniquement par un seul Worker à la fois. Pour appliquer cette contrainte, chaque instance de l’orchestration ou entité est affectée à une seule file d’attente de contrôle. Ces files d’attente de contrôle font l’objet d’un équilibrage de la charge entre les Workers pour que chaque file d’attente soit traitée uniquement par un seul Worker à la fois. Vous trouverez plus d’informations sur ce comportement dans les sections suivantes.

Les files d’attente de contrôle contiennent différents types de message couvrant le cycle de vie de l’orchestration, tels que des [messages de contrôle d’orchestrateurs](durable-functions-instance-management.md), des messages de *réponse* de fonctions d’activité et des messages de minuteurs. Au maximum, 32 messages seront enlevés d’une file d’attente de contrôle lors d’une seule interrogation. Ces messages contiennent des données de charge utile ainsi que des métadonnées, et notamment l’instance d’orchestration de destination. Si plusieurs messages enlevés de la file d’attente sont prévus pour la même instance d’orchestration, ils seront traités en tant que lot.

Les messages de file d’attente de contrôle sont constamment interrogés à l’aide d’un thread en arrière-plan. La taille de lot de chaque interrogation de file d’attente est contrôlée par le paramètre `controlQueueBatchSize` dans host.json et a une valeur par défaut de 32 (la valeur maximale prise en charge par les files d’attente Azure). Le nombre maximal de messages de file d’attente de contrôle prérécupérés qui sont mis en mémoire tampon est contrôlé par le paramètre `controlQueueBufferThreshold` dans host.json. La valeur par défaut de `controlQueueBufferThreshold` varie en fonction de divers facteurs, notamment le type de plan d’hébergement. Pour plus d’informations sur ces paramètres, consultez la documentation du [schéma host.json](../functions-host-json.md#durabletask).

> [!TIP]
> L’augmentation de la valeur de `controlQueueBufferThreshold` permet à une seule orchestration ou entité de traiter des événements plus rapidement. Toutefois, l’augmentation de cette valeur peut également entraîner une utilisation de la mémoire plus élevée. Plus l’utilisation de la mémoire est importante, plus le nombre de messages dans la file d’attente est important, en partie en raison de la récupération d’un plus grand nombre d’historiques d’orchestration dans la mémoire. La réduction de la valeur de `controlQueueBufferThreshold` peut donc être un moyen efficace de réduire l’utilisation de la mémoire.

#### <a name="queue-polling"></a>Interrogation de file d'attente

L’extension Tâche durable implémente un algorithme d’interruption exponentiel et aléatoire pour réduire l’effet de l’interrogation de file d’attente inactive sur les coûts de transactions de stockage. Lorsqu’un message est trouvé, le moteur d’exécution recherche immédiatement un autre message. Lorsqu’aucun message n’est trouvé, il attend un certain temps avant de réessayer. Après plusieurs échecs de tentatives d’obtention d’un message de file d’attente, le temps d’attente continue à augmenter jusqu’à ce qu’il atteigne le délai d’attente maximal par défaut (30 secondes).

Le délai maximal d’interrogation est configurable via la propriété `maxQueuePollingInterval` dans le [fichier host.json](../functions-host-json.md#durabletask). Une valeur plus élevée de cette propriété peut entraîner une plus grande latence lors du traitement des messages. Les latences plus élevées ne devraient survenir qu'après des périodes d'inactivité. Une valeur plus faible de cette propriété peut entraîner une [augmentation des coûts de stockage](durable-functions-billing.md#azure-storage-transactions) en raison d’un nombre plus important de transactions de stockage.

> [!NOTE]
> Lorsqu’il est exécuté dans les plans Consommation et Premium d’Azure Functions, le [contrôleur de mise à l'échelle Azure Functions](../event-driven-scaling.md) interrogera chaque contrôle et chaque file d'attente des éléments de travail toutes les 10 secondes. Cette interrogation supplémentaire est nécessaire pour déterminer quand activer les instances d'application de fonction et pour prendre des décisions de mise à l'échelle. Au moment d’écrire ces lignes, cet intervalle de 10 secondes est constant et ne peut pas être configuré.

#### <a name="orchestration-start-delays"></a>Retards de début de l’orchestration
Les instances d’orchestration sont démarrées en plaçant un message `ExecutionStarted` dans l’une des files d’attente de contrôle du hub de tâches. Dans certaines conditions, vous pouvez observer des délais de plusieurs secondes entre le moment où l’exécution d’une orchestration est planifiée et le moment où elle commence à s’exécuter. Pendant ce laps de temps, l’instance d’orchestration reste dans l’état `Pending`. Il existe deux causes possibles pour ce retard :

1. **Files d’attente de contrôle en backlog** : Si la file d’attente de contrôle de cette instance contient un grand nombre de messages, cela peut prendre du temps avant que le message `ExecutionStarted` ne soit reçu et traité par le runtime. Les backlogs de messages peuvent se produire lorsque des orchestrations traitent un grand nombre d’événements simultanément. Les événements qui se trouvent dans la file d’attente de contrôle incluent les événements de début d’orchestration, les saisies semi-automatiques d’activité, les minuteurs durables, l’achèvement et les événements externes. Si ce retard se produit dans des circonstances normales, envisagez de créer un hub de tâches avec un plus grand nombre de partitions. La configuration d’un plus grand nombre de partitions entraîne la création par le runtime d’autres files d’attente de contrôle pour la distribution de la charge. Chaque partition correspond à une relation de type 1:1 avec une file d’attente de contrôle et 16 partitions au maximum.

2. **Retards d’interrogation de secours** : Une autre cause courante des retards d’orchestration est le [comportement d’interrogation de secours précédemment décrit pour les files d’attente de contrôle](#queue-polling). Toutefois, ce retard est attendu uniquement en cas de Scale-out d’une application sur plusieurs instances. S’il n’existe qu’une seule instance d’application ou si l’instance d’application qui démarre l’orchestration est également la même instance qui interroge la file d’attente de contrôle cible, il n’y aura pas de retard d’interrogation de la file d’attente. Les retards d’interrogation de secours peuvent être réduits en mettant à jour les paramètres **host.json**, comme décrit précédemment.

### <a name="storage-account-selection"></a>Sélection du compte de stockage

Les files d’attente, les tables et les objets blob utilisés par Durable Functions sont créés dans un compte de stockage Azure configuré. Le compte à utiliser peut être spécifié à l’aide du paramètre `durableTask/storageProvider/connectionStringName`(ou du paramètre `durableTask/azureStorageConnectionStringName` dans Durable Functions 1.x) dans le fichier **host.json**.

#### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "connectionStringName": "MyStorageAccountAppSetting"
      }
    }
  }
}
```

#### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "azureStorageConnectionStringName": "MyStorageAccountAppSetting"
    }
  }
}
```

S’il n’est pas spécifié, le compte de stockage `AzureWebJobsStorage` par défaut est utilisé. Pour les charges de travail sensibles aux performances, il est toutefois recommandé de configurer un compte de stockage non défini par défaut. Fonctions durables utilise beaucoup le stockage Azure, et l’utilisation d’un compte de stockage dédié distingue l’usage du stockage Fonctions durables et l’usage interne par l’hôte Azure Functions.

> [!NOTE]
> Des comptes Stockage Azure standard à usage général sont requis lors de l’utilisation du fournisseur Stockage Azure. Tous les autres types de compte de stockage ne sont pas pris en charge. Nous vous recommandons vivement d’utiliser des comptes de stockage à usage général v1 hérités pour Durable Functions. Les comptes de stockage v2, plus récents, peuvent être beaucoup plus onéreux pour les charges de travail de Durable Functions. Pour plus d’informations sur les types de comptes de stockage Azure, consultez l’article [Vue d’ensemble du compte de stockage](../../storage/common/storage-account-overview.md).

### <a name="orchestrator-scale-out"></a>Mise à l’échelle de l’orchestrateur

Bien qu’il soit possible d’effectuer un scale-out à l’infini des fonctions d’activité en ajoutant plus de machines virtuelles de manière élastique, chaque instance d’orchestrateur et entité est contrainte d’occuper une seule partition, et le nombre maximal de partitions est limité par le paramètre `partitionCount` dans votre `host.json`. 

> [!NOTE]
> En règle générale, les fonctions d’orchestrateur sont conçues pour être légères et elles ne requièrent pas une grande puissance de calcul. Il n’est donc pas nécessaire de créer un grand nombre de partitions de file d’attente de contrôle pour obtenir un haut débit pour les orchestrations. Le travail lourd doit être principalement effectué dans les fonctions d’activité sans état, qui peuvent être mises à l’échelle à l’infini.

Le nombre de files d’attente de contrôle est défini dans le fichier **host.json**. L’exemple suivant d’extrait de code host.json définit la propriété `durableTask/storageProvider/partitionCount` (ou `durableTask/partitionCount`dans Durable Functions 1.x) sur `3`. Notez qu’il existe autant de files d’attente de contrôle que de partitions.

#### <a name="durable-functions-2x"></a>Durable Functions 2.x

```json
{
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "partitionCount": 3
      }
    }
  }
}
```

#### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "partitionCount": 3
    }
  }
}
```

Un hub de tâches peut être configuré avec 1 à 16 partitions. Si ce paramètre n’est pas spécifié, le nombre de partitions par défaut s’élève à **4**.

Dans les scénarios à faible trafic, votre application fait l’objet d’un scale-in. Ainsi, les partitions sont gérées par un petit nombre de Workers. Prenons l’exemple du diagramme ci-dessous.

![Diagramme des orchestrations d’un scale-in](./media/durable-functions-perf-and-scale/scale-progression-1.png)

Dans le diagramme précédent, nous voyons que la charge des orchestrateurs 1 à 6 est équilibrée entre les partitions. De même, les partitions, tout comme les activités, font l’objet d’un équilibrage de charge entre les Workers. La charge des partitions est équilibrée entre les Workers, quel que soit le nombre d’orchestrateurs qui démarrent.

Si vous utilisez les plans Consommation ou Élastique Premium d’Azure Functions, ou si la mise à l’échelle automatique basée sur l’équilibrage de charge est configurée, davantage de Workers sont alloués au fur et à mesure que le trafic augmente, ce qui finit par entraîner un équilibrage de la charge entre tous les Workers. Si nous continuons le scale-out, chaque partition finit par être gérée par un seul Worker. En revanche, les activités continuent de faire l’objet d’un équilibrage de charge parmi tous les Workers. Cela est illustré dans l’image ci-dessous.

![Premier diagramme d’orchestrations faisant l’objet d’un scale-out](./media/durable-functions-perf-and-scale/scale-progression-2.png)

La limite supérieure du nombre maximal d’orchestrations _actives_ simultanées à *un moment donné* est égale au nombre de Workers alloués à votre application _multiplié_ par la valeur de `maxConcurrentOrchestratorFunctions`. Cette limite supérieure peut être rendue plus précise quand vos partitions font l’objet d’un scale-out complet parmi les Workers. Quand un scale-out complet est effectué et que chaque Worker n’a qu’une seule instance d’hôte Functions, le nombre maximal d’instances d’orchestrateur simultanées _actives_ est égal au nombre de partitions _multiplié_ par la valeur de `maxConcurrentOrchestratorFunctions`.

> [!NOTE]
> Dans ce contexte, *active* signifie qu’une orchestration ou une entité est chargée en mémoire et traite *de nouveaux événements*. Si l’orchestration ou l’entité attend un plus grand nombre d’événements, comme la valeur de retour d’une fonction d’activité, elle est déchargée de la mémoire et n’est plus considérée comme *active*. Les orchestrations et les entités sont ensuite rechargées dans la mémoire uniquement lorsqu’il y a de nouveaux événements à traiter. Il n’existe aucun nombre maximal pratique d’orchestrations ou d’entités *totales* pouvant être exécutées sur une seule machine virtuelle, même si elles sont toutes à l’état « En cours d’exécution ». La seule limitation est le nombre d’instances d’entité ou d’orchestration *actives simultanément*.

L’image ci-dessous illustre un scénario de scale-out complet où des orchestrateurs supplémentaires sont ajoutés. Certains d’entre eux sont inactifs, et sont représentés en gris.

![Deuxième diagramme d’orchestrations faisant l’objet d’un scale-out](./media/durable-functions-perf-and-scale/scale-progression-3.png)

Durant le scale-out, des baux de file d’attente de contrôle peuvent être redistribués parmi les instances d’hôte Functions pour garantir une distribution uniforme des partitions. Ces baux sont implémentés de manière interne en tant que baux du service Stockage blob Azure. Ils permettent de garantir que chaque instance de l’orchestration ou entité s’exécute sur une seule instance d’hôte à la fois. Si un hub de tâches est configuré avec trois partitions (et donc trois files d’attente de contrôle), les instances de l’orchestration et les entités peuvent faire l’objet d’un équilibrage de charge sur les trois instances d’hôte détenant un bail. Il est possible d’ajouter des machines virtuelles supplémentaires pour augmenter la capacité d’exécution de la fonction d’activité.

Le diagramme suivant illustre la façon dont l’hôte Azure Functions interagit avec les entités de stockage dans un environnement mis à l’échelle.

![Diagramme de mise à l’échelle](./media/durable-functions-perf-and-scale/scale-interactions-diagram.png)

Comme indiqué dans le diagramme précédent, toutes les machines virtuelles sont en concurrence pour les messages de la file d’attente des éléments de travail. Toutefois, seules trois machines virtuelles peuvent acquérir les messages des files d’attente de contrôle, et chacune d’elles verrouille une file d’attente de contrôle.

Les instances d’orchestration et les entités sont réparties sur toutes les instances de file d’attente de contrôle. La répartition s’effectue par hachage de l’ID d’instance de l’orchestration ou de la paire nom/clé de l’entité. Les ID d’instance d’orchestration par défaut sont des GUID aléatoires, ce qui garantit la bonne répartition des instances sur toutes les files d’attente de contrôle.

En règle générale, les fonctions d’orchestrateur sont conçues pour être légères et elles ne requièrent pas une grande puissance de calcul. Il n’est donc pas nécessaire de créer un grand nombre de partitions de file d’attente de contrôle pour obtenir un haut débit pour les orchestrations. Le travail lourd doit être principalement effectué dans les fonctions d’activité sans état, qui peuvent être mises à l’échelle à l’infini.

### <a name="auto-scale"></a>Mise à l’échelle automatique

Comme avec toutes les Azure Functions exécutées dans les plans Consommation et Élastique Premium, Durable Functions prend en charge la mise à l’échelle automatique par le biais du [contrôleur de mise à l’échelle Azure Functions](../event-driven-scaling.md#runtime-scaling). Le contrôleur de mise à l’échelle surveille la latence de toutes les files d’attente en émettant régulièrement des commandes _peek_. Selon la latence des messages parcourus, le contrôleur de mise à l’échelle décide d’ajouter ou de supprimer des machines virtuelles.

Si le contrôleur de mise à l’échelle détermine que la latence des messages de file d’attente de contrôle est trop élevée, il ajoute des instances de machine virtuelle jusqu’à ce que la latence des messages diminue à un niveau acceptable ou qu’il atteigne le nombre de partitions de file d’attente de contrôle. De même, le contrôleur de mise à l’échelle ajoute continuellement des instances de machine virtuelle si le temps de latence de file d’attente des éléments de travail est élevé, quel que soit le nombre de partitions.

> [!NOTE]
> À partir de Durable Functions 2.0, les applications de fonction peuvent être configurées de sorte qu’elles s’exécutent au sein des points de terminaison de service protégés par VNET dans le plan Élastique Premium. Dans cette configuration, Durable Functions déclenche des demandes de mise à l’échelle initiées au lieu du contrôleur de mise à l’échelle. Pour plus d’informations, consultez [Surveillance de la mise à l’échelle du runtime](../functions-networking-options.md#premium-plan-with-virtual-network-triggers).

## <a name="thread-usage"></a>Utilisation de threads

Les fonctions d’orchestrateur sont exécutées sur un seul thread pour garantir la possibilité d’une exécution déterministe entre les nombreuses relectures. Compte tenu de cette exécution monothread, il est important que les threads de fonction d’orchestrateur n’effectuent pas de tâches nécessitant une utilisation intensive du processeur, effectuent des opérations d’E/S ou se bloquent pour une raison quelconque. Tout travail susceptible de nécessiter un thread d’E/S, un blocage ou plusieurs threads doit être transféré vers les fonctions d’activité.

Les fonctions d’activité ont les mêmes comportements que les fonctions normales déclenchées par une file d’attente. Elles peuvent effectuer en toute sécurité des opérations d’E/S ou des opérations gourmandes en ressources de processeur et utiliser plusieurs threads. Étant donné que les déclencheurs d’activité sont sans état, ils peuvent librement effectuer un scale-out sur un nombre illimité de machines virtuelles.

Les fonctions d’entité sont également exécutées sur un thread unique et les opérations sont traitées une par une. Toutefois, les fonctions d’entité n’ont aucune restriction quant au type de code qui peut être exécuté.

## <a name="concurrency-throttles"></a>Limitations de la concurrence

Azure Functions prend en charge l’exécution simultanée de plusieurs fonctions dans une seule instance d’application. Cette exécution simultanée contribue à augmenter le parallélisme et réduit le nombre de démarrages à froid auxquels une application classique est confrontée au fil du temps. Toutefois, une concurrence élevée peut épuiser les ressources système de chaque machine virtuelle, telles que les connexions réseau ou la mémoire disponible. En fonction des besoins de l’application de fonction, il peut être nécessaire de limiter la concurrence par instance afin d’éviter le risque de manquer de mémoire dans les situations de charge importante.

Les limites de concurrence des fonctions d’activité, d’orchestrateur et d’entité peuvent être configurées dans le fichier **host.json**. Les paramètres pertinents sont `durableTask/maxConcurrentActivityFunctions` pour les fonctions d’activité et `durableTask/maxConcurrentOrchestratorFunctions` pour les fonctions d’orchestrateur et d’entité. Ces paramètres contrôlent le nombre maximal de fonctions d’orchestrateur, d’entité ou d’activité qui peuvent être chargées en mémoire simultanément.

### <a name="functions-20"></a>Functions 2.0

```json
{
  "extensions": {
    "durableTask": {
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10
    }
  }
}
```

### <a name="functions-1x"></a>Functions 1.x

```json
{
  "durableTask": {
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10
  }
}
```

Dans l’exemple précédent, un maximum de 10 fonctions d’orchestrateur ou d’entité et 10 fonctions d’activité peut s’exécuter simultanément sur une seule machine virtuelle. S’il n’est pas spécifié, le nombre d’exécutions simultanées de fonction d’activité et de fonction d’orchestrateur ou d’entité est limité à 10 fois le nombre de cœurs sur la machine virtuelle.

Si le nombre maximal d’activités ou d’orchestrations/entités sur une machine virtuelle de travail est atteint, le déclencheur durable attend que toutes les fonctions en cours d’exécution soient terminées ou déchargées avant de démarrer de nouvelles exécutions de fonctions.

> [!NOTE]
> Ces paramètres permettent de gérer l’utilisation de la mémoire et du processeur sur une seule machine virtuelle. Toutefois, lors de la montée en chaque sur plusieurs machines virtuelles, chaque machine virtuelle a son propre ensemble de limites défini. Ces paramètres ne peuvent pas être utilisés pour contrôler la concurrence au niveau global.

> [!NOTE]
> Les orchestrations et les entités sont chargées dans la mémoire uniquement lorsqu’elles traitent activement des événements ou des opérations. Après avoir exécuté leur logique et attendu (par exemple pour une instruction `await` (C#) ou `yield` (JavaScript, Python) dans le code de la fonction d’orchestrateur), elles sont déchargées de la mémoire. Les orchestrations et les entités qui sont déchargées de la mémoire ne sont pas comptabilisées dans la limitation `maxConcurrentOrchestratorFunctions`. Même si des millions d’orchestrations ou d’entités sont dans l’état « En cours d’exécution », elles n’entrent en compte que dans le calcul de la limite lorsqu’elles sont chargées dans la mémoire active. Une orchestration qui planifie une fonction d’activité de la même manière ne compte pas dans le calcul de la limite si l’orchestration attend la fin de l’exécution de l’activité.

### <a name="language-runtime-considerations"></a>Considérations relatives au Language Runtime

Le runtime du langage que vous sélectionnez peut imposer des restrictions d’accès concurrentiel strictes sur vos fonctions. Par exemple, les applications Durable Functions écrites en Python ou PowerShell peuvent uniquement prendre en charge l’exécution d’une seule fonction à la fois sur une seule machine virtuelle. Cela peut entraîner des problèmes de performances significatifs si elles ne sont pas soigneusement prises en compte. Par exemple, si un orchestrateur s’étend sur 10 activités mais que le runtime du langage limite l’accès concurrentiel à une seule fonction, alors 9 des 10 fonctions d’activité sont bloquées en attendant une chance de s’exécuter. En outre, ces 9 activités bloquées ne peuvent pas être équilibrées en mémoire pour les autres threads, car le runtime Durable Functions les aura déjà chargées en mémoire. Cela devient particulièrement problématique si les fonctions d’activité sont de longue durée.

Si le runtime de langage que vous utilisez impose une restriction sur l’accès concurrentiel, vous devez mettre à jour les paramètres de concurrence Durable Functions pour qu’ils correspondent aux paramètres de concurrence du runtime de votre langage. Cela permet de s’assurer que le runtime Durable Functions n’essaie pas d’exécuter simultanément plus de fonctions que ce qui est autorisé par le runtime de langage, ce qui permet d’équilibrer la charge de toutes les activités en attente sur d’autres machines virtuelles. Par exemple, si vous avez une application Python qui restreint l’accès concurrentiel à 4 fonctions (peut-être qu’elle est seulement configurée avec 4 threads sur un processus de travail du langage ou 1 thread sur 4 processus de travail du langage), vous devez configurer à la fois `maxConcurrentOrchestratorFunctions` et `maxConcurrentActivityFunctions` sur 4.

Pour plus d’informations et pour obtenir des recommandations sur les performances de Python, consultez [Améliorer les performances de débit des applications Python dans Azure Functions](../python-scale-performance-reference.md). Les techniques mentionnées dans cette documentation de référence pour les développeurs Python peuvent avoir un impact important sur les performances et l’évolutivité de Durable Functions.

## <a name="extended-sessions"></a>Sessions étendues

Les sessions étendues sont un paramètre qui conserve les orchestrations et les entités en mémoire même après qu’elles aient terminé le traitement des messages. Généralement, l’activation des sessions étendues entraîne la baisse des E/S sur le magasin durable sous-jacent et un meilleur débit global.

Vous pouvez activer les sessions étendues en paramétrant `durableTask/extendedSessionsEnabled` sur `true` dans le fichier **host.json**. Le paramètre `durableTask/extendedSessionIdleTimeoutInSeconds` peut être utilisé pour contrôler la durée pendant laquelle une session inactive sera conservée en mémoire :

**Functions 2.0**
```json
{
  "extensions": {
    "durableTask": {
      "extendedSessionsEnabled": true,
      "extendedSessionIdleTimeoutInSeconds": 30
    }
  }
}
```

**Functions 1.0**
```json
{
  "durableTask": {
    "extendedSessionsEnabled": true,
    "extendedSessionIdleTimeoutInSeconds": 30
  }
}
```

Ce paramètre présente deux inconvénients potentiels à connaître :

1. Il y a une augmentation globale de l’utilisation de la mémoire de l’application de fonction, car les instances inactives ne sont pas déchargées de la mémoire aussi rapidement.
2. Le débit peut diminuer globalement s’il existe plusieurs exécutions simultanées, distinctes et de courte durée de fonctions d’orchestrateur et d’entité.

Par exemple, si `durableTask/extendedSessionIdleTimeoutInSeconds` est défini sur 30 secondes, un épisode de fonction d’orchestrateur ou d’entité de courte durée qui s’exécute en moins de 1 seconde occupe tout de même la mémoire pendant 30 secondes. Il est également comptabilisé dans le quota `durableTask/maxConcurrentOrchestratorFunctions` mentionné précédemment et empêche potentiellement l’exécution d’autres fonctions d’orchestrateur ou d’entité.

Les effets spécifiques des sessions étendues sur les fonctions d’orchestrateur et d’entité sont décrits dans les sections suivantes.

> [!NOTE]
> Les sessions étendues ne sont actuellement prises en charge que dans les langages .NET, comme C# ou F#. La définition de `extendedSessionsEnabled` sur `true` pour d’autres plateformes peut entraîner des problèmes d’exécution, tels que l’exécution en mode silencieux de l’activité et des fonctions déclenchées par orchestration.

> [!NOTE]
> La prise en charge des sessions étendues peut varier selon le [fournisseur de stockage Durable Functions que vous utilisez](durable-functions-storage-providers.md). Consultez la documentation du fournisseur de stockage pour savoir s’il prend en charge les sessions étendues.

### <a name="orchestrator-function-replay"></a>Relecture de la fonction d’orchestrateur

Comme mentionné précédemment, les fonctions d’orchestrateur sont relues à l’aide du contenu de la table d’**historique**. Par défaut, le code de fonction d’orchestrateur est relu chaque fois qu’un lot de messages est enlevé d’une file d’attente de contrôle. Même si vous utilisez le modèle fan-out/fan-in et vous attendez à ce que toutes les tâches soient terminées (par exemple, en utilisant `Task.WhenAll()` en .NET, `context.df.Task.all()` en JavaScript ou `context.task_all()` en Python), des relectures se produisent à mesure que des lots de réponses de tâche sont traités. Lorsque des sessions étendues sont activées, les instances de fonction d’orchestrateur sont conservées en mémoire plus longtemps et les nouveaux messages peuvent être traités sans une relecture complète de l’historique.

L’amélioration des performances des sessions étendues est le plus souvent observée dans les situations suivantes :

* Lorsqu’il existe un nombre limité d’instances d’orchestration s’exécutant simultanément.
* Lorsque des orchestrations ont un grand nombre d’actions séquentielles (par exemple, des centaines d’appels de fonction d’activité) qui se terminent rapidement.
* Lorsque des orchestrations exécutent une distribution ramifiée (fan-out) et un traitement de réponse parallèle (fan-in) sur un grand nombre d’actions qui se terminent à peu près au même moment.
* Lorsque des fonctions d’orchestrateur doivent traiter des messages volumineux ou effectuer un traitement de données gourmand en ressources UC.

Dans toutes les autres situations, il n’y a généralement pas d’amélioration notable des performances pour les fonctions d’orchestrateur.

> [!NOTE]
> Ces paramètres doivent être utilisés uniquement après le développement et le test complets d’une fonction d’orchestrateur. Le comportement de réexécution agressif par défaut peut être utile pour détecter les violations des [contraintes du code de la fonction d’orchestrateur](durable-functions-code-constraints.md) lors du développement, et est donc désactivé par défaut.

### <a name="entity-function-unloading"></a>Déchargement des fonctions d’entité

Les fonctions d’entité traitent jusqu’à 20 opérations dans un lot unique. Dès qu’une entité termine le traitement d’un lot d’opérations, elle conserve son état et se décharge de la mémoire. Vous pouvez retarder le déchargement d’entités de la mémoire à l’aide du paramètre des sessions étendues. Les entités continuent à conserver leurs modifications d’état, comme auparavant, mais elles restent en mémoire pendant la durée configurée pour réduire le nombre de charges à partir du stockage. Cette réduction des charges à partir du stockage peut améliorer le débit global des entités fréquemment sollicitées.

## <a name="performance-targets"></a>Cibles de performance

Lors de la planification de l’utilisation de Fonctions durables pour une application de production, il est important de tenir compte très rapidement des exigences de performances dans le processus de planification. Cette section décrit certains scénarios d’utilisation de base et les débits maximaux attendus.

* **Exécution d’activité séquentielle** : ce scénario décrit une fonction d’orchestrateur qui exécute une série de fonctions d’activité, l’une après l’autre. Il ressemble beaucoup à l’exemple de [chaînage de fonctions](durable-functions-sequence.md).
* **Exécution d’activité parallèle** : ce scénario décrit une fonction d’orchestrateur qui exécute de nombreuses fonctions d’activité en parallèle à l’aide du modèle [fan-out/fan-in](durable-functions-cloud-backup.md).
* **Traitement de la réponse en parallèle** : ce scénario constitue la seconde moitié du modèle [fan-out/fan-in](durable-functions-cloud-backup.md). Il se concentre sur les performances du scénario fan-in. Contrairement au scénario fan-out, le scénario fan-in est réalisé par une instance de fonction d’orchestrateur unique et peut donc uniquement s’exécuter sur une seule machine virtuelle.
* **Traitement des événements externes** : ce scénario représente une instance unique de la fonction d’orchestrateur, qui attend les [événements externes](durable-functions-external-events.md), un par un.
* **Traitement des opérations d’entité** : ce scénario teste la rapidité avec laquelle une _seule_ [entité Compteur](durable-functions-entities.md) peut traiter un flux constant d’opérations.

> [!TIP]
> Contrairement aux opérations fan-out, les opérations fan-in sont limitées à une seule machine virtuelle. Si votre application utilise le modèle fan-out/fan-in et si vous vous préoccupez des performances de scénario fan-in, pensez à sous-diviser le scénario fan-out de fonction d’activité sur plusieurs [sous-orchestrations](durable-functions-sub-orchestrations.md).

### <a name="azure-storage-performance-targets"></a>Cibles de performance pour Stockage Azure

Le tableau suivant indique les nombres de débit *maximaux* attendus pour les scénarios décrits précédemment lors de l’utilisation du [fournisseur Stockage Azure par défaut pour Durable Functions](durable-functions-storage-providers.md#azure-storage). Le terme « instance » fait référence à l’instance unique d’une fonction d’orchestrateur en cours d’exécution sur une petite machine virtuelle ([A1](../../virtual-machines/sizes-previous-gen.md)) unique dans Azure App Service. Dans tous les cas, il est supposé que les [sessions étendues](#orchestrator-function-replay) sont activées. Les résultats réels peuvent varier selon le travail de l’UC ou des E/S effectué par le code de fonction.

| Scénario | Débit maximal |
|-|-|
| Exécution d’activité séquentielle | 5 activités par seconde et par instance |
| Exécution d’activité parallèle (fan-out) | 100 activités par seconde et par instance |
| Traitement de réponse parallèle (fan-in) | 150 réponses par seconde et par instance |
| Traitement d’événements externes | 50 événements par seconde et par instance |
| Traitement des opérations d’entité | 64 opérations par seconde |

Si vous n’obtenez pas les débits que vous attendiez et si l’utilisation de l’UC et de la mémoire semble correcte, vérifiez [l’intégrité de votre compte de stockage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance). L’extension Fonctions durables peut placer une charge importante sur un compte de stockage Azure, et des charges suffisamment élevées peuvent entraîner la limitation du compte de stockage.

> [!TIP]
> Dans certains cas, vous pouvez augmenter considérablement le débit des événements externes, d’entrée d’activités et des opérations d’entité en augmentant la valeur du paramètre `controlQueueBufferThreshold` dans **host.json**. Si vous augmentez cette valeur au-delà de celle par défaut, le fournisseur de stockage de Durable Task Framework utilise davantage de mémoire pour prérécupérer ces événements de manière plus agressive, ce qui réduit les retards associés à la mise en file d’attente des messages des files d’attente de contrôle de Stockage Azure. Pour plus d’informations, consultez la documentation de référence [host.json](durable-functions-bindings.md#host-json).

### <a name="high-throughput-processing"></a>Traitement à haut débit

L’architecture principale de Stockage Azure impose certaines limitations sur les performances théoriques et l’extensibilité maximales de Durable Functions. Si votre test montre que Durable Functions sur Stockage Azure ne répond pas à vos besoins de débit, vous devez plutôt envisager d’utiliser le [fournisseur de stockage Netherite pour Durable Functions](durable-functions-storage-providers.md#netherite).

Le stockage principal Netherite a été conçu et développé par [Microsoft Research](https://www.microsoft.com/research). Il utilise [Azure Event Hubs](../../event-hubs/event-hubs-about.md) et la technologie de base de données [FASTER](https://www.microsoft.com/research/project/faster/) en plus d’[objets blob de pages Azure](../../storage/blobs/storage-blob-pageblob-overview.md). La conception de Netherite permet un traitement à débit beaucoup plus élevé des orchestrations et des entités par rapport aux autres fournisseurs. Dans certains scénarios de test, le débit a augmenté d’un ordre de grandeur supérieur à celui du fournisseur Stockage Azure par défaut.

Pour plus d’informations sur les options de fournisseur de stockage prises en charge pour Durable Functions et leur comparaison, consultez la documentation sur les [fournisseurs de stockage Durable Functions](durable-functions-storage-providers.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la récupération d’urgence et la géo-distribution](durable-functions-disaster-recovery-geo-distribution.md)
