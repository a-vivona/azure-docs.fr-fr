---
title: Sécurité des données des Journaux Azure Monitor | Microsoft Docs
description: Découvrez comment les Journaux Azure Monitor protègent votre vie privée et sécurisent vos données.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/11/2020
ms.openlocfilehash: c6b4f6b9ce5486efcba4a7cb291a1720e25adf62
ms.sourcegitcommit: 2d412ea97cad0a2f66c434794429ea80da9d65aa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/14/2021
ms.locfileid: "122533106"
---
# <a name="azure-monitor-logs-data-security"></a>Sécurité des données des Journaux Azure Monitor
Ce document est destiné à fournir des informations propres aux [Journaux Azure Monitor](../logs/data-platform-logs.md) en complément des informations disponibles sur le [Centre de confidentialité Azure](https://www.microsoft.com/en-us/trust-center?rtc=1).  

Cet article décrit les procédures de collecte, de traitement et de sécurisation des données par Azure Monitor. Vous pouvez utiliser des agents pour la connexion au service Web, utiliser Operations Manager pour collecter des données opérationnelles ou récupérer des données à partir des diagnostics Azure pour une utilisation par Azure Monitor. 

Les Journaux Azure Monitor gèrent vos données basées sur le cloud en toute sécurité en utilisant les méthodes suivantes :

* Ségrégation des données
* Conservation des données
* Sécurité physique
* Gestion des incidents
* Conformité
* Certifications relatives aux normes de sécurité

Vous pouvez également utiliser des fonctionnalités de sécurité supplémentaires intégrées à Azure Monitor. Ces fonctionnalités nécessitent une gestion plus poussée de la part de l’administrateur. 
* Clés (de sécurité) gérées par le client
* Stockage privé Azure
* Réseau de liaison privée 
* Azure prend en charge les limites d’accès définies par Azure Lockbox

Contactez-nous pour toute question, suggestion ou remarque concernant les informations fournies ci-dessous, y compris notre politique de sécurité dans [Options de support Azure](https://azure.microsoft.com/support/options/).

## <a name="sending-data-securely-using-tls-12"></a>Envoi sécurisé de données via TLS 1.2 

Pour garantir la sécurité des données en transit vers Azure Monitor, nous vous encourageons vivement à configurer l’agent de façon à utiliser au moins le protocole TLS (Transport Layer Security) 1.2. Les versions antérieures de TLS/SSL (Secure Sockets Layer) se sont avérées vulnérables et bien qu’elles fonctionnent encore pour assurer la compatibilité descendante, elles sont **déconseillées** et le secteur évolue rapidement vers un arrêt de la prise en charge de ces anciens protocoles. 

Le [PCI Security Standards Council](https://www.pcisecuritystandards.org/) a défini une [échéance au 30 juin 2018](https://www.pcisecuritystandards.org/pdfs/PCI_SSC_Migrating_from_SSL_and_Early_TLS_Resource_Guide.pdf) pour désactiver les versions antérieures de TLS/SSL et effectuer une mise à niveau vers des protocoles plus sécurisés. Une fois qu’Azure aura arrêté la prise en charge des versions héritées, si vos agents ne peuvent pas communiquer via au moins le protocole TLS 1.2, vous ne serez plus en mesure d’envoyer des données aux Journaux Azure Monitor. 

Nous vous recommandons de NE PAS définir explicitement votre agent pour qu’il utilise uniquement TLS 1.2, sauf si cela est absolument nécessaire. Il est préférable d’autoriser l’agent à détecter, négocier et tirer parti des normes de sécurité futures. Dans le cas contraire, vous risquez de manquer la sécurité accrue des normes plus récentes et peut-être de rencontrer des problèmes si TLS 1.2 est déprécié en faveur de ces dernières normes.    

### <a name="platform-specific-guidance"></a>Instructions spécifiques à la plateforme

|Plateforme/Langage | Support | Informations complémentaires |
| --- | --- | --- |
|Linux | Les distributions de Linux s’appuient généralement sur [OpenSSL](https://www.openssl.org) pour la prise en charge de TLS 1.2.  | Vérifiez [OpenSSL Changelog](https://www.openssl.org/news/changelog.html) pour vous assurer que votre version d’OpenSSL est prise en charge.|
| Windows 8.0 - 10 | Pris en charge, activé par défaut. | Pour confirmer que vous utilisez toujours les [paramètres par défaut](/windows-server/security/tls/tls-registry-settings).  |
| Windows Server 2012 - 2016 | Pris en charge, activé par défaut. | Pour confirmer que vous utilisez toujours les [paramètres par défaut](/windows-server/security/tls/tls-registry-settings) |
| Windows 7 SP1 et Windows Server 2008 R2 SP1 | Pris en charge, mais non activé par défaut. | Consultez la page [Paramètres de Registre de TLS](/windows-server/security/tls/tls-registry-settings) pour plus d’informations sur l’activation.  |

## <a name="data-segregation"></a>Ségrégation des données
Une fois que vos données ont été ingérées par Azure Monitor, elles sont séparées logiquement sur chaque composant dans l’ensemble du service. Toutes les données sont balisées en fonction de l’espace de travail. Ce balisage est conservé tout au long du cycle de vie des données, et il est appliqué dans chaque couche du service. Vos données sont stockées dans une base de données dédiée dans le cluster de stockage de la région que vous avez sélectionnée.

## <a name="data-retention"></a>Conservation des données
Les données indexées de recherche dans les journaux sont stockées et conservées en fonction de votre plan de tarification. Pour plus d’informations, consultez [Tarification - Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).

Microsoft conservera vos données conformément aux termes de votre [contrat d’abonnement](https://azure.microsoft.com/support/legal/subscription-agreement/).  La suppression des données client n’entraîne pas de destruction de lecteurs physiques.  

Le tableau ci-après répertorie certaines des solutions disponibles et fournit des exemples du type de données qu’elles collectent.

| **Solution** | **Types de données** |
| --- | --- |
| Capacity and Performance |Données et métadonnées de performances |
| Update Management |Métadonnées et données d'état |
| Gestion du journal |Journaux des événements défini par l’utilisateur, journaux des événements Windows et/ou journaux d’activité IIS |
| Suivi des modifications |Inventaire des logiciels, métadonnées de service Windows et de démon Linux, et métadonnées de fichiers Windows/Linux |
| Évaluation de SQL et d'Active Directory |Données WMI, données du Registre, données de performances et résultats de vues de gestion dynamique de SQL Server |

Le tableau suivant répertorie des exemples de types de données :

| **Type de données** | **Fields** |
| --- | --- |
| Alerte |Alert Name, Alert Description, BaseManagedEntityId, Problem ID, IsMonitorAlert, RuleId, ResolutionState, Priority, Severity, Category, Owner, ResolvedBy, TimeRaised, TimeAdded, LastModified, LastModifiedBy, LastModifiedExceptRepeatCount, TimeResolved, TimeResolutionStateLastModified, TimeResolutionStateLastModifiedInDB, RepeatCount |
| Configuration |CustomerID, AgentID, EntityID, ManagedTypeID, ManagedTypePropertyID, CurrentValue, ChangeDate |
| Événement |EventId, EventOriginalID, BaseManagedEntityInternalId, RuleId, PublisherId, PublisherName, FullNumber, Number, Category, ChannelLevel, LoggingComputer, EventData, EventParameters, TimeGenerated, TimeAdded <br>**Remarque :** quand vous consignez des événements avec des champs personnalisés dans le journal des événements Windows, Log Analytics les collecte. |
| Métadonnées |BaseManagedEntityId, ObjectStatus, OrganizationalUnit, ActiveDirectoryObjectSid, PhysicalProcessors, NetworkName, IPAddress, ForestDNSName, NetbiosComputerName, VirtualMachineName, LastInventoryDate, HostServerNameIsVirtualMachine, IP Address, NetbiosDomainName, LogicalProcessors, DNSName, DisplayName, DomainDnsName, ActiveDirectorySite, PrincipalName, OffsetInMinuteFromGreenwichTime |
| Performances |ObjectName, CounterName, PerfmonInstanceName, PerformanceDataId, PerformanceSourceInternalID, SampleValue, TimeSampled, TimeAdded |
| State |StateChangeEventId, StateId, NewHealthState, OldHealthState, Context, TimeGenerated, TimeAdded, StateId2, BaseManagedEntityId, MonitorId, HealthState, LastModified, LastGreenAlertGenerated, DatabaseTimeModified |

## <a name="physical-security"></a>Sécurité physique
Azure Monitor est géré par le personnel de Microsoft. Toutes les activités sont journalisées et peuvent faire l’objet d’un audit. Azure Monitor est utilisé sous la forme d’un Service Azure et satisfait à toutes les exigences en matière de conformité et de sécurité Azure. Vous pouvez afficher plus d'informations sur la sécurité physique des ressources Azure à la page 18 du document [Vue d'ensemble de la sécurité de Microsoft Azure](https://download.microsoft.com/download/6/0/2/6028B1AE-4AEE-46CE-9187-641DA97FC1EE/Windows%20Azure%20Security%20Overview%20v1.01.pdf). Les droits d’accès physique aux zones sécurisées sont modifiés en l’espace d’un jour ouvrable pour toute personne n’assumant plus de responsabilité en relation avec le service Azure Monitor, transfert et arrêt compris. Pour en savoir plus sur notre infrastructure physique globale, découvrez les [Centre de données Microsoft](https://azure.microsoft.com/global-infrastructure/).

## <a name="incident-management"></a>Gestion des incidents
Azure Monitor inclut un processus de gestion des incidents auquel tous les services Microsoft adhèrent. Pour résumer :

* Nous appliquons un modèle de responsabilité partagée en vertu duquel une partie de la sécurité relève de la responsabilité de Microsoft, et une autre de celle du client.
* Nous gérons les incidents de sécurité liés à Azure :
  * Nous lançons une investigation lors de la détection d’un incident.
  * Nous évaluons l’impact et la gravité d’un incident via l’intervention d’un membre de l’équipe chargée de répondre aux appels relatifs aux incidents. Selon les éléments de preuve apportés, l’évaluation peut ou non entraîner une réaffectation à l’équipe chargée de répondre aux problèmes de sécurité.
  * Nous diagnostiquons un incident en demandant à des experts en réponse aux problèmes de sécurité de mener les investigations informatiques ou techniques nécessaire, et d’identifier des stratégies d’endiguement, d’atténuation et de contournement. Si l’équipe de sécurité estime que des données client risquent d’être exposées à une personne illégitime ou non autorisée, le processus de notification d’incident au client commence en parallèle.  
  * Nous stabilisons l’incident et assurons la récupération. L’équipe de réponse aux incidents crée un plan de récupération pour atténuer le problème. Des mesures d’endiguement de crise telles que la mise en quarantaine des systèmes affectés peuvent être prises immédiatement et parallèlement au diagnostic. Des mesures d’atténuation à plus long terme peuvent être planifiées, qui se interviennent une fois passé le risque immédiat.  
  * Nous clôturons l’incident, puis établissons un bilan. L’équipe de réponse aux incidents dresse un bilan décrivant l’incident en détail, dans le but de réviser les stratégies, procédures et processus de façon appropriée afin d’éviter que l’événement se reproduise.
* Nous informons les clients des incidents de sécurité :
  * Nous identifions les clients affectés, et communiquons à toute personne concernée une notice aussi détaillée que possible.
  * Nous rédigeons une notice pour fournir aux clients des informations suffisamment détaillées pour qu’ils puissent mener une investigation de leur côté et respecter les engagements qu’ils ont envers leurs utilisateurs finaux sans retarder indûment le processus de notification.
  * Nous confirmons et déclarons l’incident si nécessaire.
  * Nous avertissons les clients à l’aide d’une notification d’incident sans délai déraisonnable, et conformément aux obligations légales ou contractuelles applicables. Les notifications d’incidents de sécurité sont adressées à un ou plusieurs des administrateurs du client par tout moyen que Microsoft juge opportun, y compris par courrier électronique.
* Nous assurons la formation et la préparation de nos équipes :
  * Le personnel de Microsoft est tenu de suivre une formation en matière de sécurité et de sensibilisation, qui l’aide à identifier et à signaler d’éventuels problèmes liés à la sécurité.  
  * Les opérateurs du service Microsoft Azure sont soumis à des obligations de formation supplémentaires en relation avec leur accès aux systèmes sensibles d’hébergement des données client.
  * Le personnel de réponse aux problèmes de sécurité de Microsoft reçoit une formation spécialisée en relation avec les rôles qui lui sont confiés.

Bien que cela soit très rare, Microsoft informera chaque client sous 24 heures en cas de perte significative de ses données. 

Pour plus d’informations sur la façon dont Microsoft répond aux incidents de sécurité, voir [Microsoft Azure Security Response in the Cloud](https://gallery.technet.microsoft.com/Azure-Security-Response-in-dd18c678/file/150826/4/Microsoft%20Azure%20Security%20Response%20in%20the%20cloud.pdf) (en anglais).

## <a name="compliance"></a>Conformité
Le programme de sécurité et de gouvernance des informations de l’équipe de maintenance et de développement logiciel Azure Monitor prend en charge ses besoins commerciaux et respecte les lois et réglementations telles que décrites sur [Centre de confidentialité Microsoft Azure](https://azure.microsoft.com/support/trust-center/) et [Microsoft Trust Center Compliance](https://www.microsoft.com/en-us/trustcenter/compliance/default.aspx). Y est également décrite la façon dont Azure Monitor fixe les exigences de sécurité, identifie les contrôles de sécurité et gère et analyse les risques. Chaque année, nous révisons les stratégies, normes, procédures et directives.

Chaque membre de l’équipe de développement reçoit une formation formelle en matière de sécurité des applications. En interne, nous utilisons un système de contrôle de version pour le développement de logiciels. Chaque projet de logiciel est protégé par le système de contrôle de version.

Microsoft dispose d’une équipe dédiée à la sécurité et à la conformité, qui surveille et évalue tous les services en son sein. Les responsables de la sécurité des informations qui constituent l’équipe ne sont pas associés aux équipes d’ingénierie qui développent Log Analytics. Les responsables de la sécurité ont leur propre chaîne de gestion, et effectuent des évaluations indépendantes des produits et services pour s’assurer de la sécurité et de la conformité de ceux-ci.

Le conseil d’administration de Microsoft est informé via un rapport annuel de l’ensemble des programmes relatifs à la sécurité des informations mis en œuvre au sein de Microsoft.

L’équipe de maintenance et de développement logiciel de Log Analytics travaille en étroite collaboration avec les équipes Microsoft chargées des aspects juridiques et de conformité, ainsi qu’avec d’autres partenaires du secteur, afin d’acquérir un éventail de certifications.

## <a name="certifications-and-attestations"></a>Certifications et attestations
Azure Log Analytics répond aux exigences suivantes :

* [ISO/IEC 27001](https://www.iso.org/iso/home/standards/management-standards/iso27001.htm)
* [ISO/IEC 27018:2014](https://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=61498)
* [ISO 22301](https://azure.microsoft.com/blog/iso22301/)
* [Normes de sécurité sur les données de l’industrie des cartes de paiement (PCI DSS)](https://www.microsoft.com/en-us/TrustCenter/Compliance/PCI) édictées par le PCI Security Standards Council (conseil de normalisation pour la sécurité des données de l’ICP).
* [SOC 1 Type 1 et SOC 2 Type 1](https://www.microsoft.com/en-us/TrustCenter/Compliance/SOC1-and-2).
* [HIPAA et Hi-TECH](/compliance/regulatory/offering-hipaa-hitech) pour les organisations qui disposent d’un contrat HIPAA Business Associate Agreement
* Critères de conception communs Windows
* Microsoft Trustworthy Computing
* En tant que service Azure, Azure Monitor utilise des composants conformes aux exigences de conformité d’Azure. Pour en savoir plus, voir le site du [Microsoft Trust Center](https://www.microsoft.com/en-us/trustcenter/compliance/default.aspx).

> [!NOTE]
> Dans certaines certifications/attestations, les Journaux Azure Monitor sont listés sous leur ancien nom : *Insights opérationnels*.
>
>

## <a name="cloud-computing-security-data-flow"></a>Flux de données de sécurité du cloud computing
Le diagramme suivant montre une architecture de sécurité cloud comme étant le flux d’informations provenant de votre entreprise et la manière dont il est sécurisé à mesure qu’il progresse vers Azure Monitor, avant son affichage final dans le Portail Azure. Des informations complémentaires sur chaque étape sont présentées à suite du diagramme.

![Image de la collecte de données et de la sécurité des Journaux Azure Monitor](./media/data-security/log-analytics-data-security-diagram.png)

## <a name="1-sign-up-for-azure-monitor-and-collect-data"></a>1. Inscription à Azure Monitor et à la collecte des données
Pour que votre organisation puisse envoyer des données aux Journaux Azure Monitor, vous configurez un agent Windows ou Linux s’exécutant sur des machines virtuelles Azure, ou sur des ordinateurs virtuels ou physiques de votre environnement ou d’un autre fournisseur de services cloud.  Si vous utilisez Operations Manager, vous configurez l’agent Operations Manager à partir du groupe d’administration. Les utilisateurs (vous-même, d’autres utilisateurs ou un groupe de personnes) créent un ou plusieurs espaces de travail Log Analytics et inscrivent des agents à l’aide de l’un des comptes suivants :

* [ID d'organisation](../../active-directory/fundamentals/sign-up-organization.md)
* [Compte Microsoft - Outlook, Office Live, MSN](https://account.microsoft.com/account)

Un espace de travail Log Analytics correspond à l’emplacement où les données sont collectées, agrégées, analysées et présentées. Un espace de travail sert essentiellement à partitionner les données, et chaque espace de travail est unique. Par exemple, vous pouvez choisir de gérer vos données de production avec un espace de travail, et vos données de test avec un autre espace de travail. En outre, les espaces de travail aident un administrateur à contrôler l’accès des utilisateurs aux données. Chaque espace de travail peut être associé à plusieurs comptes d’utilisateur, et chaque compte d’utilisateur peut accéder à plusieurs espaces de travail Log Analytics. Vous créez des espaces de travail en fonction d’une région de centre de données.

Pour Operations Manager, le groupe de gestion établit une connexion avec le service Azure Monitor. Vous configurez ensuite les systèmes gérés par un agent dans le groupe d’administration qui sont autorisés à collecter et envoyer des données au service. Selon la solution que vous avez activée, les données de ces solutions sont soit envoyées directement du serveur de gestion Operations Manager au service Azure Monitor, soit, en raison du volume de données collectées par le système géré par l’agent, envoyées directement au service à partir de l’agent. Dans le cas des systèmes non supervisés par Operations Manager, chaque système se connecte directement au service Azure Monitor en toute sécurité.

Toutes les communications entre les systèmes connectés et le service Azure Monitor sont chiffrées. Le protocole TLS (HTTPS) est utilisé pour le chiffrement.  Le processus Microsoft SDL est suivi pour s’assurer que Log Analytics est à jour avec les dernières avancées en matière de protocoles cryptographiques.

Chaque type d’agent collecte des données de journal pour Azure Monitor. Le type de données collectées dépend de la configuration de votre espace de travail et d’autres fonctionnalités d’Azure Monitor. 

## <a name="2-send-data-from-agents"></a>2. Envoyer des données à partir d'agents
Vous enregistrez tous les types d’agents avec une clé d’inscription et une connexion sécurisée est établie entre l’agent et le service Azure Monitor à l’aide de l’authentification par certificat et TLS avec le port 443. Azure Monitor utilise un magasin de secrets pour générer et gérer les clés. Les clés privées sont remplacées tous les 90 jours, stockées dans Azure, et gérées par des opérations Azure qui suivent des pratiques réglementaires et de conformité strictes.

Avec Operations Manager, le groupe d’administration qui est inscrit auprès d’un espace de travail Log Analytics établit une connexion HTTPS sécurisée avec un serveur d’administration Operations Manager.

Pour les agents Windows ou Linux s’exécutant sur des machines virtuelles Azure, une clé de stockage en lecture seule est utilisée pour lire des événements de diagnostic dans des tables Azure.  

Tout agent rendant compte à un groupe de gestion d’Operations Manager qui est intégré à Azure Monitor, si le serveur de gestion ne parvient pas à communiquer avec le service pour une raison quelconque, les données collectées sont stockées localement dans un cache temporaire du serveur de gestion.   Les données feront alors l’objet d’une nouvelle tentative d’envoi toutes les huit minutes pendant deux heures.  Dans le cas des données qui contournent le serveur de gestion et qui sont envoyées directement à Azure Monitor, le comportement est cohérent avec l’agent Windows.  

Les données mises en cache de l’agent Windows ou du serveur d’administration sont protégées par le magasin d’informations d’identification du système d’exploitation. Si le service ne peut pas traiter les données au bout de deux heures, les agents mettent celles-ci en file d’attente. En cas de saturation de la file d’attente, l’agent commence à supprimer certains types de données, en commençant par les données de performances. La limite de file d’attente de l’agent est une clé de Registre que vous pouvez modifier si nécessaire. Étant donné que les données collectées sont compressées et envoyées au service en contournant les bases de données du groupe d’administration Operations Manager, aucune charge n’est ajoutée à ces données. Une fois que les données collectées sont envoyées, elles sont supprimées du cache.

Comme décrit ci-dessus, les données provenant d’agents du serveur d’administration ou d’agents connectés directement sont envoyées par le biais du protocole TLS aux centres de données Microsoft Azure. Vous pouvez également utiliser ExpressRoute pour sécuriser davantage les données. ExpressRoute vous permet de vous connecter directement à Azure à partir de votre réseau étendu (WAN) existant, comme un VPN MPLS fourni par un fournisseur de services réseau. Pour plus d’informations, voir [ExpressRoute](https://azure.microsoft.com/services/expressroute/).

## <a name="3-the-azure-monitor-service-receives-and-processes-data"></a>3. Le service Azure Monitor reçoit et traite les données
Le service Azure Monitor s’assure que les données entrantes proviennent d’une source approuvée en validant des certificats et l’intégrité des données à l’aide de l’authentification Azure. Les données brutes non traitées sont ensuite stockées dans un hub d’événements Azure de la région où les données finissent par être stockées au repos. Le type des données stockées dépend des types de solutions qui ont été importées et utilisées pour collecter des données. Ensuite, le service Azure Monitor traite les données brutes et les ingère dans la base de données.

La période de rétention des données collectées stockées dans la base de données varie selon le plan de tarification. Dans le cas du niveau *gratuit*, les données collectées restent disponibles pendant sept jours. Dans le cas du niveau *payant*, les données collectées restent disponibles pendant 31 jours par défaut, mais cette durée peut être étendue à 730 jours. Les données au repos sont chiffrées et stockées dans le Stockage Azure pour préserver leur confidentialité et permettre leur réplication dans la région locale à l’aide du stockage localement redondant (LRS). Pendant les deux dernières semaines, les données sont également stockées dans le cache sur disque SSD et ce cache est chiffré.

Les données du stockage de base de données ne sont pas modifiables une fois ingérées, mais sont supprimables par le [chemin de l’API *purge*](personal-data-mgmt.md#delete). Bien que les données ne soient pas modifiables, certaines certifications exigent qu’elles soient immuables et ne puissent être ni modifiées ni supprimées dans le stockage. L’immuabilité des données peut être obtenue à l’aide de [l’exportation de données](logs-data-export.md) dans un compte de stockage configuré comme [stockage immuable](../../storage/blobs/immutable-policy-configure-version-scope.md).

## <a name="4-use-azure-monitor-to-access-the-data"></a>4. Utiliser Azure Monitor pour accéder aux données
Pour accéder à votre espace de travail Log Analytics, vous vous connectez au portail Azure à l’aide du compte professionnel ou d’un compte Microsoft que vous avez configuré précédemment. La totalité du trafic entre le Portail et le service Azure Monitor est envoyée par le biais d’un canal HTTPS sécurisé. Lorsque vous utilisez le portail, un ID de session est généré sur le client utilisateur (navigateur web), et les données sont stockées dans un cache local jusqu’à la fin de la session. Ensuite, le cache est supprimé. Les cookies côté client qui ne contiennent pas d’informations d’identification personnelle ne sont pas supprimés automatiquement. Les cookies de session sont marqués HTTPOnly et sécurisés. Après une période d’inactivité prédéfinie, la session du portail Azure prend fin.


## <a name="additional-security-features"></a>Fonctionnalités de sécurité supplémentaires
Vous pouvez utiliser ces fonctionnalités de sécurité supplémentaires pour renforcer la sécurité de votre environnement Azure Monitor. Ces fonctionnalités nécessitent une gestion plus poussée de la part de l’administrateur. 
- [Clés (de sécurité) gérées par le client](../logs/customer-managed-keys.md) : Vous pouvez utiliser des clés gérées par le client pour chiffrer les données envoyées à vos espaces de travail Log Analytics. Cela requiert l’utilisation d’Azure Key Vault. 
- [Stockage privé/managé par le client](./private-storage.md) : Gérez votre compte de stockage chiffré personnellement et indiquez à Azure Monitor de l’utiliser pour stocker les données de surveillance 
- [Mise en réseau d’Azure Private Link](./private-link-security.md) : Azure Private Link vous permet de lier en toute sécurité les services PaaS Azure (notamment Azure Monitor) à votre réseau virtuel à l’aide de points de terminaison privés. 
- [Azure Customer Lockbox](../../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-preview) : Customer Lockbox pour Microsoft Azure fournit une interface dans laquelle les clients peuvent vérifier et approuver/refuser les demandes d’accès aux données client. Il est utilisé lorsqu’un ingénieur Microsoft doit accéder aux données client dans le cadre d’une demande de support.


## <a name="next-steps"></a>Étapes suivantes
* [Consultez les différents types de données que vous pouvez collecter dans Azure Monitor](../monitor-reference.md).
