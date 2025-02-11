---
title: Meilleures pratiques pour améliorer les performances à l’aide de Azure Service Bus
description: Explique comment utiliser Service Bus pour optimiser les performances lors de l’échange de messages répartis.
ms.topic: article
ms.date: 08/30/2021
ms.openlocfilehash: d7bd692809504bb16607a431e879f0abfff953cb
ms.sourcegitcommit: 40866facf800a09574f97cc486b5f64fced67eb2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/30/2021
ms.locfileid: "123225252"
---
# <a name="best-practices-for-performance-improvements-using-service-bus-messaging"></a>Meilleures pratiques relatives aux améliorations de performances à l’aide de la messagerie Service Bus

Cet article décrit comment utiliser Azure Service Bus pour optimiser les performances lors de l’échange de messages répartis. La première partie de cet article décrit les différents mécanismes pour améliorer les performances. La deuxième partie fournit des conseils sur l’utilisation de Service Bus des services visant à offrir des performances optimales dans un scénario donné.

Dans cet article, le terme « client » fait référence à une entité qui accède à Service Bus. Un client peut jouer le rôle d’un expéditeur ou d’un destinataire. Le terme « expéditeur » est utilisé pour un client de file d’attente ou de rubrique Service Bus qui envoie des messages à une file d’attente ou une rubrique Service Bus. Le terme « destinataire » fait référence à un client de file d’attente ou d’abonnement de Bus de Service qui reçoit des messages de la part d’une file d’attente ou d’un abonnement Service Bus.

## <a name="resource-planning-and-considerations"></a>Planification et considérations relatives aux ressources

Comme pour toute ressource technique, une planification prudente est essentielle pour garantir qu’Azure Service Bus fournit les performances attendues par votre application. La configuration ou la topologie adéquate pour vos espaces de noms Service Bus dépend d’une multitude de facteurs impliquant l’architecture de votre application et la façon dont chacune des fonctionnalités de Service Bus est utilisée.

### <a name="pricing-tier"></a>Niveau tarifaire

Service Bus propose plusieurs niveaux tarifaires. Il est recommandé de choisir le niveau approprié pour les besoins de votre application.

   * **Niveau standard** : Adapté aux environnements de développement et de test ou aux scénarios à faible débit où les applications ne sont **pas sensibles** à la limitation.

   * **Niveau Premium** : Adapté aux environnements de production avec des exigences de débit variables où la latence et le débit doivent être prévisibles. En outre, les espaces de noms Premium de Service Bus peuvent être [mis à l’échelle automatiquement](automate-update-messaging-units.md) pour s’adapter aux pics de débit.

> [!NOTE]
> Si vous ne choisissez pas le niveau approprié, il existe un risque de saturation de l’espace de noms Service Bus, ce qui peut entraîner une [limitation](service-bus-throttling.md).
>
> La limitation n’entraîne pas la perte de données. Les applications tirant parti du Kit de développement logiciel (SDK) Service Bus peuvent utiliser la stratégie de nouvelles tentatives par défaut pour s’assurer que les données sont finalement acceptées par Service Bus.
>

### <a name="calculating-throughput-for-premium"></a>Calcul du débit pour le niveau Premium

Les données envoyées à Service Bus sont sérialisées en binaire, puis désérialisées lorsqu’elles sont reçues par le destinataire. Ainsi, tandis que les applications considèrent les **messages** comme des unités atomiques de travail, Service Bus mesure le débit en octets (ou mégaoctets).

Pour calculer le débit requis, tenez compte des données envoyées à Service Bus (entrée) et des données reçues de Service Bus (sortie).

Comme prévu, le débit est plus élevé lorsque les charges utiles des messages sont plus petites et peuvent être regroupées.

#### <a name="benchmarks"></a>Benchmarks

Voici un [exemple GitHub](https://github.com/Azure-Samples/service-bus-dotnet-messaging-performance) que vous pouvez exécuter pour voir le débit attendu pour votre espace de noms Service Bus. Dans nos [tests de point de référence](https://techcommunity.microsoft.com/t5/Service-Bus-blog/Premium-Messaging-How-fast-is-it/ba-p/370722), nous avons observé environ 4 Mo/s par unité de messagerie (MU) d’entrée et de sortie.

L’exemple de point de référence n’utilise pas de fonctionnalités avancées, le débit observé par vos applications est donc différent selon vos scénarios.

#### <a name="compute-considerations"></a>Considérations relatives à la capacité de calcul

L’utilisation de certaines fonctionnalités de Service Bus peut nécessiter une utilisation du calcul qui peut diminuer le débit attendu. Voici quelques-unes de ces fonctionnalités :

1. Sessions
2. Extension à plusieurs abonnements sur une même rubrique
3. Exécution de nombreux filtres sur un abonnement unique
4. Messages planifiés
5. Messages reportés
6. Transactions
7. Déduplication et fenêtre de temps des périodes passées
8. Transfert à (transfert d’une entité à une autre)

Si votre application utilise l’une des fonctionnalités ci-dessus et que vous ne recevez pas le débit attendu, vous pouvez examiner les métriques d’**utilisation de l’UC** et envisager d’effectuer un scale-up de votre espace de noms Service Bus Premium.

Vous pouvez également utiliser Azure Monitor pour [mettre automatiquement à l’échelle l’espace de noms Service Bus](automate-update-messaging-units.md).

### <a name="sharding-across-namespaces"></a>Partitionnement entre les espaces de noms

Bien que le scale-up du calcul (unités de messagerie) alloué à l’espace de noms soit une solution plus facile, il **ne peut pas** fournir une augmentation linéaire du débit. Cela est dû aux éléments internes de Service Bus (stockage, réseau, etc.) qui peuvent limiter le débit.

Dans ce cas, la solution la plus propre consiste à partition vos entités (files d’attente et rubriques) dans différents espaces de noms Service Bus Premium. Vous pouvez également envisager de partitionner vos entités sur différents espaces de noms dans différentes régions Azure.

## <a name="protocols"></a>Protocoles
Service Bus permet aux clients d’envoyer et de recevoir des messages par le biais de l’un des trois protocoles suivants :

1. Advanced Message Queuing Protocol (AMQP)
2. Service Bus Messaging Protocol (SBMP)
3. Hypertext Transfer Protocol (HTTP)

AMQP est le plus efficace, car il maintient la connexion à Service Bus. Il permet également le [traitement par lot](#batching-store-access) et la [lecture anticipée](#prefetching). Sauf mention explicite, tout le contenu de cet article suppose l’utilisation du protocole AMQP ou SBMP.

> [!IMPORTANT]
> Le protocole SBMP est disponible uniquement pour .NET Framework. AMQP est le protocole par défaut pour .NET Standard.

## <a name="choosing-the-appropriate-service-bus-net-sdk"></a>Choix du Kit de développement logiciel (SDK) .NET Service Bus approprié
Il existe trois Kits de développement logiciel (SDK) .NET Azure Service Bus pris en charge. Leurs API sont similaires, et il peut être difficile de choisir entre les deux. Reportez-vous au tableau suivant pour vous aider à prendre votre décision. Le Kit de développement logiciel (SDK) Azure.Messaging.ServiceBus est le plus récent, et nous vous recommandons de l’utiliser plutôt que d’autres SDK. Les Kits de développement logiciel (SDK) Azure.Messaging.ServiceBus et Microsoft.Azure.ServiceBus sont tous deux modernes, performants et compatibles sur différentes plateformes. En outre, ils prennent en charge AMQP sur WebSockets et font partie de la collection de Kits de développement logiciel (SDK) .NET Azure de projets open source.

| Package NuGet | Espace(s) de noms principal(-aux) | Plateforme(s) minimale(s) | Protocole(s) |
|---------------|----------------------|---------------------|-------------|
| [Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus) | `Azure.Messaging.ServiceBus`<br>`Azure.Messaging.ServiceBus.Administration` | .NET Core 2.0<br>.NET Framework 4.6.1<br>Mono 5.4<br>Xamarin.iOS 10.14<br>Xamarin.Mac 3.8<br>Xamarin.Android 8.0<br>Plateforme Windows universelle 10.0.16299 | AMQP<br>HTTP |
| [Microsoft.Azure.ServiceBus ](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus) | `Microsoft.Azure.ServiceBus`<br>`Microsoft.Azure.ServiceBus.Management` | .NET Core 2.0<br>.NET Framework 4.6.1<br>Mono 5.4<br>Xamarin.iOS 10.14<br>Xamarin.Mac 3.8<br>Xamarin.Android 8.0<br>Plateforme Windows universelle 10.0.16299 | AMQP<br>HTTP |
| [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus) | `Microsoft.ServiceBus`<br>`Microsoft.ServiceBus.Messaging` | .NET Framework 4.6.1 | AMQP<br>SBMP<br>HTTP |

Pour plus d’informations sur la prise en charge minimale de la plateforme .NET Standard, consultez [Prise en charge de l’implémentation .NET](/dotnet/standard/net-standard#net-implementation-support).

## <a name="reusing-factories-and-clients"></a>Réutilisation de structures et de clients
# <a name="azuremessagingservicebus-sdk"></a>[Kit de développement logiciel (SDK) Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
Les objets Service Bus qui interagissent avec le service, tels que [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient), [ServiceBusSender](/dotnet/api/azure.messaging.servicebus.servicebussender), [ServiceBusReceiver](/dotnet/api/azure.messaging.servicebus.servicebusreceiver) et [ServiceBusProcessor](/dotnet/api/azure.messaging.servicebus.servicebusprocessor), doivent être inscrits pour l’injection de dépendances en tant que singletons (ou instanciés une fois et partagés). ServiceBusClient peut être inscrit pour l’injection de dépendances avec [ServiceBusClientBuilderExtensions](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/servicebus/Azure.Messaging.ServiceBus/src/Compatibility/ServiceBusClientBuilderExtensions.cs). 

Nous vous recommandons de ne pas fermer ni de supprimer ces objets après l’envoi ou la réception de chaque message. La fermeture ou la suppression des objets spécifiques à une entité (ServiceBusSender/Receiver/Processor) entraîne la suppression de la liaison avec le service Service Bus. La suppression du ServiceBusClient provoque la suppression de la connexion au service Service Bus. 

# <a name="microsoftazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

Des objets clients Service Bus, tels que les implémentations de [`IQueueClient`][QueueClient] ou [`IMessageSender`][MessageSender], doivent être inscrits pour l’injection de dépendances en tant que singletons (ou instanciés une fois et partagés). Nous déconseillons de fermer les structures de messagerie ou les files d’attente, les rubriques ou les clients d’abonnement après avoir envoyé un message, et de les recréer lorsque vous envoyez le message suivant. La fermeture d’une fabrique de messagerie supprime la connexion au service Service Bus. Une nouvelle connexion est établie lors de la recréation de la fabrique. 

# <a name="windowsazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Des objets clients Service Bus, tels que `QueueClient` ou `MessageSender`, sont créés via un objet [MessagingFactory][MessagingFactory], qui assure également la gestion interne des connexions. Nous déconseillons de fermer les structures de messagerie ou les files d’attente, les rubriques ou les clients d’abonnement après avoir envoyé un message, et de les recréer lorsque vous envoyez le message suivant. La fermeture d’une structure de messagerie supprime la connexion à Service Bus et une nouvelle connexion est établie au moment de la recréation de la structure. 

---

La remarque suivante s’applique à tous les kits SDK :

> [!NOTE]
> L’établissement d’une connexion est une opération coûteuse que vous pouvez éviter en réutilisant les mêmes objets fabrique et client pour plusieurs opérations. Vous pouvez utiliser en toute sécurité ces objets clients pour envoyer des messages à partir de plusieurs threads et d’opérations asynchrones simultanées.

## <a name="concurrent-operations"></a>Opérations simultanées
Les opérations telles qu’envoyer, recevoir, supprimer, etc., prennent un certain temps. Ce temps comprend le traitement de l’opération par le service Service Bus et la latence de la requête et de la réponse. Pour augmenter le nombre d’opérations par période, les opérations doivent s’exécuter simultanément.

Le client planifie les opérations parallèles en effectuant des opérations **asynchrones**. La requête suivante démarre avant la fin de la demande précédente. Voici un exemple d’opération d’envoi asynchrone présenté sous forme d’extrait de code :

# <a name="azuremessagingservicebus-sdk"></a>[Kit de développement logiciel (SDK) Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
```csharp
var messageOne = new ServiceBusMessage(body);
var messageTwo = new ServiceBusMessage(body);

var sendFirstMessageTask =
    sender.SendMessageAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    sender.SendMessageAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");

```

# <a name="microsoftazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

```csharp
var messageOne = new Message(body);
var messageTwo = new Message(body);

var sendFirstMessageTask =
    queueClient.SendAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    queueClient.SendAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");
```

# <a name="windowsazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) WindowsAzure.ServiceBus](#tab/net-framework-sdk)

```csharp
var messageOne = new BrokeredMessage(body);
var messageTwo = new BrokeredMessage(body);

var sendFirstMessageTask =
    queueClient.SendAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    queueClient.SendAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");
```

---

Voici un exemple d’opération de réception asynchrone présenté sous forme de code.

# <a name="azuremessagingservicebus-sdk"></a>[Kit de développement logiciel (SDK) Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)

```csharp
var client = new ServiceBusClient(connectionString);
var options = new ServiceBusProcessorOptions 
{

      AutoCompleteMessages = false,
      MaxConcurrentCalls = 20
};
await using ServiceBusProcessor processor = client.CreateProcessor(queueName,options);
processor.ProcessMessageAsync += MessageHandler;
processor.ProcessErrorAsync += ErrorHandler;

static Task ErrorHandler(ProcessErrorEventArgs args)
{
    Console.WriteLine(args.Exception);
    return Task.CompletedTask;
};

static async Task MessageHandler(ProcessMessageEventArgs args)
{
    Console.WriteLine("Handle message");
    await args.CompleteMessageAsync(args.Message);
}

await processor.StartProcessingAsync();
```

# <a name="microsoftazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

Consultez le [dépôt GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/SendersReceiversWithQueues) pour obtenir des exemples complets de code source. 

```csharp
var receiver = new MessageReceiver(connectionString, queueName, ReceiveMode.PeekLock);

static Task LogErrorAsync(Exception exception)
{
    Console.WriteLine(exception);
    return Task.CompletedTask;
};

receiver.RegisterMessageHandler(
    async (message, cancellationToken) =>
    {
        Console.WriteLine("Handle message");
        await receiver.CompleteAsync(message.SystemProperties.LockToken);
    },
    new MessageHandlerOptions(e => LogErrorAsync(e.Exception))
    {
        AutoComplete = false,
        MaxConcurrentCalls = 20
    });
```

L’objet `MessageReceiver` est instancié avec la chaîne de connexion, le nom de la file d’attente et un mode de réception Verrouillage. Ensuite, l’instance `receiver` est utilisée pour inscrire le gestionnaire de messages.

# <a name="windowsazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Consultez le [dépôt GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/SendersReceiversWithQueues) pour obtenir des exemples complets de code source.

```csharp
var factory = MessagingFactory.CreateFromConnectionString(connectionString);
var receiver = await factory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);

// Register the handler to receive messages asynchronously
receiver.OnMessageAsync(
    async message =>
    {
        Console.WriteLine("Handle message");
        await message.CompleteAsync();
    },
    new OnMessageOptions
    {
        AutoComplete = false,
        MaxConcurrentCalls = 20
    });
```

`MessagingFactory` crée un objet `factory` à partir de la chaîne de connexion. Avec l’instance `factory`, un `MessageReceiver` est instancié. Ensuite, l’instance `receiver` est utilisée pour inscrire le gestionnaire de messages OnMessage.

---

## <a name="receive-mode"></a>Mode de réception

Lorsque vous créez un client de file d’attente ou d’abonnement, vous pouvez spécifier un mode de réception : *Verrouillage* ou *Recevoir et supprimer*. Le mode de réception par défaut est `PeekLock`. En mode par défaut, le client envoie une requête pour recevoir un message de Service Bus. Une fois que le client a reçu le message, il envoie une demande pour compléter le message.

Si le mode de réception est défini sur `ReceiveAndDelete`, les deux étapes sont regroupées en une seule requête. Cette étape réduit le nombre total d’opérations et peut améliorer le débit global des messages. Ce gain de performances est fourni au risque de perdre des messages.

Service Bus ne gère pas les transactions des opérations de réception-suppression. En outre, la syntaxe de verrouillage est nécessaire pour tous les scénarios dans lesquels le client souhaite différer ou ignorer un message ([lettre morte](service-bus-dead-letter-queues.md)).

## <a name="client-side-batching"></a>Traitement par lots côté client

Le traitement par lot côté client permet à un client de file d’attente ou de rubrique de retarder l’envoi d’un message pendant une période donnée. Si le client envoie des messages supplémentaires pendant cette période, il transmet les messages dans un seul lot. Le traitement par lots côté client fait également en sorte qu’une file d’attente ou un abonnement client regroupe plusieurs requêtes **complètes** en une seule requête. Le traitement par lots n’est disponible que pour les opérations **d’envoi** et **complètes** asynchrones. Les opérations synchrones sont immédiatement envoyées au service Service Bus. Le traitement par lots ne peut pas concerner des opérations de verrouillage ou de réception, ou plusieurs clients.

# <a name="azuremessagingservicebus-sdk"></a>[Kit de développement logiciel (SDK) Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
La fonctionnalité de traitement par lot pour le Kit de développement logiciel (SDK) .NET Standard n’expose pas encore de propriété à manipuler.

# <a name="microsoftazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

La fonctionnalité de traitement par lot pour le Kit de développement logiciel (SDK) .NET Standard n’expose pas encore de propriété à manipuler.

# <a name="windowsazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Par défaut, un client utilise un intervalle 20 ms entre les lots. Vous pouvez modifier l’intervalle de traitement par lots en définissant la propriété [BatchFlushInterval][BatchFlushInterval] avant de créer la structure de messagerie. Ce paramètre affecte tous les clients créés par cette structure.

Pour désactiver le traitement par lot, définissez la propriété [BatchFlushInterval][BatchFlushInterval] sur **TimeSpan.Zero**. Par exemple :

```csharp
var settings = new MessagingFactorySettings
{
    NetMessagingTransportSettings =
    {
        BatchFlushInterval = TimeSpan.Zero
    }
};
var factory = MessagingFactory.Create(namespaceUri, settings);
```

Le traitement par lot n’affecte pas le nombre d’opérations de messagerie facturables et est disponible uniquement pour le protocole client Service Bus utilisant la bibliothèque [Microsoft.ServiceBus.Messaging](https://www.nuget.org/packages/WindowsAzure.ServiceBus/). Le protocole HTTP ne prend pas en charge le traitement par lot.

> [!NOTE]
> La définition de `BatchFlushInterval` garantit que le traitement par lot est implicite du point de vue de l’application. Autrement dit, l’application effectue des appels `SendAsync` et `CompleteAsync` n’effectue pas d’appels Batch spécifiques.
>
> Le traitement par lot explicite du côté client peut être implémenté en utilisant l’appel de méthode ci-dessous :
> ```csharp
> Task SendBatchAsync(IEnumerable<BrokeredMessage> messages);
> ```
> Ici, la taille combinée des messages doit être inférieure à la taille maximale prise en charge par le niveau tarifaire.

---

## <a name="batching-store-access"></a>Accès au dispositif de stockage de traitement par lot

Pour augmenter le débit d’une file d’attente, d’une rubrique ou d’un abonnement, Service Bus regroupe plusieurs messages lorsqu’il écrit à son dispositif de stockage en interne. 

- Lorsque vous activez le traitement par lot sur une file d’attente, l’écriture des messages dans le magasin et la suppression des messages du magasin sont traitées par lot. 
- Lorsque vous activez le traitement par lot sur une rubrique, l’écriture de messages dans le magasin est regroupée. 
- Lorsque vous activez le traitement par lot sur un abonnement, l’écriture et la suppression de messages dans le magasin sont traitées par lot. 
- Lorsque l’accès au magasin par lot est activé pour une entité, Service Bus retarde l’opération d’écriture de dispositif de stockage pour cette entité de 20 ms au maximum.

> [!NOTE]
> Il n’existe aucun risque de perdre des messages avec le traitement par lot, même en cas de défaillance de Service Bus à la fin d’un intervalle de traitement par lot de 20 ms.

Les opérations de stockage supplémentaires qui se produisent pendant cet intervalle sont ajoutées au lot. L’accès au dispositif de stockage par lot affecte seulement les opérations **d’envoi** et **complètes** ; les opérations de réception ne sont pas affectées. L’accès au dispositif de stockage est une propriété d’entité. Le traitement par lot se produit sur toutes les entités qui permettent l’accès au stockage par lot.

Lorsque vous créez une file d’attente, une rubrique ou un abonnement, l’accès au stockage par lot est activé par défaut.


# <a name="azuremessagingservicebus-sdk"></a>[Kit de développement logiciel (SDK) Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
Pour désactiver l’accès au magasin par lot, vous avez besoin d’une instance `ServiceBusAdministrationClient`. Créez un `CreateQueueOptions` à partir d’une description de file d’attente qui définit la propriété `EnableBatchedOperations` sur `false`.

```csharp
var options = new CreateQueueOptions(path)
{
    EnableBatchedOperations = false
};
var queue = await administrationClient.CreateQueueAsync(options);
```


# <a name="microsoftazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

Pour désactiver l’accès au magasin par lot, vous avez besoin d’une instance `ManagementClient`. Créez une file d’attente à partir d’une description de file d’attente qui définit la propriété `EnableBatchedOperations` sur la valeur `false`.

```csharp
var queueDescription = new QueueDescription(path)
{
    EnableBatchedOperations = false
};
var queue = await managementClient.CreateQueueAsync(queueDescription);
```

Pour plus d’informations, consultez les articles suivants :
- [Propriété QueueDescription.EnableBatchedOperations](/dotnet/api/microsoft.azure.servicebus.management.queuedescription.enablebatchedoperations)
- [Propriété SubscriptionDescription.EnabledBatchedOperations](/dotnet/api/microsoft.azure.servicebus.management.subscriptiondescription.enablebatchedoperations)
* [TopicDescription.EnableBatchedOperations](/dotnet/api/microsoft.azure.servicebus.management.topicdescription.enablebatchedoperations)

# <a name="windowsazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Pour désactiver l’accès au magasin par lot, vous avez besoin d’une instance `NamespaceManager`. Créez une file d’attente à partir d’une description de file d’attente qui définit la propriété `EnableBatchedOperations` sur la valeur `false`.

```csharp
var queueDescription = new QueueDescription(path)
{
    EnableBatchedOperations = false
};
var queue = namespaceManager.CreateQueue(queueDescription);
```

Pour plus d’informations, consultez les articles suivants :
* [`Microsoft.ServiceBus.Messaging.QueueDescription.EnableBatchedOperations`](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enablebatchedoperations)
* [`Microsoft.ServiceBus.Messaging.SubscriptionDescription.EnableBatchedOperations`](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.enablebatchedoperations)
* [`Microsoft.ServiceBus.Messaging.TopicDescription.EnableBatchedOperations`](/dotnet/api/microsoft.servicebus.messaging.topicdescription.enablebatchedoperations).

---

L’accès au magasin par lot n’affecte pas le nombre d’opérations de messagerie facturables. Il s’agit d’une propriété d’une file d’attente, d’une rubrique ou d’un abonnement. Il est indépendant du mode de réception et du protocole utilisé entre un client et le service Service Bus.

## <a name="prefetching"></a>Lecture anticipée

La [lecture anticipée](service-bus-prefetch.md) permet au client de la file d’attente ou de l’abonnement de charger des messages supplémentaires à partir du service quand il reçoit des messages. Le client stocke ces messages en mémoire cache. La taille du cache est déterminée par les propriétés `QueueClient.PrefetchCount` ou `SubscriptionClient.PrefetchCount`. Chaque client qui permet la lecture anticipée gère son propre cache. Un cache n’est pas partagé par plusieurs clients. Si le client initie une opération de réception et que sa mémoire cache est vide, le service transmet un lot de messages. La taille du lot est égale à la taille du cache ou à 256 Ko, la plus faible l’emportant. Si le client initie une opération de réception et que le cache contient un message, ce dernier est extrait de la mémoire cache.

Lorsqu’un message est lu par anticipation, le service le verrouille. Ainsi, le message lu par anticipation ne peut pas être reçu par un autre destinataire. Si le destinataire ne peut pas terminer le message avant expiration du verrouillage, le message devient disponible pour les autres destinataires. La copie lue par anticipation du message reste dans le cache. Le destinataire qui consomme la copie mise en cache expirée reçoit une exception lorsqu’il essaie de terminer le message. Par défaut, le verrouillage du message expire au bout de 60 secondes. Cette valeur peut être étendue à 5 minutes. Pour empêcher la consommation des messages arrivés à expiration, définissez la taille du cache pour qu’elle soit inférieure au nombre de messages qu’un client peut utiliser au sein de l’intervalle de délai d’expiration de verrouillage.

Si vous utilisez la durée par défaut d’expiration de verrouillage, qui est de 60 secondes, une bonne valeur pour `PrefetchCount` est 20 fois les vitesses de traitement maximales de l’ensemble des récepteurs de la fabrique. Par exemple, une structure crée 10 destinataires, et chaque destinataire peut traiter jusqu’à 10 messages par seconde. Le nombre de lectures anticipées ne doit pas dépasser 20 X 3 X 10 = 600. Par défaut, `PrefetchCount` est défini sur 0, ce qui signifie qu’aucun message supplémentaire n’est récupéré à partir du service.

La lecture anticipée de messages augmente le débit global d’un abonnement ou une file d’attente, car elle permet de réduire le nombre total d’opérations de messagerie, ou les allers-retours. La lecture anticipée du premier message, cependant, prend plus de temps (en raison de la taille du message accrue). La réception de messages avec lecture anticipée depuis le cache sera plus rapide, car ces messages ont déjà été téléchargés par le client.

La propriété (TTL) de durée de vie d’un message est vérifiée par le serveur au moment où le serveur envoie le message au client. Le client ne vérifie pas la propriété TTL du message à la réception de ce dernier. Toutefois, le message peut être reçu même si la durée de vie du message a été dépassée pendant la mise en cache par le client.

La lecture anticipée n’affecte pas le nombre d’opérations de messagerie facturables et est disponible uniquement pour le protocole client Service Bus. Le protocole HTTP ne prend pas en charge la lecture anticipée. La lecture anticipée est disponible pour les opérations de réception synchrones et asynchrones.

# <a name="azuremessagingservicebus-sdk"></a>[Kit de développement logiciel (SDK) Azure.Messaging.ServiceBus](#tab/net-standard-sdk-2)
Pour plus d’informations, consultez les propriétés `PrefetchCount` suivantes :

- [ServiceBusReceiver.PrefetchCount](/dotnet/api/azure.messaging.servicebus.servicebusreceiver.prefetchcount)
- [ServiceBusProcessor.PrefetchCount](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.prefetchcount)

Vous pouvez définir des valeurs pour ces propriétés dans [ServiceBusReceiverOptions](/dotnet/api/azure.messaging.servicebus.servicebusreceiveroptions) ou [ServiceBusProcessorOptions](/dotnet/api/azure.messaging.servicebus.servicebusprocessoroptions).

# <a name="microsoftazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) Microsoft.Azure.ServiceBus](#tab/net-standard-sdk)

Pour plus d’informations, consultez les propriétés `PrefetchCount` suivantes :

* [`Microsoft.Azure.ServiceBus.QueueClient.PrefetchCount`](/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount)
* [`Microsoft.Azure.ServiceBus.SubscriptionClient.PrefetchCount`](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.prefetchcount)

# <a name="windowsazureservicebus-sdk"></a>[Kit de développement logiciel (SDK) WindowsAzure.ServiceBus](#tab/net-framework-sdk)

Pour plus d’informations, consultez les propriétés `PrefetchCount` suivantes :

* [`Microsoft.ServiceBus.Messaging.QueueClient.PrefetchCount`](/dotnet/api/microsoft.servicebus.messaging.queueclient.prefetchcount)
* [`Microsoft.ServiceBus.Messaging.SubscriptionClient.PrefetchCount`](/dotnet/api/microsoft.servicebus.messaging.subscriptionclient.prefetchcount)

---

## <a name="prefetching-and-receivebatch"></a>Prérécupération et ReceiveBatch
Même si les concepts liés à la prérécupération de plusieurs messages ensemble présentent une sémantique similaire au traitement des messages dans un lot (`ReceiveBatch`), il existe quelques différences mineures que vous devez garder à l’esprit lorsque vous optez pour ces approches simultanément.

La prérécupération est une configuration (ou un mode) sur le client (`QueueClient`et `SubscriptionClient`) tandis que `ReceiveBatch` est une opération (qui possède une sémantique requête-réponse).

Lorsque vous optez pour ces approches en même temps, envisagez les cas suivants :

* Le nombre de prérécupérations doit être supérieur ou égal au nombre de messages que vous vous attendez à recevoir de `ReceiveBatch`.
* Le nombre de prérécupérations peut aller jusqu’à n/3 fois le nombre de messages traités par seconde, sachant que n est la durée de verrouillage par défaut.

Quelques difficultés surviennent lors d’une approche gourmande (par exemple, en conservant le nombre de prérécupérations élevé), car cela implique le verrouillage du message sur un récepteur particulier. La recommandation à suivre est d’essayer des valeurs de prérécupération entre les seuils mentionnés ci-dessus et d’identifier empiriquement ce qui convient.

## <a name="multiple-queues"></a>Files d’attente multiples

Si une file d’attente ou une rubrique ne peut pas gérer le volume attendu, utilisez plusieurs entités de messagerie. Lorsque vous utilisez plusieurs entités, créez un client dédié pour chacune d’elles au lieu d’utiliser le même client pour toutes les entités.

## <a name="development-and-testing-features"></a>Fonctionnalités de développement et de test

> [!NOTE]
> Cette section s’applique uniquement au Kit de développement logiciel (SDK) WindowsAzure.ServiceBus, car Microsoft.Azure.ServiceBus et Azure.Messaging.ServiceBus n’exposent pas cette fonctionnalité.

Service Bus possède une fonctionnalité utilisée spécifiquement pour le développement et qui **ne doit jamais être utilisée dans les configurations de production** : [`TopicDescription.EnableFilteringMessagesBeforePublishing`][TopicDescription.EnableFiltering].

Lorsque des règles ou des filtres sont ajoutés à la rubrique, vous pouvez utiliser [`TopicDescription.EnableFilteringMessagesBeforePublishing`][TopicDescription.EnableFiltering] pour vérifier que la nouvelle expression de filtre fonctionne comme prévu.

## <a name="scenarios"></a>Scénarios

Les sections suivantes décrivent les scénarios de messagerie classiques et soulignent les paramètres Service Bus favoris par défaut. Les débits sont classés dans la catégorie Petit (moins d’1 message/seconde), Modéré (1 message/s ou plus, mais moins de 100 messages par seconde) et Élevé (100 messages/seconde ou plus). Le nombre de clients est classé en tant que petit (5 ou moins), modéré (plus de 5, mais inférieur ou égal à 20) et grand (plus de 20).

### <a name="high-throughput-queue"></a>File d’attente à débit élevé

Objectif : Maximiser le débit d’une file d’attente unique. Le nombre d’expéditeurs et de destinataires est faible.

* Pour augmenter la vitesse de transmission globale dans la file d’attente, utilisez plusieurs structures de messages pour créer des expéditeurs. Pour chaque expéditeur, utilisez des opérations asynchrones ou plusieurs threads.
* Pour augmenter la vitesse de réception depuis la file d’attente, utilisez plusieurs structures de messages pour créer des destinataires.
* Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.
* Définissez l’intervalle de mise en lot sur 50 ms pour réduire le nombre de transmissions de protocole client Service Bus. Si plusieurs expéditeurs sont utilisés, augmentez l’intervalle de traitement par lot à 100 ms.
* Désactivez l’accès au magasin par lot. Cet accès augmente la cadence à laquelle les messages peuvent être écrits dans la file d’attente.
* Définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement maximale de la totalité des destinataires d’une structure. Cela réduit le nombre de transmissions de protocole client Service Bus.

### <a name="multiple-high-throughput-queues"></a>Plusieurs files d’attente haut débit

Objectif : Optimiser le débit global de plusieurs files d’attente. Le débit d’une file d’attente individuelle est modéré ou élevé.

Pour obtenir un débit maximal sur plusieurs files d’attente, utiliser les paramètres conseillés pour maximiser le débit d’une file d’attente unique. En outre, utiliser des structures différentes pour créer des clients procédant à des envois ou des réceptions de la part de différentes files d’attente.

### <a name="low-latency-queue"></a>File d’attente à latence faible

Objectif : Réduisez la latence d’une file d’attente ou d’une rubrique. Le nombre d’expéditeurs et de destinataires est faible. Le débit de la file d’attente est faible ou modéré.

* Désactiver le traitement par lots côté client. Le client envoie immédiatement un message.
* Désactiver l’accès au magasin par lot. Le service écrit immédiatement le message pour le magasin.
* Si vous utilisez un seul client, définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement du destinataire. Si plusieurs messages arrivent à la file d’attente en même temps, le protocole client Service Bus les transmet tous simultanément. Lorsque le client reçoit le message suivant, ce message est déjà dans le cache local. Le cache doit être de petite taille.
* Si vous utilisez plusieurs clients, définissez le nombre de lectures anticipées sur 0. Ainsi, le deuxième client peut recevoir le deuxième message alors que le premier client traite toujours le premier message.

### <a name="queue-with-a-large-number-of-senders"></a>File d’attente comportant un grand nombre d’expéditeurs

Objectif : Maximiser le débit d’une file d’attente ou d’une rubrique comportant un grand nombre d’expéditeurs. Chaque expéditeur envoie des messages à une vitesse modérée. Le nombre de destinataires est faible.

Service Bus permet jusqu’à 1000 connexions simultanées vers une entité de messagerie. Cette limite s’applique au niveau de l’espace de noms et les rubriques, files d’attente ou abonnements sont limités par la limite de connexions simultanées par espace de noms. Pour les files d’attente, ce nombre est partagé entre les expéditeurs et les destinataires. Si les 1 000 connexions sont requises pour les expéditeurs, remplacez la file d’attente par une rubrique et un seul abonnement. Une rubrique accepte jusqu’à 1000 connexions simultanées des expéditeurs. L’abonnement accepte 1000 connexions simultanées supplémentaires de la part des destinataires. Si plus de 1 000 expéditeurs simultanés sont requis, les expéditeurs doivent envoyer leurs messages vers le protocole de Service Bus via HTTP.

Pour maximiser le débit, procédez comme suit :

* Si chaque expéditeur se trouve dans un processus différent, utilisez uniquement une structure par processus.
* Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.
* Utilisez la valeur par défaut de l’intervalle de 20 ms pour réduire le nombre de transmissions de protocole client Service Bus.
* Désactivez l’accès au magasin par lot. Cet accès augmente la cadence à laquelle les messages peuvent être écrits dans la file d’attente ou la rubrique.
* Définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement maximale de la totalité des destinataires d’une structure. Cela réduit le nombre de transmissions de protocole client Service Bus.

### <a name="queue-with-a-large-number-of-receivers"></a>File d’attente comportant un grand nombre de destinataires

Objectif : Optimiser la vitesse de réception d’une file d’attente ou d’abonnement comportant un grand nombre de destinataires. Chaque destinataire reçoit les messages à une vitesse modérée. Le nombre d’expéditeurs est faible.

Service Bus permet jusqu’à 1 000 connexions simultanées vers une entité. Si une file d’attente nécessite plus de 1 000 destinataires, remplacez la file d’attente par une rubrique et plusieurs abonnements. Chaque abonnement peut prendre en charge jusqu’à 1 000 connexions simultanées. Les destinataires peuvent également accéder à la file d’attente via le protocole HTTP.

Pour maximiser le débit, procédez comme suit :

* Si chaque destinataire se trouve dans un processus différent, utilisez uniquement une structure par processus.
* Les récepteurs peuvent utiliser des opérations synchronisées ou asynchrones. Étant donné la vitesse de réception modérée d’un destinataire individuel, le traitement côté client de la requête complète n’affecte pas le débit du destinataire.
* Désactivez l’accès au magasin par lot. Cet accès réduit la charge globale de l’entité. Cette opération réduit également la cadence générale à laquelle les messages peuvent être écrits dans la file d’attente ou la rubrique.
* Définir le nombre de lectures anticipées sur une valeur faible (par exemple, PrefetchCount = 10). Cela empêche les destinataires de rester oisifs pendant que d’autres mettent en cache un grand nombre de messages.

### <a name="topic-with-a-few-subscriptions"></a>Rubrique avec quelques abonnements

Objectif : Maximiser le débit d’une rubrique comportant quelques abonnements. Un message est reçu par un grand nombre d’abonnements, ce qui signifie que la vitesse de réception combinée sur l’ensemble des abonnements est supérieure à la vitesse d’envoi. Le nombre d’expéditeurs est faible. Le nombre de récepteurs par abonnement est faible.

Pour maximiser le débit, procédez comme suit :

* Pour augmenter la vitesse de transmission globale dans la rubrique, utilisez plusieurs structures de messages pour créer des expéditeurs. Pour chaque expéditeur, utilisez des opérations asynchrones ou plusieurs threads.
* Pour augmenter la vitesse de réception d’ensemble d’un abonnement, utilisez plusieurs structures de message pour créer des destinataires. Pour chaque destinataire, utilisez des opérations asynchrones ou plusieurs threads.
* Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.
* Utilisez la valeur par défaut de l’intervalle de 20 ms pour réduire le nombre de transmissions de protocole client Service Bus.
* Désactivez l’accès au magasin par lot. Cet accès augmente la cadence à laquelle les messages peuvent être écrits dans la rubrique.
* Définissez le nombre de lectures anticipées à 20 fois la vitesse de traitement maximale de la totalité des destinataires d’une structure. Cela réduit le nombre de transmissions de protocole client Service Bus.

### <a name="topic-with-a-large-number-of-subscriptions"></a>Rubrique comportant un grand nombre d’abonnements

Objectif : Maximiser le débit d’une rubrique comportant un grand nombre d’abonnements. Un message est reçu par un grand nombre d’abonnements, ce qui signifie que la vitesse de réception associée à l’ensemble des abonnements est supérieure à la vitesse d’envoi. Le nombre d’expéditeurs est faible. Le nombre de récepteurs par abonnement est faible.

Les rubriques comportant un grand nombre d’abonnements affichent généralement un faible débit global si tous les messages sont acheminés vers tous les abonnements. Cela est dû au fait que chaque message est reçu plusieurs fois et tous les messages d’une rubrique et tous les abonnements associés sont stockés dans le même magasin. Il est supposé ici que le nombre d’expéditeurs et le nombre de récepteurs par abonnement est faible. Service Bus prend en charge jusqu’à 2 000 abonnements par rubrique.

Pour maximiser le débit, procédez comme suit :

* Utilisez des opérations asynchrones pour tirer parti du traitement par lot côté client.
* Utilisez la valeur par défaut de l’intervalle de 20 ms pour réduire le nombre de transmissions de protocole client Service Bus.
* Désactivez l’accès au magasin par lot. Cet accès augmente la cadence à laquelle les messages peuvent être écrits dans la rubrique.
* Définissez le nombre de lectures anticipées à 20 fois la vitesse de réception prévue en secondes. Cela réduit le nombre de transmissions de protocole client Service Bus.

<!-- .NET Standard SDK, Microsoft.Azure.ServiceBus -->
[QueueClient]: /dotnet/api/microsoft.azure.servicebus.queueclient
[MessageSender]: /dotnet/api/microsoft.azure.servicebus.core.messagesender

<!-- .NET Framework SDK, Microsoft.Azure.ServiceBus -->
[MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[BatchFlushInterval]: /dotnet/api/microsoft.servicebus.messaging.messagesender.batchflushinterval
[ForcePersistence]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage.forcepersistence
[EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.enablepartitioning
[TopicDescription.EnableFiltering]: /dotnet/api/microsoft.servicebus.messaging.topicdescription.enablefilteringmessagesbeforepublishing

<!-- Local links -->
[Partitioned messaging entities]: service-bus-partitioning.md
