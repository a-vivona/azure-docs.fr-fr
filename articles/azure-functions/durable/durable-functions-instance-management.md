---
title: Gérer des instances dans Fonctions durables (Azure)
description: Découvrez comment gérer des instances dans l’extension Fonctions durables pour Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 05/11/2021
ms.author: azfuncdf
ms.openlocfilehash: 04c3e9f1a5c5a1a23a618f3274057a5e03a9f0e1
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122563341"
---
# <a name="manage-instances-in-durable-functions-in-azure"></a>Gérer des instances dans Durable Functions dans Azure

Les orchestrations dans Durable Functions sont des fonctions avec état à exécution longue qui peuvent être démarrées, interrogées et terminées à l’aide d’API de gestion intégrées. Plusieurs autres API de gestion d’instance sont également exposées par la [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client) Durable Functions, par exemple l’envoi d’événements externes à des instances, la purge de l’historique des instances, etc. Cet article présente les détails de toutes les opérations de gestion des instances prises en charge.

## <a name="start-instances"></a>Démarrer des instances

La méthode `StartNewAsync` (.NET), `startNew` (JavaScript) ou `start_new` (Python) sur la [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client) démarre une nouvelle instance d’orchestration. En interne, cette méthode écrit un message par le biais du [fournisseur de stockage Durable Functions](durable-functions-storage-providers.md), puis retourne. Ce message déclenche de manière asynchrone le démarrage d’une [fonction d’orchestration](durable-functions-types-features-overview.md#orchestrator-functions) avec le nom spécifié.

Les paramètres de démarrage d’une nouvelle instance d’orchestration sont les suivants :

* **Name** : Nom de la fonction orchestrator à planifier.
* **Entrée**: Toutes les données JSON sérialisables devant être passées comme entrée à la fonction orchestrator.
* **InstanceId** : (Facultatif) ID unique de l’instance. Si vous ne spécifiez pas ce paramètre, la méthode utilise un ID aléatoire.

> [!TIP]
> Utilisez un identificateur aléatoire pour l’ID d’instance quand cela est possible. Les ID d’instance aléatoires permettent de garantir une distribution égale de la charge lors de la mise à l’échelle des fonctions orchestrator sur plusieurs machines virtuelles. Le moment qui convient pour l’utilisation des ID d’instance non aléatoires correspond au moment où l’ID provient d’une source externe ou lors de l’implémentation du modèle [orchestrateur singleton](durable-functions-singletons.md).

Le code suivant est un exemple de fonction qui démarre une nouvelle instance d’orchestration :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("HelloWorldQueueTrigger")]
public static async Task Run(
    [QueueTrigger("start-queue")] string input,
    [DurableClient] IDurableOrchestrationClient starter,
    ILogger log)
{
    string instanceId = await starter.StartNewAsync("HelloWorld", input);
    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

<a name="javascript-function-json"></a>Sauf indication contraire, les exemples de cette page utilisent le déclencheur HTTP avec le function  JSON suivant.

**function.json**

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": ["post"]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

> [!NOTE]
> Cet exemple cible la version 2  x de Durable Functions. Dans la version 1. x, utilisez `orchestrationClient`au lieu de`durableClient`.

**index.js**

```javascript
const df = require("durable-functions");

module.exports = async function(context, input) {
    const client = df.getClient(context);

    const instanceId = await client.startNew("HelloWorld", undefined, input);
    context.log(`Started orchestration with ID = ${instanceId}.`);
};
```

# <a name="python"></a>[Python](#tab/python)

<a name="javascript-function-json"></a>Sauf indication contraire, les exemples de cette page utilisent le déclencheur HTTP avec le function  JSON suivant.

**function.json**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [    
    {
      "name": "msg",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages",
      "connection": "AzureStorageQueuesConnectionString"
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

> [!NOTE]
> Cet exemple cible la version 2  x de Durable Functions. Dans la version 1. x, utilisez `orchestrationClient`au lieu de`durableClient`.

**__init__.py**

```python
import logging
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    
    instance_id = await client.start_new('HelloWorld', None, None)
    logging.log(f"Started orchestration with ID = ${instance_id}.")

```

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Vous pouvez également démarrer une instance directement avec la commande [`func durable start-new` ](../functions-core-tools-reference.md#func-durable-start-new) dans les Core Tools, qui utilise les paramètres suivants :

* **`function-name` (obligatoire)**  : Nom de la fonction à démarrer.
* **`input` (facultatif)**  : Entrée de la fonction, en ligne ou par le biais d’un fichier JSON. Pour les fichiers, ajoutez un préfixe au chemin du fichier avec `@`, par exemple, `@path/to/file.json`.
* **`id` (facultatif)**  : ID de l’instance d’orchestration. Si vous ne spécifiez pas ce paramètre, la commande utilise un GUID aléatoire.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. La valeur par défaut est AzureWebJobsStorage.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. La valeur par défaut est DurableFunctionsHub. Vous pouvez également le définir dans [host.json](durable-functions-bindings.md#host-json) à l’aide de durableTask:HubName.

> [!NOTE]
> Les commandes Core Tools supposent que vous les exécutiez à partir du répertoire racine d’une application de fonction. Si vous fournissez explicitement les paramètres `connection-string-setting` et `task-hub-name`, vous pouvez exécuter les commandes à partir de n’importe quel répertoire. Même si vous pouvez exécuter ces commandes sans exécuter d’hôte d’application de fonction, vous constaterez peut-être que vous ne pouvez pas observer certains effets si l’hôte n’est pas exécuté. Par exemple, la commande `start-new` met en file d’attente un message de démarrage dans le hub de tâches cible, mais l’orchestration ne peut pas réellement s’exécuter à moins d’avoir un processus hôte d’application de fonction en cours d’exécution pouvant traiter le message.

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande suivante démarre la fonction nommée HelloWorld et lui passe le contenu du fichier `counter-data.json` :

```bash
func durable start-new --function-name HelloWorld --input @counter-data.json --task-hub-name TestTaskHub
```

## <a name="query-instances"></a>Interroger des instances

Après le démarrage de nouvelles instances d’orchestration, vous devrez probablement interroger leur état d’exécution pour savoir si elles sont en cours d’exécution, si elles ont terminé ou si elles ont échoué.

La méthode `GetStatusAsync` (.NET), `getStatus` (JavaScript) ou `get_status` (Python) sur la [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client) demande l’état d’une instance de l’orchestration.

Elle prend `instanceId` (obligatoire), `showHistory` (facultatif), `showHistoryOutput` (facultatif) et `showInput` (facultatif) comme paramètres.

* **`showHistory`**  : Si la valeur est définie sur `true`, la réponse contient l’historique d’exécution.
* **`showHistoryOutput`**  : Si la valeur est définie sur `true`, l’historique d’exécution contient les sorties de l’activité.
* **`showInput`**  : Si la valeur est définie sur `false`, la réponse ne contient pas l’entrée de la fonction. La valeur par défaut est `true`.

La méthode renvoie un objet avec les propriétés suivantes :

* **Name** : Nom de la fonction orchestrator.
* **InstanceId** : ID d’instance de l’orchestration (doit être identique à l’entrée `instanceId`).
* **CreatedTime** : Heure à laquelle la fonction orchestrator a commencé à s’exécuter.
* **LastUpdatedTime** : Heure du dernier point de contrôle d’orchestration.
* **Entrée**: Entrée de la fonction sous forme de valeur JSON. Ce champ n’est pas rempli si `showInput` est false.
* **CustomStatus** : État personnalisé de l’orchestration au format JSON.
* **Sortie**: Sortie de la fonction sous forme de valeur JSON (si cette fonction est terminée). En cas d’échec de la fonction d’orchestrateur, cette propriété inclut les détails de l’échec. En cas d’interruption de la fonction de l’orchestrateur, cette propriété indique la raison de cette interruption (s’il y en a une).
* **RuntimeStatus** : L’une des valeurs suivantes :
  * **Pending** : L’instance a été planifiée mais n’est pas encore en cours d’exécution.
  * **En cours d’exécution** : L’instance a commencé à s’exécuter.
  * **Completed** : L’instance s’est terminée normalement.
  * **ContinuedAsNew** : L’instance a redémarré automatiquement avec un nouvel historique. Cet état est temporaire.
  * **Échec** : Échec de l’instance avec une erreur.
  * **Terminé** : L’instance a été brusquement arrêtée.
* **History** : L’historique d’exécution de l’orchestration. Ce champ est renseigné uniquement si `showHistory` est défini sur `true`.

> [!NOTE]
> Un orchestrateur n’est pas marqué comme `Completed` avant que toutes ses tâches planifiées soient terminées _et_ que l’orchestrateur ait renvoyé son résultat. En d’autres termes, il n’est pas suffisant qu’un orchestrateur atteigne son instruction `return` pour qu’il soit marqué comme `Completed`. Cela s’avère particulièrement pertinent dans les cas où `WhenAny` est utilisé : ces orchestrateurs sont souvent `return` avant l’exécution de toutes les tâches planifiées.

Cette méthode renvoie `null` (.NET), `undefined` (JavaScript) ou `None` (Python) si l’instance n’existe pas.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("GetStatus")]
public static async Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("check-status-queue")] string instanceId)
{
    DurableOrchestrationStatus status = await client.GetStatusAsync(instanceId);
    // do something based on the current status.
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const status = await client.getStatus(instanceId);
    // do something based on the current status.
    // example: if status.runtimeStatus === df.OrchestrationRuntimeStatus.Running: ...
}
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str, instance_id: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    status = await client.get_status(instance_id)
    # do something based on the current status
    # example: if (existing_instance.runtime_status is df.OrchestrationRuntimeStatus.Running) { ...
```

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Il est également possible d’obtenir l’état d’une instance d’orchestration à l’aide de la commande [`func durable get-runtime-status` ](../functions-core-tools-reference.md#func-durable-get-runtime-status) dans Core Tools.

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande `durable get-runtime-status` utilise les paramètres suivants :

* **`id` (obligatoire)**  : ID de l’instance d’orchestration.
* **`show-input` (facultatif)**  : Si la valeur est définie sur `true`, la réponse contient l’entrée de la fonction. La valeur par défaut est `false`.
* **`show-output` (facultatif)**  : Si la valeur est définie sur `true`, la réponse contient la sortie de la fonction. La valeur par défaut est `false`.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, il s’agit de `DurableFunctionsHub`. Vous pouvez également le définir dans [host.json](durable-functions-bindings.md#host-json), à l’aide de durableTask:HubName.

La commande suivante récupère l’état (y compris l’entrée et la sortie) d’une instance avec l’ID d’instance d’orchestration 0ab8c55a66644d68a3a8b220b12d209c. Cela suppose que la commande `func` est exécutée à partir du répertoire racine de l’application de fonction :

```bash
func durable get-runtime-status --id 0ab8c55a66644d68a3a8b220b12d209c --show-input true --show-output true
```

Vous pouvez utiliser la commande `durable get-history` pour récupérer l’historique d’une instance d’orchestration. Les paramètres suivants sont pris en compte :

* **`id` (obligatoire)**  : ID de l’instance d’orchestration.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, il s’agit de `DurableFunctionsHub`. Vous pouvez également le définir dans host.json à l’aide de durableTask:HubName.

```bash
func durable get-history --id 0ab8c55a66644d68a3a8b220b12d209c
```

## <a name="query-all-instances"></a>Interroger toutes les instances

Vous pouvez utiliser la méthode [ListInstancesAsync](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableorchestrationclient.listinstancesasync) (.NET), [getStatusAll](/javascript/api/durable-functions/durableorchestrationclient#getstatusall--) (JavaScript) ou `get_status_all` (Python) pour demander l’état de toutes les instances de l’orchestration dans votre [hub de tâches](durable-functions-task-hubs.md). Cette méthode retourne une liste d’objets qui représentent les instances de l’orchestration correspondant aux paramètres de la requête.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("GetAllStatus")]
public static async Task Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient client,
    ILogger log)
{
    var noFilter = new OrchestrationStatusQueryCondition();
    OrchestrationStatusQueryResult result = await client.ListInstancesAsync(
        noFilter,
        CancellationToken.None);
    foreach (DurableOrchestrationStatus instance in result.DurableOrchestrationState)
    {
        log.LogInformation(JsonConvert.SerializeObject(instance));
    }
    
    // Note: ListInstancesAsync only returns the first page of results.
    // To request additional pages provide the result.ContinuationToken
    // to the OrchestrationStatusQueryCondition's ContinuationToken property.
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, req) {
    const client = df.getClient(context);

    const instances = await client.getStatusAll();
    instances.forEach((instance) => {
        context.log(JSON.stringify(instance));
    });
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import logging
import json
import azure.functions as func
import azure.durable_functions as df


async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    instances = await client.get_status_all()

    for instance in instances:
        logging.log(json.dumps(instance))
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Vous pouvez aussi interroger des instances directement à l’aide de la commande [`func durable get-instances` dans ](../functions-core-tools-reference.md#func-durable-get-instances) Core Tools.

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande `durable get-instances` utilise les paramètres suivants :

* **`top` (facultatif)**  : Cette commande prend en charge la pagination. Ce paramètre correspond au nombre d’instances récupérées par demande. La valeur par défaut est de 10.
* **`continuation-token` (facultatif)**  : Jeton qui indique la page ou la section des instances à récupérer. Chaque exécution de `get-instances` retourne un jeton à l’ensemble suivant d’instances.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, il s’agit de `DurableFunctionsHub`. Vous pouvez également le définir dans [host.json](durable-functions-bindings.md#host-json), à l’aide de durableTask:HubName.

```bash
func durable get-instances
```

## <a name="query-instances-with-filters"></a>Interroger les instances en appliquant des filtres

Il est possible que vous n’ayez pas vraiment besoin de toutes les informations fournies par une requête d’instance standard. Peut-être avez-vous seulement besoin de connaître l’heure de création de l’orchestration ou de l’état de son exécution ? Vous pouvez affiner votre requête en appliquant des filtres.

Utilisez la méthode [ListInstancesAsync](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.idurableorchestrationclient.listinstancesasync#Microsoft_Azure_WebJobs_Extensions_DurableTask_IDurableOrchestrationClient_ListInstancesAsync_Microsoft_Azure_WebJobs_Extensions_DurableTask_OrchestrationStatusQueryCondition_System_Threading_CancellationToken_) (.NET) ou [getStatusBy](/javascript/api/durable-functions/durableorchestrationclient#getstatusby-date---undefined--date---undefined--orchestrationruntimestatus---) (JavaScript) pour obtenir la liste des instances de l’orchestration correspondant à un ensemble de filtres prédéfinis.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("QueryStatus")]
public static async Task Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient client,
    ILogger log)
{
    // Get the first 100 running or pending instances that were created between 7 and 1 day(s) ago
    var queryFilter = new OrchestrationStatusQueryCondition
    {
        RuntimeStatus = new[]
        {
            OrchestrationRuntimeStatus.Pending,
            OrchestrationRuntimeStatus.Running,
        },
        CreatedTimeFrom = DateTime.UtcNow.Subtract(TimeSpan.FromDays(7)),
        CreatedTimeTo = DateTime.UtcNow.Subtract(TimeSpan.FromDays(1)),
        PageSize = 100,
    };
    
    OrchestrationStatusQueryResult result = await client.ListInstancesAsync(
        queryFilter,
        CancellationToken.None);
    foreach (DurableOrchestrationStatus instance in result.DurableOrchestrationState)
    {
        log.LogInformation(JsonConvert.SerializeObject(instance));
    }
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, req) {
    const client = df.getClient(context);

    const runtimeStatus = [
        df.OrchestrationRuntimeStatus.Completed,
        df.OrchestrationRuntimeStatus.Running,
    ];
    const instances = await client.getStatusBy(
        new Date(2021, 3, 10, 10, 1, 0),
        new Date(2021, 3, 10, 10, 23, 59),
        runtimeStatus
    );
    instances.forEach((instance) => {
        context.log(JSON.stringify(instance));
    });
};
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

```python
import logging
from datetime import datetime
import json
import azure.functions as func
import azure.durable_functions as df
from azure.durable_functions.models.OrchestrationRuntimeStatus import OrchestrationRuntimeStatus

async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    runtime_status = [OrchestrationRuntimeStatus.Completed, OrchestrationRuntimeStatus.Running]

    instances = await client.get_status_by(
        datetime(2021, 3, 10, 10, 1, 0),
        datetime(2021, 3, 10, 10, 23, 59),
        runtime_status
    )

    for instance in instances:
        logging.log(json.dumps(instance))
```

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Dans Azure Functions Core Tools, vous pouvez également utiliser la commande `durable get-instances` avec des filtres. Outre les paramètres `top`, `continuation-token`, `connection-string-setting` et `task-hub-name` mentionnés ci-dessus, vous pouvez utiliser trois paramètres de filtre (`created-after`, `created-before` et `runtime-status`).

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

Voici les paramètres de la commande `durable get-instances`.

* **`created-after` (facultatif)**  : Récupère les instances créées après cette date/heure (UTC). Les dates et heures au format ISO 8601 sont acceptées.
* **`created-before` (facultatif)**  : Récupère les instances créées avant cette date/heure (UTC). Les dates et heures au format ISO 8601 sont acceptées.
* **`runtime-status` (facultatif)**  : Récupérez les instances avec un état particulier (par exemple, En cours d’exécution ou Terminé). Peut fournir plusieurs états (séparés par des espaces).
* **`top` (facultatif)**  : Nombre d’instances récupérées par requête. La valeur par défaut est de 10.
* **`continuation-token` (facultatif)**  : Jeton qui indique la page ou la section des instances à récupérer. Chaque exécution de `get-instances` retourne un jeton à l’ensemble suivant d’instances.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, il s’agit de `DurableFunctionsHub`. Vous pouvez également le définir dans [host.json](durable-functions-bindings.md#host-json), à l’aide de durableTask:HubName.

Si vous n’appliquez pas de filtres (`created-after`, `created-before` ou `runtime-status`), la commande récupère simplement les instances `top`, sans se soucier de l’état de l’exécution ou de l’heure de création.

```bash
func durable get-instances --created-after 2021-03-10T13:57:31Z --created-before  2021-03-10T23:59Z --top 15
```

## <a name="terminate-instances"></a>Arrêter les instances

Si vous disposez d’une instance d’orchestration dont l’exécution est trop longue, ou si vous devez simplement l’arrêter avant la fin pour une raison quelconque, vous avez la possibilité de l’arrêter.

Vous pouvez utiliser la méthode `TerminateAsync` (.NET) ou `terminate` (JavaScript) ou `terminate` (Python) de la [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client) pour terminer les instances. Les deux paramètres sont une chaîne `instanceId` et `reason`, qui est écrite dans les journaux et dans l’état de l’instance.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TerminateInstance")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("terminate-queue")] string instanceId)
{
    string reason = "Found a bug";
    return client.TerminateAsync(instanceId, reason);
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const reason = "Found a bug";
    return client.terminate(instanceId, reason);
};
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str, instance_id: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    reason = "Found a bug"
    return client.terminate(instance_id, reason)
```

---

Une fois terminées, les instances passent à l’état `Terminated`. Toutefois, ce passage ne se produit pas immédiatement. En effet, l’opération de terminaison est mise en file d’attente dans le hub de tâches avec les autres opérations concernant cette instance. Vous pouvez utiliser les API d’[interrogation d’instance](#query-instances) pour savoir à quel moment une instance terminée est passée à l’état `Terminated`.

> [!NOTE]
> Il n’est pas possible de propager l’arrêt des instances. Les fonctions et sous-orchestrations de l’activité s’exécutent jusqu’à la fin, même si l’instance d’orchestration qui les a appelées a été arrêtée.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Vous pouvez aussi directement arrêter une instance d’orchestration, à l’aide de la commande [`func durable terminate` ](../functions-core-tools-reference.md#func-durable-terminate) dans Core Tools.

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande `durable terminate` utilise les paramètres suivants :

* **`id` (obligatoire)**  : ID de l’instance d’orchestration à arrêter.
* **`reason` (facultatif)**  : Raison de l’arrêt.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, il s’agit de `DurableFunctionsHub`. Vous pouvez également le définir dans [host.json](durable-functions-bindings.md#host-json), à l’aide de durableTask:HubName.

La commande suivante arrête une instance d’orchestration avec l’ID 0ab8c55a66644d68a3a8b220b12d209c :

```bash
func durable terminate --id 0ab8c55a66644d68a3a8b220b12d209c --reason "Found a bug"
```

## <a name="send-events-to-instances"></a>Envoyer des événements à des instances

Dans certains scénarios, les fonctions d’orchestrateur doivent attendre et écouter les événements externes. Les scénarios de [surveillance](durable-functions-overview.md#monitoring) et d’[interaction humaine](durable-functions-overview.md#human) sont des exemples de scénarios où cela est utile.

Vous pouvez envoyer des notifications d’événements à des instances en cours d’exécution à l’aide de la méthode `RaiseEventAsync` (.NET), `raiseEvent` (JavaScript) ou `raise_event` (Python) du [client d’orchestration](durable-functions-bindings.md#orchestration-client). Les instances pouvant gérer ces événements sont celles qui sont en attente d’un appel à `WaitForExternalEvent` (.NET), obéissant à une tâche `waitForExternalEvent` (JavaScript) ou `wait_for_external_event` (Python).

Les paramètres destinés à `RaiseEventAsync` (.NET) et à `raiseEvent` (JavaScript) sont les suivants :

* **InstanceId** : ID unique de l’instance.
* **EventName** : Nom de l’événement à envoyer.
* **EventData** : Charge utile JSON sérialisable à envoyer à l’instance.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RaiseEvent")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("event-queue")] string instanceId)
{
    int[] eventData = new int[] { 1, 2, 3 };
    return client.RaiseEventAsync(instanceId, "MyEvent", eventData);
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const eventData = [ 1, 2, 3 ];
    return client.raiseEvent(instanceId, "MyEvent", eventData);
};
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str, instance_id: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    event_data = [1, 2 ,3]
    return client.raise_event(instance_id, 'MyEvent', event_data)
```

---

> [!NOTE]
> S’il n’existe aucune instance d’orchestration avec l’ID d’instance spécifié, le message d’événement est ignoré. Si une instance existe, mais qu’elle n’attend pas encore l’événement, ce dernier est stocké dans l’état de l’instance jusqu’à ce qu’il soit prêt à être reçu et traité.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Vous pouvez aussi déclencher un événement dans une instance d’orchestration, à l’aide de la commande [`func durable raise-event` ](../functions-core-tools-reference.md#func-durable-raise-event) dans Core Tools.

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande `durable raise-event` utilise les paramètres suivants :

* **`id` (obligatoire)**  : ID de l’instance d’orchestration.
* **`event-name`**  : Nom de l’événement à déclencher.
* **`event-data` (facultatif)**  : Données à envoyer à l’instance d’orchestration. Il peut s’agir d’un chemin de fichier JSON. Vous pouvez également fournir les données directement sur la ligne de commande.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, il s’agit de `DurableFunctionsHub`. Vous pouvez également le définir dans [host.json](durable-functions-bindings.md#host-json), à l’aide de durableTask:HubName.

```bash
func durable raise-event --id 0ab8c55a66644d68a3a8b220b12d209c --event-name MyEvent --event-data @eventdata.json
```

```bash
func durable raise-event --id 1234567 --event-name MyOtherEvent --event-data 3
```

## <a name="wait-for-orchestration-completion"></a>Attendre la fin de l’orchestration

Dans les orchestrations à exécution longue, vous pouvez souhaiter attendre les résultats de l’orchestration. Dans ce cas, il est également utile de définir un délai d’attente pour l’orchestration. Si le délai d’attente est dépassé, l’état de l’orchestration doit être retourné à la place des résultats.

Vous pouvez utiliser la méthode `WaitForCompletionOrCreateCheckStatusResponseAsync` (.NET), `waitForCompletionOrCreateCheckStatusResponse` (JavaScript) ou `wait_for_completion_or_create_check_status_response` (Python) pour obtenir la sortie réelle d’une instance d’orchestration de façon synchrone. Par défaut, ces méthodes utilisent la valeur par défaut de 10 secondes pour `timeout` et de 1 seconde pour `retryInterval`.  

Voici un exemple de fonction de déclencheur HTTP qui montre comment utiliser cette API :

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpSyncStart.cs)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/HttpSyncStart/index.js)]

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

```python
import logging
import azure.functions as func
import azure.durable_functions as df

timeout = "timeout"
retry_interval = "retryInterval"

async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    instance_id = await client.start_new(req.route_params['functionName'], None, req.get_body())
    logging.log(f"Started orchestration with ID = '${instance_id}'.")

    timeout_in_milliseconds = get_time_in_seconds(req, timeout)
    timeout_in_milliseconds = timeout_in_milliseconds if timeout_in_milliseconds != None else 30000
    retry_interval_in_milliseconds = get_time_in_seconds(req, retry_interval)
    retry_interval_in_milliseconds = retry_interval_in_milliseconds if retry_interval_in_milliseconds != None else 1000

    return client.wait_for_completion_or_create_check_status_response(
        req,
        instance_id,
        timeout_in_milliseconds,
        retry_interval_in_milliseconds
    )

def get_time_in_seconds(req: func.HttpRequest, query_parameter_name: str):
    query_value = req.params.get(query_parameter_name)
    return query_value if query_value != None else 1000
```

---

Appelez la fonction avec la ligne suivante. Utilisez une valeur de 2 secondes pour le délai d’attente et de 0,5 seconde pour l’intervalle avant nouvelle tentative :

```bash
curl -X POST "http://localhost:7071/orchestrators/E1_HelloSequence/wait?timeout=2&retryInterval=0.5"
```

Selon le temps nécessaire pour obtenir la réponse de l’instance d’orchestration, il existe deux cas de figure :

* Les instances d’orchestration sont terminées dans le délai imparti (dans ce cas, 2 secondes). La réponse est la sortie d’instance d’orchestration qui est remise de manière synchrone :

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Thu, 14 Dec 2021 06:14:29 GMT
Transfer-Encoding: chunked

[
    "Hello Tokyo!",
    "Hello Seattle!",
    "Hello London!"
]
```

* Les instances d’orchestration ne peuvent pas être terminées dans le délai imparti. La réponse est celle par défaut qui est décrite dans [Découverte de l’URL de l’API HTTP](durable-functions-http-api.md) :

```http
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Date: Thu, 14 Dec 2021 06:13:51 GMT
Location: http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177?taskHub={taskHub}&connection={connection}&code={systemKey}
Retry-After: 10
Transfer-Encoding: chunked

{
    "id": "d3b72dddefce4e758d92f4d411567177",
    "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177/raiseEvent/{eventName}?taskHub={taskHub}&connection={connection}&code={systemKey}",
    "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177?taskHub={taskHub}&connection={connection}&code={systemKey}",
    "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177/terminate?reason={text}&taskHub={taskHub}&connection={connection}&code={systemKey}"
}
```

> [!NOTE]
> Le format des URL webhook peut varier selon la version de l’hôte Azure Functions que vous exécutez. L’exemple précédent utilise l’hôte Azure Functions 3.0.

## <a name="retrieve-http-management-webhook-urls"></a>Récupérer les URL webhook de gestion HTTP

Vous pouvez utiliser un système externe pour superviser ou déclencher des événements au sein d’une orchestration. Les systèmes externes peuvent communiquer avec Durable Functions via les URL webhook qui font partie de la réponse par défaut décrite dans [Découverte de l’URL de l’API HTTP](durable-functions-http-features.md#http-api-url-discovery). Vous pouvez également accéder par programme aux URL du webhook à l’aide de la [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client). Vous pouvez utiliser les méthodes `CreateHttpManagementPayload` (.NET) ou `createHttpManagementPayload` (JavaScript) pour obtenir un objet sérialisable qui contient ces URL de webhook.

Les méthodes `CreateHttpManagementPayload` (.NET) et `createHttpManagementPayload` (JavaScript) incluent un paramètre :

* **instanceId** : ID unique de l’instance.

Les méthodes renvoient un objet avec les propriétés de chaîne suivantes :

* **Id** : ID d’instance de l’orchestration (doit être identique à l’entrée `InstanceId`).
* **StatusQueryGetUri** : L’URL de l’état de l’instance d’orchestration.
* **SendEventPostUri** : L’URL « Raise event » de l’instance d’orchestration.
* **TerminatePostUri** : L’URL « d’arrêt » de l’instance d’orchestration.
* **PurgeHistoryDeleteUri** : URL de « purge de l’historique » de l’instance d’orchestration.

Les fonctions peuvent envoyer des instances de ces objets à des systèmes externes pour surveiller ou déclencher des événements sur les orchestrations correspondantes, comme indiqué dans les exemples suivants :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("SendInstanceInfo")]
public static void SendInstanceInfo(
    [ActivityTrigger] IDurableActivityContext ctx,
    [DurableClient] IDurableOrchestrationClient client,
    [DocumentDB(
        databaseName: "MonitorDB",
        collectionName: "HttpManagementPayloads",
        ConnectionStringSetting = "CosmosDBConnection")]out dynamic document)
{
    HttpManagementPayload payload = client.CreateHttpManagementPayload(ctx.InstanceId);

    // send the payload to Cosmos DB
    document = new { Payload = payload, id = ctx.InstanceId };
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `DurableActivityContext` au lieu de l’attribut `IDurableActivityContext`, et vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et devez utiliser le type de paramètre `DurableOrchestrationClient` et non `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

modules.exports = async function(context, ctx) {
    const client = df.getClient(context);

    const payload = client.createHttpManagementPayload(ctx.instanceId);

    // send the payload to Cosmos DB
    context.bindings.document = JSON.stringify({
        id: ctx.instanceId,
        payload,
    });
};
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str, instance_id: str) -> func.cosmosdb.cdb.Document:
    client = df.DurableOrchestrationClient(starter)

    payload = client.create_check_status_response(req, instance_id).get_body().decode()

    return func.cosmosdb.CosmosDBConverter.encode({
        id: instance_id,
        payload: payload
    })
```
---

## <a name="rewind-instances-preview"></a>Rembobiner des instances (préversion)

Si une orchestration échoue pour une raison quelconque, vous pouvez *rembobiner* l’instance vers un état sain précédent à l’aide d’une API intégrée à cet effet.

> [!NOTE]
> Cette API n’est pas destinée à se substituer à des stratégies appropriées de nouvelles tentatives et de gestion des erreurs. Son utilisation est plutôt uniquement réservée dans les cas où les instances d’orchestration échouent pour des raisons inattendues. Pour en savoir plus sur les stratégies de gestion des nouvelles tentatives et des erreurs, consultez l’article [Gestion des erreurs](durable-functions-error-handling.md).

Utilisez la méthode `RewindAsync` (.NET) ou `rewind` (JavaScript) de la [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client) pour replacer l’orchestration à l’état *Exécution en cours*. Cette méthode réexécute également l’activité ou la sous-orchestration qui a échoué et provoqué l’échec de l’orchestration.

Par exemple, supposons que vous ayez un workflow impliquant une suite d’[approbations humaines](durable-functions-overview.md#human). Supposons que vous ayez une suite de fonctions d’activité qui informe une personne que son approbation est nécessaire, et qui attend la réponse en temps réel. Une fois que toutes les activités d’approbation ont reçu des réponses ou ont expiré, supposons qu’une autre activité échoue en raison d’un problème de configuration d’application (par exemple, une chaîne de connexion de base de données non valide). Il en résulte un échec de l’orchestration, survenu en profondeur dans le workflow. Avec l’API `RewindAsync` (.NET) ou `rewind` (JavaScript), un administrateur d’application peut corriger l’erreur de configuration et rembobiner (rewind) l’orchestration ayant échoué jusqu’à l’état situé immédiatement avant l’échec. Aucune des étapes nécessitant une interaction humaine ne doit être approuvée de nouveau ; l’orchestration peut désormais s’effectuer correctement.

> [!NOTE]
> La fonctionnalité de *rembobinage* ne prend pas en charge le rembobinage des instances d’orchestration qui utilisent des minuteurs durables.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RewindInstance")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("rewind-queue")] string instanceId)
{
    string reason = "Orchestrator failed and needs to be revived.";
    return client.RewindAsync(instanceId, reason);
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const reason = "Orchestrator failed and needs to be revived.";
    return client.rewind(instanceId, reason);
};
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

> [!NOTE]
> Cette fonctionnalité n’est pas prise en charge en Python.

<!-- ```python
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str, instance_id: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    reason = "Orchestrator failed and needs to be revived."
    return client.rewind(instance_id, reason)
``` -->

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Vous pouvez aussi rembobiner une instance d’orchestration, à l’aide de la commande [`func durable rewind` ](../functions-core-tools-reference.md#func-durable-rewind) dans Core Tools.

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande `durable rewind` utilise les paramètres suivants :

* **`id` (obligatoire)**  : ID de l’instance d’orchestration.
* **`reason` (facultatif)**  : Motif de rembobinage de l’instance d’orchestration.
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, le nom du hub de tâches dans le fichier [host.json](durable-functions-bindings.md#host-json) est utilisé.

```bash
func durable rewind --id 0ab8c55a66644d68a3a8b220b12d209c --reason "Orchestrator failed and needs to be revived."
```

## <a name="purge-instance-history"></a>Vider l’historique des instances

Pour supprimer toutes les données associées à une orchestration, vous pouvez vider l’historique des instances. Par exemple, vous souhaiterez peut-être supprimer toutes les lignes de table Azure et les objets blob des messages volumineux associés à une instance terminée. Pour ce faire, utilisez la méthode `PurgeInstanceHistoryAsync` (.NET), `purgeInstanceHistory` (JavaScript) ou `purge_instance_history` (Python) de l’objet [client d’orchestration](durable-functions-bindings.md#orchestration-client).

Cette méthode comporte deux surcharges. La première vide l’historique par ID d’instance d’orchestration :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("PurgeInstanceHistory")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("purge-queue")] string instanceId)
{
    return client.PurgeInstanceHistoryAsync(instanceId);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);
    return client.purgeInstanceHistory(instanceId);
};
```

Consultez les [Instances de démarrage](#javascript-function-json) pour la configuration de la function  JSON.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str, instance_id: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)

    return client.purge_instance_history(instance_id)
```

---

L’exemple suivant affiche une fonction déclenchée, par un minuteur, qui purge l’historique de toutes les instances d’orchestration terminées après l’intervalle de temps spécifié. Dans ce cas, elle supprime les données de toutes les instances terminées depuis 30 jours ou plus. Cet exemple de fonction est programmé pour s’exécuter une fois par jour, à midi UTC :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("PurgeInstanceHistory")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [TimerTrigger("0 0 12 * * *")] TimerInfo myTimer)
{
    return client.PurgeInstanceHistoryAsync(
        DateTime.MinValue,
        DateTime.UtcNow.AddDays(-30),  
        new List<OrchestrationStatus>
        {
            OrchestrationStatus.Completed
        });
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La méthode `purgeInstanceHistoryBy` peut être utilisée pour vider l’historique des instances de plusieurs instances de manière conditionnelle.

**function.json**

```json
{
  "bindings": [
    {
      "schedule": "0 0 12 * * *",
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

> [!NOTE]
> Cet exemple cible la version 2  x de Durable Functions. Dans la version 1. x, utilisez `orchestrationClient`au lieu de`durableClient`.

**index.js**

```javascript
const df = require("durable-functions");

module.exports = async function (context, myTimer) {
    const client = df.getClient(context);
    const createdTimeFrom = new Date(0);
    const createdTimeTo = new Date().setDate(today.getDate() - 30);
    const runtimeStatuses = [ df.OrchestrationRuntimeStatus.Completed ];
    return client.purgeInstanceHistoryBy(createdTimeFrom, createdTimeTo, runtimeStatuses);
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from azure.durable_functions.models.DurableOrchestrationStatus import OrchestrationRuntimeStatus
from datetime import datetime, timedelta

async def main(req: func.HttpRequest, starter: str, instance_id: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    created_time_from = datetime.min
    created_time_to = datetime.today() + timedelta(days = -30)
    runtime_statuses = [OrchestrationRuntimeStatus.Completed]

    return client.purge_instance_history_by(created_time_from, created_time_to, runtime_statuses)
```
---

> [!NOTE]
> Pour que cette opération aboutisse, l’état d’exécution de l’instance cible doit être **Terminé**, **Terminé** ou **Échec**.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Vous pouvez aussi supprimer définitivement l’historique d’une instance d’orchestration, à l’aide de la commande [`func durable purge-history` ](../functions-core-tools-reference.md#func-durable-purge-history) dans Core Tools. Tout comme dans le deuxième exemple C# de la section précédente, elle vide l’historique de toutes les instances d’orchestration créées dans un intervalle de temps spécifié. Vous pouvez filtrer encore davantage les instances vidées, selon leur état d’exécution.

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande `durable purge-history` a plusieurs paramètres :

* **`created-after` (facultatif)**  : Purge l’historique des instances créées après cette date/heure (UTC). Les dates et heures au format ISO 8601 sont acceptées.
* **`created-before` (facultatif)**  : Purge l’historique des instances créées avant cette date/heure (UTC). Les dates et heures au format ISO 8601 sont acceptées.
* **`runtime-status` (facultatif)**  : Videz l’historique des instances avec un état particulier (par exemple, Running ou Completed). Peut fournir plusieurs états (séparés par des espaces).
* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, le nom du hub de tâches dans le fichier [host.json](durable-functions-bindings.md#host-json) est utilisé.

La commande suivante supprime l’historique de toutes les instances qui ont été créées avant le 14 novembre 2021 à 19:35 (UTC) et qui ont échoué.

```bash
func durable purge-history --created-before 2021-11-14T19:35:00.0000000Z --runtime-status failed
```

## <a name="delete-a-task-hub"></a>Supprimer un hub de tâches

À l’aide de la commande [`func durable delete-task-hub` ](../functions-core-tools-reference.md#func-durable-delete-task-hub) dans Core Tools, vous pouvez supprimer tous les artefacts de stockage associés à un concentrateur de tâches particulier, notamment les tables de stockage, les files d’attente et les objets blob Azure. 

> [!NOTE]
> Les commandes Core Tools ne sont actuellement prises en charge que lors de l’utilisation du [fournisseur Stockage Azure](durable-functions-storage-providers.md) par défaut pour rendre l’état d’exécution persistant.

La commande `durable delete-task-hub` a deux paramètres :

* **`connection-string-setting` (facultatif)**  : Nom du paramètre d’application contenant la chaîne de connexion de stockage à utiliser. Par défaut, il s’agit de `AzureWebJobsStorage`.
* **`task-hub-name` (facultatif)**  : Nom du hub de tâches Durable Functions à utiliser. Par défaut, le nom du hub de tâches dans le fichier [host.json](durable-functions-bindings.md#host-json) est utilisé.

La commande suivante supprime toutes les données de stockage Azure associées au hub de tâches `UserTest`.

```bash
func durable delete-task-hub --task-hub-name UserTest
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment gérer les versions](durable-functions-versioning.md)

> [!div class="nextstepaction"]
> [Référence sur l'API HTTP intégrée pour la gestion des instances](durable-functions-http-api.md)
