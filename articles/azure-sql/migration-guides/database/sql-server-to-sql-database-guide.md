---
title: 'De SQL Server vers Azure SQL Database : Guide de migration'
description: Suivez ce guide pour migrer vos bases de données SQL Server vers Azure SQL Database.
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: cawrites
ms.date: 03/19/2021
ms.openlocfilehash: 8e044cee9e91c0de16105bc4c954af9e61acea5a
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122524951"
---
# <a name="migration-guide-sql-server-to-azure-sql-database"></a>Guide de migration : de SQL Server vers SQL Database
[!INCLUDE[appliesto--sqldb](../../includes/appliesto-sqldb.md)]

Dans ce guide, vous allez apprendre [à migrer](https://azure.microsoft.com/migration/migration-journey) votre instance SQL Server vers Azure SQL Database. 

Vous pouvez migrer SQL Server s’exécutant en local ou sur : 

- SQL Server sur les machines virtuelles  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform – GCP)  
- Cloud SQL pour SQL Server (Google Cloud Platform - GCP) 

Pour plus d’informations sur la migration, consultez la [vue d’ensemble de la migration](sql-server-to-sql-database-overview.md). Pour obtenir d’autres guides de migration, consultez [Migration de base de données](/data-migration). 

:::image type="content" source="media/sql-server-to-database-overview/migration-process-flow-small.png" alt-text="Flux du processus de migration":::

## <a name="prerequisites"></a>Prérequis 

