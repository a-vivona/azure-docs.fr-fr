---
title: Sauvegarde et restauration - Azure Database pour MySQL
description: Apprenez-en davantage sur la restauration et les sauvegardes automatiques de votre serveur Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 3/27/2020
ms.custom: references_regions
ms.openlocfilehash: a3ef9a8f56f091e7be93e940f9e359adef15cb7f
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122563162"
---
# <a name="backup-and-restore-in-azure-database-for-mysql"></a>Sauvegarde et restauration dans Azure Database pour MySQL

[!INCLUDE[applies-to-mysql-single-server](includes/applies-to-mysql-single-server.md)]

Azure Database pour MySQL crée automatiquement des sauvegardes de serveur et les conserve dans un stockage géoredondant ou redondant localement configuré par l’utilisateur. Les sauvegardes peuvent être utilisées pour restaurer votre serveur à un point dans le temps. La sauvegarde et la restauration sont une partie essentielle de toute stratégie de continuité d’activité, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles.

## <a name="backups"></a>Sauvegardes

Azure Database pour MySQL effectue des sauvegardes des fichiers de données et du journal des transactions. Celles-ci vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde configurée. La période de rétention de sauvegarde par défaut est de sept jours. Vous pouvez [éventuellement la configurer](howto-restore-server-portal.md#set-backup-configuration) sur 35 jours maximum. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Ces fichiers de sauvegarde ne sont pas exposés à l’utilisateur et ne peuvent pas être exportés. Ces sauvegardes sont utilisables uniquement pour les opérations de restauration dans Azure Database pour MySQL. Vous pouvez utiliser [mysqldump](concepts-migrate-dump-restore.md) pour copier une base de données.

Le type et la fréquence de sauvegarde dépendent du stockage back-end pour les serveurs.

### <a name="backup-type-and-frequency"></a>Type et fréquence de sauvegarde

#### <a name="basic-storage-servers"></a>Serveurs de stockage de base

Le stockage De base est le stockage back-end qui prend en charge les [serveurs de niveau De base](concepts-pricing-tiers.md). Les sauvegardes sur les serveurs de stockage de base sont basées sur des captures instantanées. Une capture instantanée de base de données complète est effectuée quotidiennement. Aucune sauvegarde différentielle n’est effectuée pour les serveurs de stockage de base, et toutes les sauvegardes de captures instantanées sont uniquement des sauvegardes complètes de base de données.

Les sauvegardes des journaux des transactions se produisent toutes les cinq minutes.

#### <a name="general-purpose-storage-v1-servers-supports-up-to-4-tb-storage"></a>Serveurs V1 de stockage à usage général (prend en charge jusqu’à 4 to de stockage)

Le stockage à usage général est le stockage back-end qui prend en charge les serveurs de niveau [Usage général](concepts-pricing-tiers.md) et [Mémoire optimisée](concepts-pricing-tiers.md). Pour les serveurs avec un stockage à usage général jusqu’à 4 To, les sauvegardes complètes se produisent une fois par semaine. Les sauvegardes différentielles se produisent deux fois par jour. Les sauvegardes des journaux des transactions se produisent toutes les cinq minutes. Les sauvegardes sur un stockage à usage général jusqu’à 4 To ne sont pas basées sur des captures instantanées et consomment de la bande passante en E/S. Pour les grandes bases de données (>1 To) sur un stockage de 4 To, nous vous recommandons d’envisager :

- Le provisionnement de plus d’IOPS pour prendre en compte les E/S des sauvegardes OU
- Une migration vers un stockage à usage général qui prend en charge jusqu’à 16 To de stockage si l’infrastructure de stockage sous-jacente est disponible dans vos [régions Azure](./concepts-pricing-tiers.md#storage) favorites. Il n’existe aucun coût supplémentaire pour le stockage à usage général qui prend en charge jusqu’à 16 To de stockage. Pour obtenir de l’aide sur la migration vers un stockage de 16 To, veuillez ouvrir un ticket de support à partir du portail Azure.

#### <a name="general-purpose-storage-v2-servers-supports-up-to-16-tb-storage"></a>Serveurs V2 de stockage à usage général (prend en charge jusqu’à 16 to de stockage)

Dans un sous-ensemble de [régions Azure](./concepts-pricing-tiers.md#storage), tous les serveurs nouvellement provisionnés peuvent prendre en charge jusqu’à 16 To de stockage à usage général. En d’autres termes, le stockage jusqu’à 16 To est le stockage à usage général par défaut pour toutes les [régions](concepts-pricing-tiers.md#storage) où il est pris en charge. Les sauvegardes sur ces serveurs de stockage de 16 To sont basées sur des captures instantanées. La première sauvegarde de captures instantanées est planifiée immédiatement après la création d’un serveur. Les sauvegardes de captures instantanées sont effectuées quotidiennement une fois. Les sauvegardes des journaux des transactions se produisent toutes les cinq minutes.

Pour plus d’informations sur le stockage à usage général et de base, consultez la [documentation relative au stockage](./concepts-pricing-tiers.md#storage).

### <a name="backup-retention"></a>Rétention des sauvegardes

Les sauvegardes sont conservées en fonction du paramètre de période de rétention de sauvegarde sur le serveur. Vous pouvez sélectionner une période de rétention comprise entre 7 et 35 jours. La période de conservation par défaut est 7 jours. Vous pouvez définir la période de rétention lors de la création du serveur ou ultérieurement en mettant à jour la configuration de la sauvegarde à l’aide du [portail Azure](./howto-restore-server-portal.md#set-backup-configuration) ou d’[Azure CLI](./howto-restore-server-cli.md#set-backup-configuration).

La période de rétention de sauvegarde détermine jusqu’à quelle date une restauration à un point dans le temps peut être récupérée, dans la mesure où elle est basée sur les sauvegardes disponibles. La période de rétention de sauvegarde peut également être traitée comme une fenêtre de récupération du point de vue de la restauration. Toutes les sauvegardes requises pour effectuer une restauration à un instant dans le passé au cours de la période de rétention de sauvegarde sont conservées dans le stockage de sauvegarde. Par exemple, si la période de conservation des sauvegardes est définie sur 7 jours, la fenêtre de récupération est considérée comme les 7 derniers jours. Dans ce scénario, toutes les sauvegardes nécessaires à la restauration du serveur au cours des 7 derniers jours sont conservées. Avec une fenêtre de rétention de sauvegarde de sept jours :

- Les serveurs v1 de stockage à usage général (qui prennent en charge jusqu’à 4 To de stockage) peuvent conserver jusqu’à 2 sauvegardes complètes de bases de données, toutes les sauvegardes différentielles et les sauvegardes de fichier journal effectuées depuis la première sauvegarde complète de la base de données.
- Les serveurs v2 de stockage à usage général (qui prennent en charge jusqu’à 16 To de stockage) peuvent conserver les captures instantanées complètes de bases de données et les sauvegardes de fichier journal au cours des 8 derniers jours.

#### <a name="long-term-retention"></a>Rétention à long terme

La conservation à long terme des sauvegardes au-delà de 35 jours n’est pas encore prise en charge de manière native par le service. Vous avez la possibilité d’utiliser mysqldump pour effectuer des sauvegardes et les stocker pour une conservation à long terme. Notre équipe de support technique a publié un [article pas à pas](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/automate-backups-of-your-azure-database-for-mysql-server-to/ba-p/1791157) pour vous permettre de savoir comment y parvenir.

### <a name="backup-redundancy-options"></a>Options de redondance de sauvegarde

Azure Database pour MySQL offre la possibilité de choisir entre le stockage de sauvegarde géoredondant ou redondant localement dans les niveaux Usage général et À mémoire optimisée. Lorsque les sauvegardes sont conservées dans le stockage de sauvegarde géoredondant, elles ne sont pas uniquement conservées dans la région d’hébergement de votre serveur, mais sont également répliquées dans un [centre de données jumelé](../best-practices-availability-paired-regions.md). Cette redondance géographique permet de bénéficier d’une meilleure protection et de restaurer votre serveur dans une région différente en cas de sinistre. Le niveau De base propose uniquement un stockage de sauvegarde redondant localement.

> [!NOTE]
>Pour les régions suivantes : Inde centrale, France Centre, Émirats arabes unis Nord, Afrique du Sud Nord ; Le stockage v2 à usage général est en préversion publique. Si vous créez un serveur source dans le stockage v2 à usage général (qui prennent en charge jusqu’à 16 to de stockage) dans les régions mentionnées ci-dessus, l’activation de la sauvegarde géo-redondante n’est pas prise en charge. 

#### <a name="moving-from-locally-redundant-to-geo-redundant-backup-storage"></a>Passage du stockage localement redondant au stockage géoredondant

La configuration du stockage géoredondant ou redondant localement pour la sauvegarde est uniquement possible lors de la création du serveur. Une fois que le serveur est approvisionné, vous ne pouvez pas modifier l’option de redondance du stockage de sauvegarde. Pour déplacer votre stockage de sauvegarde d’un stockage localement redondant vers un stockage géo-redondant, la création d’un nouveau serveur et la migration des données à l’aide du [vidage et de la restauration](concepts-migrate-dump-restore.md) est la seule option prise en charge.

### <a name="backup-storage-cost"></a>Coût du stockage de sauvegarde

Azure Database pour MySQL fournit jusqu’à 100 % du stockage de votre serveur approvisionné en stockage de sauvegarde sans coût supplémentaire. Tous les stockages de sauvegarde supplémentaires utilisés sont facturés en Go par mois. Par exemple, si vous avez configuré un serveur avec 250 Go de stockage, vous disposez de 250 Go de stockage supplémentaire pour les sauvegardes de serveur sans frais supplémentaires. Le stockage utilisé pour les sauvegardes de plus de 250 Go est facturé conformément au [modèle de tarification](https://azure.microsoft.com/pricing/details/mysql/).

Vous pouvez utiliser la métrique [Stockage de sauvegarde utilisé](concepts-monitoring.md) dans Azure Monitor, disponible via le portail Azure, pour superviser le stockage de sauvegarde consommé par un serveur. La métrique Stockage de sauvegarde utilisé représente le total du stockage consommé par l’ensemble des sauvegardes de base de données complètes, sauvegardes différentielles et sauvegardes de journaux conservées en fonction de la période de rétention de sauvegarde définie pour le serveur. La fréquence des sauvegardes est gérée par le service et expliquée plus haut. Une activité transactionnelle importante sur le serveur peut entraîner une augmentation de l’utilisation du stockage de sauvegarde, quelle que soit la taille totale de la base de données. Pour le stockage géo-redondant, l’utilisation du stockage de sauvegarde est le double de celle du stockage localement redondant.

Le principal moyen de contrôler le coût fu stockage de sauvegarde consiste à définir la période de rétention de sauvegarde appropriée et à choisir les options de redondance de sauvegarde appropriées pour atteindre les objectifs de récupération souhaités. Vous pouvez sélectionner une période de conservation comprise entre 7 et 35 jours. Les serveurs à usage général et à mémoire optimisée peuvent disposer d’un stockage géoredondant pour les sauvegardes.

## <a name="restore"></a>Restaurer

Dans Azure Database pour MySQL, l’exécution d’une restauration crée un serveur à partir de sauvegardes du serveur d’origine et restaure toutes les bases de données contenues dans le serveur.

Deux types de restauration sont disponibles :

- La **restauration à un point dans le temps** est disponible avec l’option de redondance de sauvegarde, et crée un serveur dans la même région que votre serveur d’origine en utilisant la combinaison de sauvegarde complète et de sauvegarde du journal des transactions.
- La **géorestauration** est disponible uniquement si vous avez configuré votre serveur pour le stockage géoredondant. Elle vous permet de restaurer votre serveur dans une autre région en utilisant la sauvegarde la plus récente.

La durée estimée pour la récupération du serveur dépend de plusieurs facteurs :
* la taille des bases de données ;
* le nombre de journaux d’activité de transactions impliqués ;
* la quantité d’activité devant être relue pour effectuer une récupération au point de restauration ;
* la bande passante du réseau, si la restauration s’effectue dans une autre région ;
* le nombre de demandes de restauration simultanées en cours de traitement dans la région cible.
* la présence d’une clé primaire dans les tables de la base de données. Pour accélérer la récupération, envisagez d’ajouter une clé primaire pour toutes les tables de votre base de données. Pour vérifier si vos tables ont une clé primaire, vous pouvez utiliser la requête suivante :
```sql
select tab.table_schema as database_name, tab.table_name from information_schema.tables tab left join information_schema.table_constraints tco on tab.table_schema = tco.table_schema and tab.table_name = tco.table_name and tco.constraint_type = 'PRIMARY KEY' where tco.constraint_type is null and tab.table_schema not in('mysql', 'information_schema', 'performance_schema', 'sys') and tab.table_type = 'BASE TABLE' order by tab.table_schema, tab.table_name;
```
Pour une base de données volumineuse ou très active, la restauration peut prendre plusieurs heures. En cas de panne prolongée dans une région, il est possible qu’un grand nombre de requêtes de géo-restauration soient lancées pour la récupération d’urgence. S’il y a un grand nombre de requêtes, le temps de récupération des bases de données individuelles peut s’en trouver augmenté. La plupart des restaurations de bases de données s’effectuent en moins de 12 heures.

> [!IMPORTANT]
> Une fois supprimés, les serveurs ne peuvent être restaurés que dans un délai de **cinq jours**, au terme duquel les sauvegardes sont supprimées. La sauvegarde de base de données est accessible et peut être restaurée uniquement à partir de l’abonnement Azure qui héberge le serveur. Pour restaurer un serveur supprimé, reportez-vous aux [étapes documentées](howto-restore-dropped-server.md). À l'issue du déploiement, pour protéger les ressources du serveur d'une suppression accidentelle ou de changements inattendus, les administrateurs peuvent utiliser des [verrous de gestion](../azure-resource-manager/management/lock-resources.md).

### <a name="point-in-time-restore"></a>Restauration dans le temps

Quelle que soit l’option de redondance de sauvegarde, vous pouvez effectuer une restauration à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde. Un serveur est créé dans la même région Azure que le serveur d’origine. Il est créé avec la configuration du serveur d’origine pour le niveau de tarification, la génération de calcul, le nombre de vCores, la taille de stockage, la période de rétention de sauvegarde et l’option de redondance de sauvegarde.

> [!NOTE]
> Deux paramètres de serveur sont réinitialisés aux valeurs par défaut (et ne sont pas copiés à partir du serveur principal) après l’opération de restauration
>
> - time_zone : valeur à affecter à la valeur par défaut **SYSTEM**
> - event_scheduler : valeur définie sur **OFF** sur le serveur restauré
>
> Vous devrez définir ces paramètres de serveur en reconfigurant le [paramètre de serveur](howto-server-parameters.md)

La restauration à un point dans le temps est utile dans plusieurs scénarios. Par exemple, lorsqu’un utilisateur supprime accidentellement des données, perd une base de données ou une table importante ou si une application remplace accidentellement des données correctes par des données erronées en raison d’un défaut de l’application.

Vous devez peut-être attendre la prochaine sauvegarde du journal des transactions avant de pouvoir restaurer à un point dans le temps dans les cinq dernières minutes.

### <a name="geo-restore"></a>La géorestauration

Vous pouvez restaurer un serveur dans une autre région Azure où le service est disponible si vous avez configuré votre serveur pour les sauvegardes géoredondantes. 
- Les serveurs v1 de stockage à usage général (qui prennent en charge un stockage jusqu’à 4 to) peuvent être restaurés dans la région géo-couplée ou dans n’importe quelle région Azure qui prend en charge le service de serveur unique Azure Database pour MySQL.
- Les serveurs v2 de stockage à usage général (qui prennent en charge un stockage jusqu’à 16 to) peuvent uniquement être restaurés dans des régions Azure qui prennent en charge l’infrastructure de serveurs v2 de stockage à usage général. Passez en revue [Niveaux tarifaires d’Azure Database pour MySQL](./concepts-pricing-tiers.md#storage) pour obtenir la liste des régions prises en charge.

La géorestauration constitue l’option de récupération par défaut lorsque votre serveur est indisponible en raison d’un incident dans la région où il est hébergé. Si un incident à grande échelle dans une région entraîne l’indisponibilité de votre application de base de données, vous pouvez restaurer un serveur à partir des sauvegardes géoredondantes sur un serveur situé dans n’importe quelle autre région. La géorestauration utilise la sauvegarde la plus récente du serveur. Il peut y avoir un délai entre le moment où une sauvegarde est effectuée et celui où elle est répliquée dans une autre région. Ce délai peut atteindre une heure. En cas d’incident, il peut donc y avoir jusqu’à une heure de pertes de données.

> [!IMPORTANT]
>Si une géorestauration est effectuée pour un serveur nouvellement créé, la synchronisation de la sauvegarde initiale peut prendre plus de 24 heures en fonction de la taille des données, car la durée de la sauvegarde initiale d’un instantané complet est bien supérieure. Les sauvegardes d’instantanés ultérieures sont des copies incrémentielles. Par conséquent, les restaurations sont plus rapides 24 heures après la création du serveur. Si vous évaluez des géorestaurations pour définir votre RTO, nous vous recommandons d’attendre et de les évaluer **uniquement 24 heures après** la création du serveur pour obtenir de meilleures estimations.

Pendant la géorestauration, les configurations de serveur qui peuvent être changées incluent la génération de calcul, les vCores, la période de conservation des sauvegardes et les options de redondance de sauvegarde. Le changement de niveau tarifaire (De base, Usage général ou Mémoire optimisée) ou de la taille du stockage pendant la géorestauration n’est pas pris en charge.

Le délai estimé de récupération dépend de plusieurs facteurs, notamment du nombre total de bases de données à récupérer dans la même région au même moment, de la taille des bases de données, de la taille du journal des transactions et de la bande passante réseau. Le délai de récupération est généralement inférieur à 12 heures.

### <a name="perform-post-restore-tasks"></a>Effectuer des tâches de post-restauration

Après une restauration à l’aide d’un de ces mécanismes de récupération, vous devez effectuer les tâches suivantes afin que les utilisateurs et les applications soient de nouveau opérationnels :

- Si le nouveau serveur est destiné à remplacer le serveur d’origine, redirigez les clients et les applications clientes vers le nouveau serveur
- Vérifiez que les règles de réseau virtuel appropriées sont en place pour permettre aux utilisateurs de se connecter. Ces règles ne sont pas copiées à partir du serveur d’origine.
- Assurez-vous que les connexions et les autorisations appropriées au niveau de la base de données sont en place
- Configurer les alertes, selon les besoins

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la continuité d’activité, consultez la  [vue d’ensemble de la continuité d’activité](concepts-business-continuity.md).
- Pour effectuer une restauration à un point dans le temps à l’aide du portail Azure, consultez  [Restaurer un serveur à un point dans le temps à l’aide du portail Azure](howto-restore-server-portal.md).
- Pour effectuer une restauration à un point dans le temps à l’aide d’Azure CLI, consultez  [Restaurer un serveur à un point dans le temps à l’aide d’Azure CLI](howto-restore-server-cli.md).
