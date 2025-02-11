---
title: Azure Database pour PostgreSQL - Notes de publication sur le serveur flexible
description: Notes de publication sur Azure Database pour PostgreSQL - Serveur flexible
author: sr-msft
ms.author: srranga
ms.custom: references_regions
ms.service: postgresql
ms.topic: overview
ms.date: 06/23/2021
ms.openlocfilehash: 87af6f9764c2ab01b0e0d02d8eb4a6c7342ca31c
ms.sourcegitcommit: 5be51a11c63f21e8d9a4d70663303104253ef19a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/25/2021
ms.locfileid: "112894623"
---
# <a name="release-notes---azure-database-for-postgresql---flexible-server"></a>Notes de publication - Azure Database pour PostgreSQL - Serveur flexible

Cette page fournit les toutes dernières informations concernant les ajouts de fonctionnalités, les versions de moteur prises en charge et les extensions. Elle inclut également toutes les annonces relatives au serveur flexible - PostgreSQL.

> [!IMPORTANT]
> Le serveur flexible Azure Database pour PostgreSQL est en préversion

## <a name="release-june-2021"></a>Version : juin 2021

* Prise en charge des [dernières versions mineures de PostgreSQL](./concepts-supported-versions.md) 13.3, 12.7 et 11.12 avec les nouvelles créations de serveur<sup>$</sup>.
* Support pour de [nouvelles régions](overview.md#azure-regions), notamment Australie Sud-Est, Brésil Sud, Corée Centre, Norvège Est, Afrique du Sud Nord, Suisse Nord, Émirats arabes unis Nord et USA Ouest.
* Support des fonctionnalités de [basculement à la demande](./concepts-high-availability.md#on-demand-failover), notamment le basculement forcé et le basculement planifié pour les déploiements à haute disponibilité redondants dans une zone.
* La prise en charge de l’[authentification SCRAM](how-to-connect-scram.md) pour toutes les versions majeures avec le nouveau serveur crée <sup>$</sup>.
* La prise en charge de `pg_prewarm` doit être préchargée à l’aide de `shared_preload_libraries` avec les nouvelles créations de serveur<sup>$</sup>.
* Prise en charge de l’extension lo. Consultez la [page des extensions](./concepts-extensions.md) pour les versions prises en charge avec chaque version majeure <sup>$</sup>.
* Plusieurs corrections de bogues, et amélioration de la stabilité et des performances<sup>$</sup>.
  
<sup> **$** </sup> Vos serveurs existants seront automatiquement mis à niveau vers la dernière version mineure prise en charge et les nouvelles fonctionnalités seront également activées lors de la future fenêtre de maintenance de votre serveur.

## <a name="release-may-2021"></a>Version : mai 2021

* Prise en charge de la [version majeure 13 de PostgreSQL](./concepts-supported-versions.md).
* Prise en charge des extensions, notamment pg_partman, pg_cron et pgaudit. Consultez la [page des extensions](./concepts-extensions.md) pour les versions prises en charge avec chaque version majeure.
* Plusieurs corrections de bogues, et amélioration de la stabilité et des performances.

## <a name="release-april-2021"></a>Version : avril 2021

* Prise en charge des [dernières versions mineures de PostgreSQL](./concepts-supported-versions.md) 12.6 et 11.11 avec les nouvelles créations de serveur
* Prise en charge de la [zone DNS privée](./concepts-networking.md#private-access-vnet-integration) du réseau virtuel
* Prise en charge du choix de la zone de disponibilité pendant l’opération de récupération jusqu’à une date et heure
* Prise en charge de nouvelles [régions](./overview.md#azure-regions), y compris Australie Est, Canada Centre et France Centre
* Prise en charge du pooler de connexions [PgBouncer intégré](./concepts-pgbouncer.md) 
<!--- * Support for [pglogical](https://github.com/2ndQuadrant/pglogical) extension version 2.3.2. -->
* [Performances intelligentes](concepts-query-store.md) en préversion
* Plusieurs corrections de bogues et amélioration de la stabilité et des performances

## <a name="contacts"></a>Contacts

Pour toute question ou suggestion au sujet du serveur flexible Azure Database pour PostgreSQL, envoyez un e-mail à l’équipe Azure Database pour PostgreSQL ([@Ask Azure DB pour PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com)). Notez que cette adresse e-mail n’est pas un alias de support technique.

En outre, tenez compte des points de contact suivants le cas échéant :

- Pour contacter le support technique Azure, [émettez un ticket à partir du Portail Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- Pour résoudre un problème relatif à votre compte, enregistrez une [demande de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) sur le portail Azure.
- Pour donner votre avis ou demander de nouvelles fonctionnalités, créez une entrée via [UserVoice](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).
  

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez lu l'introduction au mode de déploiement Azure Database pour PostgreSQL - Serveur flexible, vous êtes prêt à créer votre premier serveur : [Créer une instance Azure Database pour PostgreSQL - Serveur flexible à l’aide du portail Azure](./quickstart-create-server-portal.md)