Pour la [migration de votre serveur SQL Server](https://azure.microsoft.com/migration/sql-server/) vers Azure SQL Database, veillez à avoir : 

- Choisi une [méthode de migration](sql-server-to-sql-database-overview.md#compare-migration-options) et des outils correspondants.
- Installé l’[Assistant Migration de données (DMA)](https://www.microsoft.com/download/details.aspx?id=53595) sur une machine qui peut se connecter à votre serveur SQL Server source.
- Créé une instance [Azure SQL Database](../../database/single-database-create-quickstart.md) cible. 
- Configuré la connectivité et les autorisations nécessaires pour accéder à la fois à la source et à la cible. 
- Examiné les fonctionnalités du moteur de base de données [disponibles dans Azure SQL Database](../../database/features-comparison.md). 



## <a name="pre-migration"></a>Prémigration

Une fois que vous avez vérifié que votre environnement source est pris en charge, commencez par la phase de prémigration. Découvrez toutes les sources de données existantes, évaluez la faisabilité de la migration et identifiez les problèmes bloquants susceptibles d’empêcher la [migration vers le cloud Azure](https://azure.microsoft.com/migration) de s’effectuer correctement.

### <a name="discover"></a>Découvrez

Durant la phase de découverte, analysez le réseau pour identifier toutes les instances et fonctionnalités de SQL Server utilisées par votre organisation. 

Utilisez [Azure Migrate](../../../migrate/migrate-services-overview.md) pour évaluer la pertinence de la migration des serveurs locaux, effectuer un dimensionnement en fonction des performances, et fournir des estimations de coût pour une exécution dans Azure. 

Vous pouvez également utiliser  [Microsoft Assessment and Planning Toolkit (« MAP Toolkit »)](https://www.microsoft.com/download/details.aspx?id=7826) pour évaluer votre infrastructure informatique actuelle. La boîte à outils fournit un puissant outil d’inventaire, d’évaluation et de création de rapports, qui permet de simplifier le processus de planification de la migration. 

Pour plus d’informations sur les outils utilisables au cours de la phase de découverte, consultez [Services et outils disponibles pour les scénarios de migration de données](../../../dms/dms-tools-matrix.md). 

### <a name="assess"></a>Évaluer 

[!INCLUDE [assess-estate-with-azure-migrate](../../../../includes/azure-migrate-to-assess-sql-data-estate.md)]

Une fois les sources de données découvertes, évaluez les bases de données SQL Server locales que vous pouvez migrer vers Azure SQL Database pour identifier les obstacles ou les problèmes de compatibilité liés à la migration. 

Vous pouvez utiliser l’Assistant Migration de données (version 4.1 et ultérieure) pour évaluer les bases de données et obtenir les informations suivantes : 

- [Recommandations relatives aux cibles Azure](/sql/dma/dma-assess-sql-data-estate-to-sqldb)
- [Recommandations relatives aux références SKU Azure](/sql/dma/dma-sku-recommend-sql-db)

Pour évaluer votre environnement à l’aide de l’Assistant Migration de données, suivez les étapes ci-dessous : 

1. Ouvrez l’outil [DMA (Assistant Migration de données)](https://www.microsoft.com/download/details.aspx?id=53595). 
1. Sélectionnez **Fichier**, puis choisissez **Nouvelle évaluation**. 
1. Spécifiez un nom de projet, sélectionnez le type de serveur source SQL Server, et le type de serveur cible Azure SQL Database. 
1. Sélectionnez les types de rapport d’évaluation à générer. Par exemple, la compatibilité de base de données et la parité des fonctionnalités. Selon le type d’évaluation, les autorisations nécessaires sur le serveur SQL Server source peuvent être différentes.  L’outil DMA met en évidence les autorisations nécessaires d’après l’outil Advisor choisi avant d’exécuter l’évaluation.
    - La catégorie **parité des fonctionnalités** fournit un ensemble complet de recommandations, d’alternatives disponibles dans Azure et d’étapes d’atténuation, qui vous aident à planifier votre projet de migration. (autorisations d’administrateur système nécessaires)
    - La catégorie **problèmes de compatibilité** identifie les problèmes de compatibilité des fonctionnalités partiellement ou non prises en charge, qui peuvent bloquer la migration. Elle fournit également les recommandations à suivre pour les résoudre (autorisations `CONNECT SQL`, `VIEW SERVER STATE` et `VIEW ANY DEFINITION` nécessaires).
1. Spécifiez les détails de la connexion source de votre serveur SQL Server, puis connectez-vous à la base de données source.
1. Sélectionnez **Démarrer l’évaluation**. 
1. À la fin du processus, sélectionnez et passez en revue les rapports d’évaluation pour identifier les problèmes bloquants et les problèmes de parité des fonctionnalités relatifs à la migration. Vous pouvez également exporter le rapport d’évaluation vers un fichier partagé avec d’autres équipes ou membres du personnel de votre organisation. 
1. Déterminez le niveau de compatibilité de la base de données pour réduire les efforts postmigration.  
1. Identifiez la meilleure référence SKU d’Azure SQL Database pour votre charge de travail locale. 

Pour en savoir plus, consultez [Effectuer une évaluation de migration SQL Server avec l’Assistant Migration de données](/sql/dma/dma-assesssqlonprem).

Si l’évaluation révèle plusieurs obstacles qui indiquent que votre base de données n’est pas prête pour une migration vers Azure SQL Database, d’autres options existent :

- [Azure SQL Managed Instance](../managed-instance/sql-server-to-managed-instance-overview.md), s’il existe plusieurs dépendances dont l’étendue se limite à l’instance
- [SQL Server sur les machines virtuelles Azure](../virtual-machines/sql-server-to-sql-on-azure-vm-migration-overview.md), si SQL Database et SQL Managed Instance ne sont pas des cibles appropriées. 



#### <a name="scaled-assessments-and-analysis"></a>Évaluations mises à l’échelle et analyse
L’Assistant Migration de données prend en charge l’exécution d’évaluations mises à l’échelle et la centralisation des rapports d’évaluation à des fins d’analyse. 

Si vous disposez de plusieurs serveurs et bases de données qui doivent être évalués et analysés à grande échelle afin de fournir une vue plus large du parc de données, consultez les liens suivants pour en savoir plus :

- [Effectuer des évaluations mises à l’échelle à l’aide de PowerShell](/sql/dma/dma-consolidatereports)
- [Analyse des rapports d’évaluation à l’aide de Power BI](/sql/dma/dma-consolidatereports#dma-reports)

> [!IMPORTANT]
> L’exécution d’évaluations à grande échelle sur plusieurs bases de données, en particulier les plus volumineuses, peut également être automatisée à l’aide de l’[utilitaire en ligne de commande de DMA](/sql/dma/dma-commandline). Vous pouvez ensuite charger ces évaluations vers [Azure Migrate](/sql/dma/dma-assess-sql-data-estate-to-sqldb#view-target-readiness-assessment-results) pour approfondir l’analyse et préparer la cible.

## <a name="migrate"></a>Migrer

Une fois que vous avez effectué les tâches associées à la phase de prémigration, vous êtes prêt à effectuer la migration du schéma et des données. 

Migrez les données à l’aide de la [méthode de migration](sql-server-to-sql-database-overview.md#compare-migration-options) de votre choix. 

Ce guide décrit les deux options les plus répandues : l’Assistant Migration de données et Azure Database Migration Service. 

### <a name="data-migration-assistant-dma"></a>Assistant Migration de données (DMA)

Pour migrer une base de données de SQL Server vers Azure SQL Database à l’aide de l’outil DMA, suivez les étapes ci-dessous : 

1. Téléchargez et installez l’[Assistant Migration de données](https://www.microsoft.com/download/details.aspx?id=53595).
1. Créez un projet et sélectionnez le type de projet **Migration**.
1. Définissez le type de serveur source **SQL Server** et le type de serveur cible **Azure SQL Database**, sélectionnez l’étendue de migration **Schéma et données**, puis sélectionnez **Créer**.
1. Dans le projet de migration, spécifiez les détails du serveur source, notamment le nom du serveur, les informations d’identification permettant de se connecter au serveur ainsi que la base de données source à migrer.
1. Dans les détails du serveur cible, spécifiez le nom du serveur Azure SQL Database, les informations d’identification permettant de se connecter au serveur ainsi que la base de données cible vers laquelle effectuer la migration.
1. Sélectionnez les objets de schéma, puis déployez-les sur le serveur Azure SQL Database cible.
1. Enfin, sélectionnez **Démarrer la migration des données** et supervisez la progression de la migration.

Pour obtenir un tutoriel détaillé, consultez [Migrer une instance de SQL Server locale ou SQL Server sur les machines virtuelles Azure vers Azure SQL Database à l’aide de l’Assistant Migration de données](/sql/dma/dma-migrateonpremsqltosqldb). 


> [!NOTE]
> - Mettez à l’échelle votre base de données à un niveau de service et une taille de calcul supérieurs pendant le processus d’importation pour optimiser la vitesse d’importation en fournissant davantage de ressources. Vous pouvez ensuite effectuer un scale-down après l’importation.</br>
> - Le niveau de compatibilité de la base de données importée est basé sur le niveau de compatibilité de votre base de données source. 


### <a name="azure-database-migration-service-dms"></a>Azure Database Migration Service (DMS)

Pour migrer des bases de données de SQL Server vers Azure SQL Database à l’aide du service DMS, suivez les étapes ci-dessous :

1. Si vous ne l’avez pas déjà fait, inscrivez le fournisseur de ressources **Microsoft.DataMigration** dans votre abonnement. 
1. Créez une instance d’Azure Database Migration Service à l’emplacement de votre choix (de préférence dans la même région que votre instance cible d’Azure SQL Database). Sélectionnez un réseau virtuel existant, ou créez-en un autre pour héberger votre instance de DMS.
1. Une fois votre instance de DMS créée, créez un projet de migration, puis spécifiez le type de serveur source **SQL Server** et le type de serveur cible **Azure SQL Database**. Choisissez le type d’activité **Migration de données hors connexion** dans le panneau de création de projet de migration.
1. Spécifiez les détails du serveur SQL Server source dans la page **Détails de la source de migration** ainsi que les détails de la cible Azure SQL Database dans la page **Détails de la cible de migration**.
1. Mappez les bases de données source et cible pour la migration, puis sélectionnez les tables à migrer.
1. Passez en revue le récapitulatif de la migration, puis sélectionnez **Exécuter la migration**. Vous pouvez ensuite superviser l’activité de migration et vérifier la progression de la migration de votre base de données.

Pour accéder à un tutoriel détaillé, consultez [Migrer SQL Server vers Azure SQL Database à l’aide de DMS](../../../dms/tutorial-sql-server-to-azure-sql.md). 

## <a name="data-sync-and-cutover"></a>Synchronisation des données et basculement

Quand vous utilisez des options de migration qui répliquent/synchronisent en permanence les changements apportés aux données de la source vers la cible, les données et le schéma sources peuvent varier et dériver par rapport à la cible. Pendant la synchronisation des données, vérifiez que tous les changements apportés à la source sont capturés et appliqués à la cible au cours du processus de migration. 

Une fois que vous avez vérifié que les données sont identiques sur la source et la cible, vous pouvez effectuer le basculement de l’environnement source vers l’environnement cible. Il est important de planifier le processus de basculement avec les équipes métier/applicatives. Ainsi, vous avez la garantie que l’interruption minimale qui a lieu pendant le basculement n’affecte pas la continuité de l’activité. 

> [!IMPORTANT]
> Pour plus d’informations sur les étapes spécifiques associées à l’exécution d’un basculement dans le cadre de migrations à l’aide du service DMS, consultez [Exécution du basculement de migration](../../../dms/tutorial-sql-server-to-azure-sql.md).

## <a name="migration-recommendations"></a>Recommandations en matière de migration

Pour accélérer la migration vers Azure SQL Database, vous devez prendre en compte les recommandations suivantes :

|  | Contention de ressources | Recommandation |
|--|--|--|
| **Source (généralement en local)** |Le goulot d’étranglement principal au cours de la migration dans la source est dû aux E/S de données et à la latence sur le fichier de données qui doit être analysé avec précaution.  |En fonction des E/S de données et de la latence du fichier de données et selon qu’il s’agit d’une machine virtuelle ou d’un serveur physique, vous devez faire appel à l’administrateur du stockage et explorer les options permettant d’atténuer le goulot d’étranglement. |
|**Cible (Azure SQL Database)**|Le plus grand facteur limitant est le taux de génération des journaux et la latence du fichier journal. Avec Azure SQL Database, vous pouvez obtenir un taux de génération de journal maximal de 96 Mo/s. | Pour accélérer la migration, effectuez un scale-up de la base de données SQL cible vers le niveau « Critique pour l’entreprise Gen5 8 vCores » pour obtenir le taux maximal de génération de journaux de 96 Mo/s et obtenir une faible latence pour le fichier journal. Le niveau de service [Hyperscale](../../database/service-tier-hyperscale.md) fournit un taux de journalisation de 100 Mo/s, quel que soit le niveau de service choisi. |
|**Réseau** |La bande passante réseau nécessaire est égale au taux maximal d’ingestion des journaux de 96 Mo/s (768 Mo/s). |En fonction de la connectivité réseau entre votre centre de données local et Azure, vérifiez que la bande passante réseau (en général [Azure ExpressRoute](../../../expressroute/expressroute-introduction.md#bandwidth-options)) prend en charge le taux maximal d’ingestion des journaux. |
|**Machine virtuelle utilisée pour l’Assistant Migration de données (DMA)** |Le processeur est le principal goulot d’étranglement de la machine virtuelle exécutant l’Assistant Migration de données. |Éléments à prendre en compte pour accélérer la migration des données : </br>- Utiliser des machines virtuelles Azure nécessitant beaucoup de ressources système </br>- Utiliser au moins une machine virtuelle F8s_v2 (8 vCores) pour l’exécution de l’Assistant Migration de données </br>- S’assurer que la machine virtuelle s’exécute dans la même région Azure que la cible |
|**Azure Database Migration Service (DMS)** |Considérations relatives à la contention des ressources de calcul et aux objets de base de données pour DMS |Utilisez le niveau Premium 4 vCores. DMS s’occupe automatiquement des objets de base de données tels que les clés étrangères, les déclencheurs, les contraintes et les index non-cluster, et n’a besoin d’aucune intervention manuelle.  |


## <a name="post-migration"></a>Postmigration

Une fois que vous avez réussi la phase de migration, vous devez effectuer toute une série de tâches postmigration pour vérifier que tout fonctionne de manière fluide et efficace. 

La phase postmigration est cruciale pour résoudre les problèmes de justesse et d’exhaustivité des données ainsi que pour gérer les problèmes de performances liés à la charge de travail. 

### <a name="remediate-applications"></a>Corriger les applications 

Une fois les données migrées vers l’environnement cible, toutes les applications qui consommaient la source doivent commencer à consommer la cible. Dans certains cas, cela vous oblige à apporter des changements aux applications.

### <a name="perform-tests"></a>Effectuer des tests

L’approche de test pour la migration de base de données comprend les activités suivantes :

1. **Développer des tests de validation** : pour tester la migration d’une base de données, vous devez utiliser des requêtes SQL. Vous devez créer les requêtes de validation à exécuter sur les bases de données source et cible. Vos requêtes de validation doivent couvrir l’étendue que vous avez définie.
1. **Configurer un environnement de test**: l’environnement de test doit contenir une copie de la base de données source et de la base de données cible. Veillez à isoler l’environnement de test.
1. **Exécuter des tests de validation** : exécutez les tests de validation sur la source et sur la cible, puis analysez les résultats.
1. **Exécuter des tests de performances**: exécutez un test de performances sur la source et sur la cible, puis analysez et comparez les résultats.


## <a name="leverage-advanced-features"></a>Tirer parti des fonctionnalités avancées 

Veillez à tirer parti des fonctionnalités cloud avancées offertes par SQL Database, notamment la [haute disponibilité intégrée](../../database/high-availability-sla.md), la [détection des menaces](../../database/azure-defender-for-sql.md) ainsi que la [supervision et le paramétrage de votre charge de travail](../../database/monitor-tune-overview.md). 

Certaines fonctionnalités SQL Server sont disponibles uniquement une fois que vous avez fait passer le [niveau de compatibilité de la base de données](/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) au dernier niveau (150). 

Pour en savoir plus, consultez [Gestion d’Azure SQL Database après la migration](../../database/manage-data-after-migrating-to-database.md)


## <a name="next-steps"></a>Étapes suivantes

- Si vous souhaitez obtenir une matrice des services et outils Microsoft et tiers qui peuvent vous aider dans différents scénarios de migration de données et de base de données ainsi que pour des tâches spécialisées, consultez [Services et outils disponibles pour les scénarios de migration de données](../../../dms/dms-tools-matrix.md).


- Pour en apprendre davantage sur [Azure Migrate](https://azure.microsoft.com/services/azure-migrate), consultez
   - [Azure Migrate](../../../migrate/migrate-services-overview.md)

- Pour en savoir plus sur SQL Database, consultez :
    - [Vue d’ensemble d’Azure SQL Database](../../database/sql-database-paas-overview.md)
   - [Outil de calcul du coût total de possession Azure](https://azure.microsoft.com/pricing/tco/calculator/) 


- Pour plus d’informations sur l’infrastructure et le cycle d’adoption pour les migrations cloud, consultez :
   -  [Cloud Adoption Framework pour Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Meilleures pratiques pour l’évaluation des coûts et le dimensionnement des charges de travail migrées vers Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 
   -  [Ressources de migration vers le cloud](https://azure.microsoft.com/migration/resources)

- Pour évaluer la couche d’accès aux applications, consultez [Data Access Migration Toolkit (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Pour savoir comment exécuter un test A/B sur la couche d’accès aux données, consultez [Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview).