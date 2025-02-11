---
title: Intégrer Key Vault à SQL Server sur des machines virtuelles Windows dans Azure Resource Manager | Microsoft Docs
description: Apprenez à automatiser la configuration du chiffrement de SQL Server pour une utilisation avec Azure Key Vault. Cette rubrique explique comment utiliser l’intégration d’Azure Key Vault avec des machines virtuelles SQL créées à l’aide de Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: cd66dfb1-0e9b-4fb0-a471-9deaf4ab4ab8
ms.service: virtual-machines-sql
ms.subservice: security
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/30/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 76363f171ba08c28e7460a988ff72afbf546716b
ms.sourcegitcommit: beff1803eeb28b60482560eee8967122653bc19c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2021
ms.locfileid: "113434847"
---
# <a name="configure-azure-key-vault-integration-for-sql-server-on-azure-vms-resource-manager"></a>Configurer l’intégration d’Azure Key Vault pour SQL Server sur des machines virtuelles (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Il existe plusieurs fonctionnalités de chiffrement SQL Server, telles que le [chiffrement transparent des données (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption), le [chiffrement au niveau des colonnes (CLE)](/sql/t-sql/functions/cryptographic-functions-transact-sql) et le [chiffrement de sauvegarde](/sql/relational-databases/backup-restore/backup-encryption). Ces types de chiffrement nécessitent que vous gériez et stockiez les clés de chiffrement que vous utilisez pour le chiffrement. Le service Azure Key Vault est conçu pour optimiser la sécurité et la gestion de ces clés dans un emplacement sécurisé et hautement disponible. Le [connecteur SQL Server](https://www.microsoft.com/download/details.aspx?id=45344) permet à SQL Server d’utiliser ces clés depuis le coffre de clés Azure.

Si vous exécutez SQL Server en local, vous devez suivre la [procédure d’accès à Azure Key Vault à partir de votre instance SQL Server locale](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server). Mais, pour SQL Server sur des machines virtuelles Azure, vous pouvez gagner du temps à l’aide de la fonctionnalité *Intégration d’Azure Key Vault*.

Lorsque cette fonctionnalité est activée, elle installe automatiquement le connecteur SQL Server, configure le fournisseur EKM pour accéder à Azure Key Vault et crée les informations d'identification vous permettant d'accéder à votre coffre. Si vous avez examiné les étapes décrites dans la documentation locale mentionnée précédemment, vous pouvez voir que cette fonctionnalité automatise les étapes 2 et 3. La seule étape que vous devez encore exécuter manuellement consiste à créer le coffre de clé et des clés. À partir de là, la configuration complète de votre machine virtuelle SQL Server est automatisée. Une fois que la fonctionnalité a terminé cette configuration, vous pouvez exécuter des instructions Transact-SQL (T-SQL) pour commencer à chiffrer vos bases de données et vos sauvegardes comme vous le feriez normalement.

[!INCLUDE [Prepare for Key Vault integration](../../../../includes/virtual-machines-sql-server-akv-prepare.md)]

  >[!NOTE]
  > La version 1.0.4.0 du fournisseur EKM (Extensible Key Management) est installée sur la machine virtuelle SQL Server via l’[extension IaaS (infrastructure as a service) SQL](./sql-server-iaas-agent-extension-automate-management.md). La mise à niveau de l’extension IaaS SQL ne met pas à jour la version du fournisseur. Si nécessaire, mettez manuellement à niveau la version du fournisseur EKM (par exemple, lors de la migration vers SQL Managed Instance).


## <a name="enabling-and-configuring-key-vault-integration"></a>Activation et configuration de l’intégration de Key Vault
Vous pouvez activer l’intégration de Key Vault lors de l’approvisionnement ou vous pouvez la configurer pour les machines virtuelles existantes.

### <a name="new-vms"></a>Nouvelles machines virtuelles
Si vous approvisionnez une nouvelle machine virtuelle SQL avec Resource Manager, le portail Azure permet d’activer l’intégration d’Azure Key Vault. La fonctionnalité Azure Key Vault est disponible uniquement pour les éditions Enterprise, Developer et Evaluation de SQL Server.

![Intégration du coffre de clés Azure SQL](./media/azure-key-vault-integration-configure/azure-sql-arm-akv.png)

Pour une procédure pas à pas de l’approvisionnement, consultez [Approvisionner une machine virtuelle SQL dans le portail Azure](create-sql-vm-portal.md).

### <a name="existing-vms"></a>Machines virtuelles existantes

Pour les machines virtuelles SQL existantes, ouvrez votre [ressource Machines virtuelles SQL](manage-sql-vm-portal.md#access-the-resource) et sélectionnez **Sécurité** sous **Paramètres**. Sélectionnez **Activer** pour activer l’intégration d’Azure Key Vault. 

![Intégration de Key Vault SQL pour les machines virtuelles existantes](./media/azure-key-vault-integration-configure/azure-sql-rm-akv-existing-vms.png)

Quand vous avez terminé, cliquez sur le bouton **Appliquer** en bas de la page **Sécurité** pour enregistrer vos modifications.

> [!NOTE]
> Le nom des informations d’identification que nous avons créées ici sera mappé ultérieurement à une connexion SQL. La connexion SQL pourra ainsi accéder au coffre de clés. 


> [!NOTE]
> Vous pouvez également configurer l’intégration de Key Vault à l’aide d’un modèle. Pour plus d’informations, consultez l’article [Azure quickstart template for Azure Key Vault integration](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.compute/vm-sql-existing-keyvault-update)(Modèle de démarrage rapide d’Azure pour l’intégration d’Azure Key Vault).


[!INCLUDE [Key Vault integration next steps](../../../../includes/virtual-machines-sql-server-akv-next-steps.md)]