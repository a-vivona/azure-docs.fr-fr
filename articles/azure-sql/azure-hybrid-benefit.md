---
title: Azure Hybrid Benefit
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Utilisez des licences SQL Server existantes pour recevoir des remises sur Azure SQL Database et SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service-overview
ms.custom: sqldbrb=4
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: sashan, moslake
ms.date: 02/16/2021
ms.openlocfilehash: d8557c24b1f6ef46559f57d09c6e273dd4041272
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/11/2021
ms.locfileid: "112018168"
---
# <a name="azure-hybrid-benefit---azure-sql-database--sql-managed-instance"></a>Azure Hybrid Benefit – Azure SQL Database et SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Dans le niveau de calcul approvisionné du modèle d’achat vCore, vous pouvez échanger vos licences existantes avec des tarifs réduits sur Azure SQL Database et Azure SQL Managed Instance en utilisant [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/). Cet avantage Azure vous permet d’économiser jusqu’à 30 % même plus sur SQL Database et SQL Managed Instance en utilisant vos licences SQL Server avec Software Assurance. La page [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) comporte une calculatrice pour vous aider à déterminer les économies réalisées.  Notez qu’Azure Hybrid Benefit ne s’applique pas à Azure SQL Database serverless.

> [!NOTE]
> Le passage à Azure Hybrid Benefit n’implique aucun temps d’arrêt.

![Structure tarifaire vCore](./media/azure-hybrid-benefit/pricing.png)

## <a name="choose-a-license-model"></a>Choisir un modèle de licence

Avec Azure Hybrid Benefit, vous pouvez choisir de payer uniquement pour l’infrastructure Azure sous-jacente en utilisant votre licence SQL Server existante pour le moteur de base de données SQL Server lui-même (tarif de calcul de base) ou de payer à la fois pour l’infrastructure sous-jacente et pour la licence SQL Server (tarif avec licence comprise).

Vous pouvez choisir ou changer votre modèle de licence dans le portail Azure : 
- Pour les nouvelles bases de données, sélectionnez **Configurer la base de données** dans l’onglet **Informations de base** lors de la création et sélectionnez l’option **Réaliser des économies**.
- Pour les bases de données existantes, sélectionnez **Calcul + stockage** dans le menu **Paramètres** et sélectionnez l’option **Réaliser des économies**.

Vous pouvez également configurer une base de données nouvelle ou existante à l’aide de l’une des API suivantes :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour définir ou mettre à jour le type de licence à l’aide de PowerShell :

- [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)
- [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)
- [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance)
- [Set-AzSqlInstance](/powershell/module/az.sql/set-azsqlinstance)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour définir ou mettre à jour le type de licence à l’aide d’Azure CLI :

- [az sql db create](/cli/azure/sql/db#az_sql_db_create)
- [az sql mi create](/cli/azure/sql/mi#az_sql_mi_create)
- [az sql mi update](/cli/azure/sql/mi#az_sql_mi_update)

# <a name="rest-api"></a>[REST API](#tab/rest)

Pour définir ou mettre à jour le type de licence à l’aide de l’API REST :

- [Bases de données : Create ou Update](/rest/api/sql/databases/createorupdate)
- [Bases de données - Mettre à jour](/rest/api/sql/databases/update)
- [Instances managées - Créer ou Mettre à jour](/rest/api/sql/managedinstances/createorupdate)
- [Instances managées - Mettre à jour](/rest/api/sql/managedinstances/update)

* * *


### <a name="azure-hybrid-benefit-questions"></a>Questions sur Azure Hybrid Benefit

#### <a name="are-there-dual-use-rights-with-azure-hybrid-benefit-for-sql-server"></a>Y a-t-il des droits d’utilisation double avec Azure Hybrid Benefit pour SQL Server ?

Vous disposez de 180 jours de droits d’utilisation double de la licence pour vous assurer que les migrations s’exécutent en toute transparence. À l’issue de cette période de 180 jours, vous ne pouvez utiliser la licence SQL Server que dans le cloud dans SQL Database. Vous n’avez plus de droits d’utilisation double localement et dans le cloud.

#### <a name="how-does-azure-hybrid-benefit-for-sql-server-differ-from-license-mobility"></a>Quelle est la différence entre Azure Hybrid Benefit pour SQL Server et la mobilité de licence ?

Nous proposons aux clients SQL Server des avantages de mobilité de licence avec Software Assurance. Cela permet de réattribuer leurs licences à des serveurs partagés d’un partenaire. Vous pouvez utiliser cet avantage sur Azure IaaS et AWS EC2.

La différence entre Azure Hybrid Benefit pour SQL Server et la mobilité de licence se trouve dans deux domaines clés :

- Azure Hybrid Benefit pour SQL Server propose des avantages économiques pour la migration de charges de travail hautement virtualisées vers Azure. Les clients SQL Server Enterprise Edition peuvent obtenir quatre cœurs dans Azure dans la référence (SKU) Usage général pour chaque cœur qu’ils possèdent localement pour des applications hautement virtualisées. La mobilité de licence n’offre aucun avantage spécifique en termes de coûts pour la migration des charges de travail virtualisées vers le cloud.
- Elle fournit une destination PaaS sur Azure (SQL Managed Instance) qui est hautement compatible avec SQL Server.

#### <a name="what-are-the-specific-rights-of-the-azure-hybrid-benefit-for-sql-server"></a>Quels sont les droits spécifiques associés à Azure Hybrid Benefit pour SQL Server ?

Les clients SQL Database et SQL Managed Instance ont les droits suivants associés à Azure Hybrid Benefit pour SQL Server :

|Empreinte de licence|Que vous propose Azure Hybrid Benefit pour SQL Server ?|
|---|---|
|Clients avec cœurs SQL Server Enterprise Edition et SA|<li>Possibilité de payer un taux de base pour les références SKU Hyperscale, Usage général et Critique pour l’entreprise</li><br><li>1 cœur local = 4 cœurs dans la référence SKU Hyperscale</li><br><li>1 cœur local = 4 cœurs dans la référence SKU Usage général</li><br><li>1 cœur local = 1 cœur dans la référence SKU Critique pour l’entreprise</li>|
|Clients avec cœurs SQL Server Standard Edition et SA|<li>Possibilité de payer un taux de base pour les références SKU Hyperscale, Usage général et Critique pour l’entreprise</li><br><li>1 cœur local = 1 cœur dans la référence SKU Hyperscale</li><br><li>1 cœur local = 1 cœur dans la référence SKU Usage général</li><br><li>4 cœurs locaux = 1 cœur dans la référence SKU Critique pour l’entreprise</li>|
|||


## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment choisir une option de déploiement Azure SQL, consultez [Choisir l'option de déploiement appropriée dans Azure SQL](azure-sql-iaas-vs-paas-what-is-overview.md).
- Pour comparer les fonctionnalités de SQL Database et de SQL Managed Instance, consultez [Fonctionnalités SQL Database et SQL Managed Instance](database/features-comparison.md).
