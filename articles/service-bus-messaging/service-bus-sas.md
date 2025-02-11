---
title: Contrôle d’accès Azure Service Bus avec des signatures d’accès partagé
description: Vue d’ensemble du contrôle d’accès Service Bus avec des signatures d’accès partagé, et informations sur l’autorisation SAP avec Azure Service Bus.
ms.topic: article
ms.date: 04/27/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 5659e159604c80ffe19cbc4b57dea5d99c1acb98
ms.sourcegitcommit: 351279883100285f935d3ca9562e9a99d3744cbd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2021
ms.locfileid: "112379813"
---
# <a name="service-bus-access-control-with-shared-access-signatures"></a>Contrôle d’accès Service Bus avec des signatures d’accès partagé

Cet article traite des *signatures d’accès partagé* (SAP), de leur fonctionnement et de la façon de les utiliser de manière indépendante de la plateforme.

SAP protège l’accès à Service Bus en fonction de règles d’autorisation. Celles-ci sont configurées sur un espace de noms ou sur une entité de messagerie (file d’attente ou rubrique). Une règle d’autorisation a un nom, est associée à des droits spécifiques et comporte une paire de clés de chiffrement. Vous utilisez le nom et la clé de la règle via le Kit de développement logiciel (SDK) Service Bus ou dans votre propre code pour générer un jeton SAP. Un client peut ensuite passer le jeton à Service Bus pour prouver l’autorisation pour l’opération demandée.

> [!NOTE]
> Azure Service Bus prend en charge l’autorisation de l’accès à un espace de noms Service Bus et à ses entités à l’aide d’Azure Active Directory (Azure AD). L’autorisation des utilisateurs ou des applications avec un jeton OAuth 2.0 retourné par Azure AD assure une meilleure sécurité que les signatures d’accès partagé. De plus, elle offre une plus grande simplicité d’utilisation. Azure AD vous évite d’avoir à stocker les jetons dans votre code. Vous êtes ainsi moins exposé au risque de failles de sécurité.
>
> Nous vous recommandons d’utiliser Azure AD avec vos applications Azure Service Bus dans la mesure du possible. Pour plus d’informations, consultez les articles suivants :
> - [Authentifier et autoriser une application avec Azure Active Directory pour accéder aux entités Azure Service Bus](authenticate-application.md).
> - [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Azure Service Bus](service-bus-managed-service-identity.md)

## <a name="overview-of-sas"></a>Présentation des signatures d’accès partagé (SAS)

Les signatures d’accès partagé sont un mécanisme d’autorisation reposant sur des revendications à l’aide de simples jetons. À l’aide d’une SAP, les clés ne sont jamais transmises simultanément. Les clés sont utilisées pour signer par chiffrement des informations qui peuvent être vérifiées ultérieurement par le service. Une SAP peut être utilisée de façon similaire à un schéma de nom d’utilisateur et de mot de passe où le client est en possession immédiate d’un nom de règle d’autorisation et d’une clé correspondante. Une SAP peut également être utilisée comme un modèle de sécurité fédéré, où le client reçoit un jeton d’accès signé et limité dans le temps d’un service d’émission de jeton de sécurité sans jamais être en possession de la clé de signature.

L’authentification SAP dans Service Bus est configurée avec des [stratégies d’autorisation d’accès partagé](#shared-access-authorization-policies) nommées ayant les droits d’accès associés, ainsi qu’une paire de clés de chiffrement primaire et secondaire. Les clés sont des valeurs de 256 bits dans une représentation Base64. Vous pouvez configurer des règles au niveau de l’espace de noms, sur les [files d’attente](service-bus-messaging-overview.md#queues) et les [rubriques](service-bus-messaging-overview.md#topics) Service Bus.

Le jeton de signature d’accès partagé contient le nom de la stratégie d’autorisation choisie, l’URI de la ressource à laquelle accéder, un délai d’expiration et une signature de chiffrement HMAC-SHA256 calculé sur ces champs à l’aide de la clé de chiffrement primaire ou secondaire de la règle d’autorisation choisie.

## <a name="shared-access-authorization-policies"></a>Stratégies d’autorisation d’accès partagé

Chaque espace de noms Service Bus et chaque entité Service Bus ont une stratégie d’autorisation d’accès partagé constituée de règles. La stratégie au niveau de l’espace de noms s’applique à toutes les entités à l’intérieur de l’espace de noms, quelle que soit leur configuration de stratégie individuelle.

Pour chaque règle de stratégie d’autorisation, vous choisissez trois éléments d’information : le **nom**, **l’étendue** et les **autorisations**. Le **nom** est un nom unique au sein de cette étendue. L’étendue est l’URI de la ressource en question. Pour un espace de noms Service Bus, l’étendue est le nom de domaine complet, par exemple `https://<yournamespace>.servicebus.windows.net/`.

Les droits qui sont attribués par la règle de stratégie peuvent être une combinaison de :

* « Envoyer » - donne le droit d’envoyer des messages à l’entité.
* « Écouter » - donne le droit de recevoir (file d’attente, abonnements) et tout le traitement des messages y afférents.
* « Gérer » - donne le droit de gérer la topologie de l’espace de noms, y compris la création et la suppression des entités.

Le droit « Gérer » inclut les droits « Envoyer » et « Recevoir ».

Une stratégie d’entité et d’espace de noms peut contenir 12 règles d’autorisation d’accès partagé maximum, ce qui laisse de l’espace à trois ensembles de règles, chacun couvrant les droits basiques et la combinaison des droits « Envoyer » et « Écouter ». Cette limite souligne que le magasin de stratégies SAP n’est pas destiné à être magasin de comptes de service ou d’utilisateur. Si votre application doit accorder l’accès aux identités de service ou d’utilisateur basées sur Service Bus, elle doit implémenter un service d’émission de jeton de sécurité qui émet des jetons SAP après une authentification et une vérification de l’accès.

Une *clé primaire* et une *clé secondaire* sont attribuées à une règle d’autorisation. Il s’agit de clés de chiffrement fortes. Ne les perdez pas et ne les diffusez pas ; elles seront toujours disponibles sur le [portail Azure][Azure portal]. Vous pouvez utiliser n’importe laquelle des clés générées et vous pouvez les régénérer à tout moment. Si vous régénérez ou modifiez une clé dans la stratégie, tous les jetons précédemment émis en fonction de cette clé deviennent instantanément non valides. Toutefois, les connexions en cours créées en fonction de ces jetons continuent de fonctionner jusqu’à ce que le jeton expire.

Lorsque vous créez un espace de noms Service Bus, une règle de stratégie nommée **RootManageSharedAccessKey** est automatiquement créée pour l’espace de noms. Cette stratégie dispose d’autorisations Gérer pour l’espace de noms complet. Il est recommandé de traiter cette règle comme un compte **racine** d’administration et de ne pas l’utiliser dans votre application. Vous pouvez créer des règles de stratégies supplémentaires sous l’onglet **Configurer** pour l’espace de noms dans le portail via PowerShell ou Azure CLI.

## <a name="best-practices-when-using-sas"></a>Bonnes pratiques lors de l’utilisation de SAP
Lorsque vous utilisez des signatures d'accès partagé dans vos applications, vous devez être conscient de deux risques potentiels :

- Si une signature d’accès partagé est divulguée, toute personne qui se la procure peut s’en servir, ce qui peut compromettre vos ressources Service Bus.
- Si une signature d’accès partagé fournie à une application cliente expire et que l’application est incapable d’en récupérer une nouvelle à partir de votre service, le fonctionnement de votre application risque d’être entravé.

Les recommandations suivantes relatives à l’utilisation des signatures d’accès partagé peuvent aider à limiter ces risques :

- **Faites en sorte que les clients renouvellent automatiquement la signature d’accès partagé si nécessaire** : les clients doivent renouveler la signature d’accès partagé bien avant l’heure d’expiration pour laisser suffisamment de temps pour de nouvelles tentatives si le service qui fournit la signature est indisponible. Si votre signature d’accès partagé doit être utilisée pour un petit nombre d’opérations immédiates de courte durée, censées être terminées avant l’heure d’expiration, cela ne sera peut-être pas nécessaire, car il n’est pas prévu que la signature d’accès partagé soit renouvelée. Toutefois, si vous avez un client qui effectue régulièrement des demandes par le biais de signatures d'accès partagé, le risque d'expiration est à prendre en compte. La principale considération consiste à trouver un équilibre entre la nécessité que la signature d’accès partagé ait une durée de vie limitée (comme indiqué plus haut) et la nécessité de veiller à ce que le client demande le renouvellement suffisamment tôt pour éviter une interruption due à une expiration de la signature avant le renouvellement effectif.
- **Faites attention à la date de début de la signature d’accès partagé** : si vous définissez la date de début d’une signature d’accès partagé sur **maintenant**, en raison du décalage d’horloge (différences constatées dans l’heure actuelle sur des machines différentes), des défaillances peuvent être observées par intermittence pendant les premières minutes. En règle générale, définissez une heure de début située au moins 15 minutes avant l’heure courante ou ne la définissez pas du tout, et elle sera alors valide immédiatement dans tous les cas. Généralement, le même principe s’applique également à l’heure d’expiration. N’oubliez pas que vous pouvez observer jusqu’à 15 minutes de décalage d’horloge (dans un sens ou dans l’autre) sur une demande. 
- **Soyez précis quant à la ressource pour laquelle vous voulez configurer l’accès** : il est recommandé de fournir à l’utilisateur les privilèges minimaux requis, ce qui s’inscrit dans les bonnes pratiques de sécurité. Si un utilisateur a besoin d'un accès en lecture à une seule entité, accordez-lui un accès en lecture à cette seule entité, plutôt qu'un accès en lecture/écriture/suppression à toutes les entités. Cela permet également d’atténuer les dégâts si une signature d’accès partagé est compromise, car son pouvoir est moindre entre les mains d’un attaquant.
- **N’utilisez pas toujours une signature d’accès partagé** : parfois, les risques associés à une opération particulière sur vos hubs d’événements l’emportent sur les avantages offerts par la signature d’accès partagé. Pour ces opérations, créez un service de niveau intermédiaire qui écrit dans vos hubs d’événements après avoir effectué la validation des règles métier, l’authentification et un audit.
- **Utilisez toujours HTTPS** : utilisez toujours HTTPS pour créer ou distribuer une signature d’accès partagé. Si une signature d’accès partagé est transmise sur HTTP et interceptée, un attaquant qui lance une attaque de type « attaque de l’intercepteur » (man-in-the-middle) peut lire la signature et s’en servir exactement comme l’utilisateur concerné aurait pu le faire, d’où le risque que les données sensibles soient compromises ou que les données soient altérées par l’utilisateur malveillant.

## <a name="configuration-for-shared-access-signature-authentication"></a>Configuration de l’authentification de signature d’accès partagé

Vous pouvez configurer la stratégie Shared Access Authorization sur les espaces de noms, les files d’attente ou les rubriques Service Bus. La configuration sur un abonnement Service Bus n’est actuellement pas prise en charge, mais vous pouvez utiliser les règles configurées dans des espaces de noms ou des rubriques pour sécuriser l’accès aux abonnements. 

![SAS](./media/service-bus-sas/service-bus-namespace.png)

Dans cette figure, les règles d’autorisation *manageRuleNS*, *sendRuleNS* et *listenRuleNS* s’appliquent à la file d’attente Q1 et à la rubrique T1, tandis que *listenRuleQ* et *sendRuleQ* s’appliquent uniquement à la file d’attente Q1, et *sendRuleT* uniquement à la rubrique T1.

## <a name="generate-a-shared-access-signature-token"></a>Générer un jeton de signature d’accès partagé

Les clients qui ont accès au nom d’une règle d’autorisation et à celui de ses clés de signature peuvent générer un jeton SAP. Le jeton est généré suite à l’élaboration d’une chaîne au format suivant :

```
SharedAccessSignature sig=<signature-string>&se=<expiry>&skn=<keyName>&sr=<URL-encoded-resourceURI>
```

- `se` - Délai d’expiration du jeton. Entier reflétant les secondes depuis l’époque `00:00:00 UTC` du 1er janvier 1970 (époque UNIX) lorsque le jeton expire.
- `skn` - Nom de la règle d’autorisation.
- `sr` - URI encodé par URL de la ressource faisant l’objet de l’accès.
- `sig` - Signature HMACSHA256 encodée par URL. Le calcul de hachage est similaire au code de pseudo suivant et retourne base64 de sortie binaire brute.

    ```
    urlencode(base64(hmacsha256(urlencode('https://<yournamespace>.servicebus.windows.net/') + "\n" + '<expiry instant>', '<signing key>')))
    ```

> [!IMPORTANT]
> Pour obtenir des exemples de génération d’un jeton SAS au moyen de différents langages de programmation, consultez [Générer un jeton SAS](/rest/api/eventhub/generate-sas-token). 


Le jeton contient les valeurs non hachées afin que le destinataire puisse recalculer le hachage avec les mêmes paramètres, en vérifiant que l’émetteur est en possession d’une clé de signature valide.

L’URI de ressource est l’URI complet de la ressource Service Bus à laquelle vous souhaitez accéder. Par exemple, `http://<namespace>.servicebus.windows.net/<entityPath>` ou `sb://<namespace>.servicebus.windows.net/<entityPath>` ; qui est, `http://contoso.servicebus.windows.net/contosoTopics/T1/Subscriptions/S3`. 

**L’URI doit être [encodée en pourcentage](/dotnet/api/system.web.httputility.urlencode).**

La règle de l’autorisation d’accès partagé utilisée pour la signature doit être configurée sur l’entité spécifiée par cette URI, ou par un de ses parents hiérarchiques. Par exemple, `http://contoso.servicebus.windows.net/contosoTopics/T1` ou `http://contoso.servicebus.windows.net` dans l’exemple précédent.

Un jeton SAP est valable pour toutes les ressources avec le préfixe `<resourceURI>` utilisé dans `signature-string`.


## <a name="regenerating-keys"></a>Régénération des clés

Il est recommandé de régénérer régulièrement les clés utilisées dans la stratégie Shared Access Authorization. Les emplacements des clés primaires et secondaires existent afin que vous puissiez permuter progressivement les clés. Si votre application utilise généralement la clé primaire, vous pouvez copier la clé primaire dans l’emplacement de la clé secondaire et alors seulement régénérer la clé primaire. La nouvelle valeur de clé primaire peut ensuite être configurée dans les applications clientes, qui bénéficient d’un accès continu à l’aide de l’ancienne clé primaire dans l’emplacement secondaire. Une fois que tous les clients sont mis à jour, vous pouvez régénérer la clé secondaire pour enfin mettre hors service l’ancienne clé primaire.

Si vous savez ou si vous suspectez qu’une clé est compromise et si vous devez révoquer les clés, vous pouvez régénérer la clé primaire et la clé secondaire d’une stratégie Shared Access Authorization et les remplacer par de nouvelles clés. Cette procédure annule tous les jetons signés avec les anciennes clés.

## <a name="shared-access-signature-authentication-with-service-bus"></a>Authentification par signature d’accès partagé avec Service Bus

Les scénarios décrits ci-après incluent la configuration des règles d’autorisation, la génération de jetons SAP et l’autorisation de client.

Pour visionner un exemple d’application Service Bus qui illustre la configuration et l’autorisation SAP, consultez [Authentification de la Signature d’accès partagé avec Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/ManagingEntities/SASAuthorizationRule).

## <a name="access-shared-access-authorization-rules-on-an-entity"></a>Accès aux règles d’autorisation d’accès partagé sur une entité

Utilisez l’opération obtenir/mettre à jour sur les files d’attente ou les rubriques dans les [bibliothèques de gestion pour Service Bus](service-bus-management-libraries.md) pour accéder aux règles d’autorisation d’accès partagé correspondantes ou les mettre à jour. Vous pouvez également ajouter les règles lors de la création des files d’attente ou des rubriques à l’aide de ces bibliothèques.

## <a name="use-shared-access-signature-authorization"></a>Utilisation de l’autorisation de la signature d’accès partagé (SAP)

Les applications utilisant l’un des kits de développement logiciel (SDK) Service Bus dans l’un des langages officiellement pris en charge tels que .NET, Java, JavaScript et Python peuvent utiliser l’autorisation SAS via les chaînes de connexion passées au constructeur client.

Les chaînes de connexion peuvent inclure un nom de règle (*SharedAccessKeyName*) et la clé de la règle (*SharedAccessKey*) ou un jeton émis précédemment (*SharedAccessSignature*). Lorsque ceux-ci sont présents dans la chaîne de connexion transmise à un constructeur ou une méthode de fabrique acceptant une chaîne de connexion, le fournisseur de jetons SAP est automatiquement créé et renseigné.

Pour utiliser une autorisation SAS avec des abonnements Service Bus, vous pouvez utiliser des clés SAS configurées sur un espace de noms Service Bus ou sur une rubrique.

## <a name="use-the-shared-access-signature-at-http-level"></a>Utilisation de la signature d’accès partagé (au niveau de HTTP)

Maintenant que vous savez comment créer des signatures d’accès partagé pour les entités dans Service Bus, vous êtes prêt à effectuer une requête HTTP POST :

```http
POST https://<yournamespace>.servicebus.windows.net/<yourentity>/messages
Content-Type: application/json
Authorization: SharedAccessSignature sr=https%3A%2F%2F<yournamespace>.servicebus.windows.net%2F<yourentity>&sig=<yoursignature from code above>&se=1438205742&skn=KeyName
ContentType: application/atom+xml;type=entry;charset=utf-8
```

N’oubliez pas que cela fonctionne pour tout. Vous pouvez créer une signature d’accès partagé pour une file d’attente, une rubrique ou un abonnement.

Si vous donnez un jeton SAS à un expéditeur ou un client, celui-ci ne dispose pas directement de la clé et il ne peut pas inverser le hachage pour l’obtenir. Ainsi, vous contrôlez le contenu auquel il peut accéder et pour quelle durée. Il est important de ne pas oublier que si vous modifiez la clé primaire dans la stratégie, les signatures d’accès partagé créées à partir de celle-ci ne sont plus valides.

## <a name="use-the-shared-access-signature-at-amqp-level"></a>Utilisation de la signature d’accès partagé (au niveau d’AMQP)

Dans la section précédente, vous avez vu comment utiliser le jeton SAS avec une requête HTTP POST pour envoyer des données au Service Bus. Comme vous le savez, vous pouvez accéder au Service Bus à l’aide du protocole AMQP (Advanced Message Queuing Protocol) qui est le protocole privilégié pour des raisons de performances dans de nombreux scénarios. L’utilisation de jetons SAS avec AMQP est décrite dans le document [AMQP Claim-Based Security Version 1.0](https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc), qui est à l’état de brouillon depuis 2013, mais prise en charge par Azure aujourd’hui.

Avant de commencer à envoyer des données vers Service Bus, le serveur de publication doit envoyer le jeton SAP dans un message AMQP à un nœud AMQP bien défini nommé **$cbs** (il s’agit d’une sorte de file d’attente « spéciale » que le service utilise pour acquérir et valider tous les jetons SAP). Le serveur de publication doit spécifier le champ **ReplyTo** dans le message AMQP. Il s’agit du nœud sur lequel le service répond au serveur de publication avec le résultat de la validation du jeton (système de demande/réponse simple entre le serveur de publication et le service). Ce nœud de réponse est créé « à la volée » en ce qui concerne la « création dynamique du nœud à distance », comme le décrit la spécification AMQP 1.0. Après avoir vérifié que le jeton SAS est valide, le serveur de publication peut continuer et commencer à envoyer des données au service.

Les étapes suivantes montrent comment envoyer le jeton SAP avec le protocole AMQP à l’aide de la bibliothèque [AMQP.NET Lite](https://github.com/Azure/amqpnetlite). C’est utile si vous ne pouvez pas utiliser le Kit de développement logiciel (SDK) Service Bus officiel (par exemple sur WinRT, .NET Compact Framework, .NET Micro Framework et Mono) pour le développement en C\#. Bien évidemment, cette bibliothèque est utile pour comprendre comment la sécurité basée sur les revendications fonctionne au niveau AMQP, tout comme vous avez pu voir comment cela fonctionne au niveau HTTP (avec une demande HTTP POST et le jeton SAP envoyé dans l’en-tête « Autorisation »). Si vous n’avez pas besoin de ces connaissances approfondies sur AMQP, vous pouvez utiliser le kit de développement logiciel (SDK) Service Bus officiel dans l’un des langages pris en charge tels que .NET, Java, JavaScript, Python et Go, qui le fera pour vous.

### <a name="c35"></a>C&#35;

```csharp
/// <summary>
/// Send claim-based security (CBS) token
/// </summary>
/// <param name="shareAccessSignature">Shared access signature (token) to send</param>
private bool PutCbsToken(Connection connection, string sasToken)
{
    bool result = true;
    Session session = new Session(connection);

    string cbsClientAddress = "cbs-client-reply-to";
    var cbsSender = new SenderLink(session, "cbs-sender", "$cbs");
    var cbsReceiver = new ReceiverLink(session, cbsClientAddress, "$cbs");

    // construct the put-token message
    var request = new Message(sasToken);
    request.Properties = new Properties();
    request.Properties.MessageId = Guid.NewGuid().ToString();
    request.Properties.ReplyTo = cbsClientAddress;
    request.ApplicationProperties = new ApplicationProperties();
    request.ApplicationProperties["operation"] = "put-token";
    request.ApplicationProperties["type"] = "servicebus.windows.net:sastoken";
    request.ApplicationProperties["name"] = Fx.Format("amqp://{0}/{1}", sbNamespace, entity);
    cbsSender.Send(request);

    // receive the response
    var response = cbsReceiver.Receive();
    if (response == null || response.Properties == null || response.ApplicationProperties == null)
    {
        result = false;
    }
    else
    {
        int statusCode = (int)response.ApplicationProperties["status-code"];
        if (statusCode != (int)HttpStatusCode.Accepted && statusCode != (int)HttpStatusCode.OK)
        {
            result = false;
        }
    }

    // the sender/receiver may be kept open for refreshing tokens
    cbsSender.Close();
    cbsReceiver.Close();
    session.Close();

    return result;
}
```

La méthode `PutCbsToken()` reçoit la *connexion* (instance de classe de connexion AMQP telle que fournie par la [bibliothèque AMQP .NET Lite](https://github.com/Azure/amqpnetlite)) qui représente la connexion TCP au service et le paramètre *sasToken* correspondant au jeton SAP à envoyer.

> [!NOTE]
> Il est important que la connexion soit créée avec le **mécanisme d’authentification SASL défini sur ANONYMOUS** (et non sur le paramètre par défaut PLAIN avec le nom d’utilisateur et le mot de passe utilisés lorsque vous n’avez pas besoin d’envoyer le jeton SAP).
>
>

Le serveur de publication crée ensuite deux liens AMQP pour envoyer le jeton SAP et recevoir la réponse (résultat de validation du jeton) depuis le service.

Le message AMQP inclut un ensemble de propriétés et plus d’informations qu’un simple message. Le jeton SAP est le corps du message (à l’aide de son constructeur). La propriété **« ReplyTo »** est définie sur le nom du nœud permettant de recevoir le résultat de validation sur le lien du récepteur (vous pouvez modifier son nom si vous le souhaitez, il sera créé dynamiquement par le service). Les trois dernières propriétés personnalisées / application sont utilisées par le service pour indiquer le type d’opération à exécuter. Comme l’indique le projet de spécification CBS, il doit s’agir du **nom de l’opération** (« put-token »), du **type de jeton** (dans ce cas, `servicebus.windows.net:sastoken`) et du **« nom » de l’audience** à laquelle le jeton s’applique (entité entière).

Après avoir envoyé le jeton SAP sur le lien de l’expéditeur, le serveur de publication doit lire la réponse sur le lien du récepteur. La réponse est un simple message AMQP avec une propriété d’application nommée **« status-code »** qui peut contenir les mêmes valeurs qu’un code d’état HTTP.

## <a name="rights-required-for-service-bus-operations"></a>Droits requis pour les opérations Service Bus

Le tableau suivant affiche les droits d’accès requis pour effectuer diverses opérations sur les ressources Service Bus.

| Opération | Revendication obligatoire | Étendue de la revendication |
| --- | --- | --- |
| **Espace de noms** | | |
| Configure une règle d’autorisation dans un espace de noms |Gérer |N’importe quelle adresse d’espace de noms |
| **Registre de service** | | |
| Énumération des stratégies privées |Gérer |N’importe quelle adresse d’espace de noms |
| Commencer à écouter sur un espace de noms |Écouter |N’importe quelle adresse d’espace de noms |
| Envoyer des messages à un écouteur sur un espace de noms |Envoyer |N’importe quelle adresse d’espace de noms |
| **File d'attente** | | |
| Créer une file d’attente |Gérer |N’importe quelle adresse d’espace de noms |
| Suppression d'une file d'attente |Gérer |N’importe quelle adresse de file d’attente valide |
| Énumérer les files d’attente |Gérer |/$Resources/Queues |
| Obtenir la description de file d’attente |Gérer |N’importe quelle adresse de file d’attente valide |
| Configure une règle d’autorisation pour une file d’attente |Gérer |N’importe quelle adresse de file d’attente valide |
| Envoyer dans la file d’attente |Envoyer |N’importe quelle adresse de file d’attente valide |
| Réception des messages d'une file d'attente |Écouter |N’importe quelle adresse de file d’attente valide |
| Abandonner ou terminer des messages après la réception du message en mode de verrouillage |Écouter |N’importe quelle adresse de file d’attente valide |
| Différer un message pour une récupération ultérieure |Écouter |N’importe quelle adresse de file d’attente valide |
| Mettre un message au rebut |Écouter |N’importe quelle adresse de file d’attente valide |
| Obtenir l’état associé à une session de file d’attente |Écouter |N’importe quelle adresse de file d’attente valide |
| Obtenir l’état associé à une session de file d’attente de message |Écouter |N’importe quelle adresse de file d’attente valide |
| Planifier un message pour une remise ultérieure |Écouter | N’importe quelle adresse de file d’attente valide
| **Rubrique** | | |
| Création d'une rubrique |Gérer |N’importe quelle adresse d’espace de noms |
| Supprimer une rubrique |Gérer |N’importe quelle adresse de rubrique valide |
| Énumérer les rubriques |Gérer |/$Resources/Topics |
| Obtenir la description de la rubrique |Gérer |N’importe quelle adresse de rubrique valide |
| Configure une règle d’autorisation pour une rubrique |Gérer |N’importe quelle adresse de rubrique valide |
| Envoyer à la rubrique |Envoyer |N’importe quelle adresse de rubrique valide |
| **Abonnement** | | |
| Création d’un abonnement |Gérer |N’importe quelle adresse d’espace de noms |
| Supprimer l’abonnement |Gérer |../myTopic/Subscriptions/mySubscription |
| Énumérer les abonnements |Gérer |../myTopic/Subscriptions |
| Obtenir la description de l’abonnement |Gérer |../myTopic/Subscriptions/mySubscription |
| Abandonner ou terminer des messages après la réception du message en mode de verrouillage |Écouter |../myTopic/Subscriptions/mySubscription |
| Différer un message pour une récupération ultérieure |Écouter |../myTopic/Subscriptions/mySubscription |
| Mettre un message au rebut |Écouter |../myTopic/Subscriptions/mySubscription |
| Obtenir l’état associé à une session de rubrique |Écouter |../myTopic/Subscriptions/mySubscription |
| Définir l’état associé à une session de rubrique |Écouter |../myTopic/Subscriptions/mySubscription |
| **Règles** | | |
| Créer une règle |Gérer |../myTopic/Subscriptions/mySubscription |
| Supprimer une règle |Gérer |../myTopic/Subscriptions/mySubscription |
| Énumérer des règles |Gérer ou écouter |.. /myTopic/Subscriptions/mySubscription/Rules

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la messagerie Service Bus, voir les rubriques suivantes.

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Utilisation des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[Azure portal]: https://portal.azure.com
