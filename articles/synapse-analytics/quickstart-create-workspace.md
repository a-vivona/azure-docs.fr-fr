---
title: 'Démarrage rapide : créer un espace de travail Synapse'
description: Créez un espace de travail Synapse en suivant les étapes décrites dans ce guide.
services: synapse-analytics
author: saveenr
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: workspace
ms.date: 09/03/2020
ms.author: saveenr
ms.reviewer: jrasnick
ms.custom: subject-rbac-steps
ms.openlocfilehash: 0f593d801bdcc477d6084a395630211393ffa9c7
ms.sourcegitcommit: 6bd31ec35ac44d79debfe98a3ef32fb3522e3934
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2021
ms.locfileid: "113217284"
---
# <a name="quickstart-create-a-synapse-workspace"></a>Démarrage rapide : Créer un espace de travail Synapse
Ce guide de démarrage rapide décrit les étapes à suivre pour créer un espace de travail Azure Synapse à l’aide du portail Azure.

## <a name="create-a-synapse-workspace"></a>Créer un espace de travail Synapse

1. Ouvrez le [Portail Azure](https://portal.azure.com) et, en haut, recherchez **Synapse**.
1. Dans les résultats de la recherche, sous **Services**, sélectionnez **Azure Synapse Analytics**.
1. Sélectionnez **Ajouter** pour créer un espace de travail.
1. Dans l’onglet **Général**, donnez un nom unique à l’espace de travail. Dans ce document, nous allons utiliser **myworkspace**.
1. Un compte ADLS Gen2 est nécessaire pour créer un espace de travail. Le choix le plus simple consiste à en créer un nouveau. Si vous souhaitez en réutiliser un, vous devrez effectuer une configuration supplémentaire. 
1. OPTION 1 Création d’un nouveau compte ADLS Gen2 : 
    1. Sous **Sélectionnez Data Lake Storage Gen2**, cliquez sur **Créer** et nommez-le **contosolake**.
    1. Sous **Sélectionnez Data Lake Storage Gen2**, cliquez sur **Système de fichiers** et nommez-le **users**.
1. OPTION 2 Consultez les instructions **Préparation d’un compte de stockage** qui se trouvent à la fin de ce document.
1. Votre espace de travail Azure Synapse utilisera ce compte de stockage comme compte de stockage « principal » et le conteneur pour stocker les données de l’espace de travail. L’espace de travail stocke les données dans des tables Apache Spark. Il stocke les journaux des applications Spark dans un dossier appelé **/synapse/workspacename**.
1. Sélectionnez **Vérifier + créer** > **Créer**. Votre espace de travail est prêt en quelques minutes.

> [!NOTE]
> Après avoir créé votre espace de travail Azure Synapse, vous ne pourrez pas le déplacer vers un autre locataire Azure Active Directory. Si vous effectuez cette opération par le biais d’une migration d’abonnement ou d’autres actions, vous risquez de perdre l’accès aux artefacts dans l’espace de travail.
> De plus, vous ne pouvez pas créer un espace de travail Synapse Analytics dans un abonnement [Fournisseur de solutions cloud (CSP)](/partner-center/csp-overview) pour le moment.

## <a name="open-synapse-studio"></a>Ouvrir Synapse Studio

Après avoir créé votre espace de travail Azure Synapse, vous pouvez ouvrir Synapse Studio de deux manières :

* Ouvrez votre espace de travail Synapse dans le [Portail Azure](https://portal.azure.com). En haut de la section **Vue d’ensemble**, sélectionnez **Lancer Synapse Studio**.
* Accédez à `https://web.azuresynapse.net` et connectez-vous à votre espace de travail.

## <a name="prepare-an-existing-storage-account-for-use-with-azure-synapse-analytics"></a>Préparer un compte de stockage existant à utiliser avec Azure Synapse Analytics

1. Ouvrez le [portail Azure](https://portal.azure.com).
1. Accéder à un compte de stockage ADLSGEN2 existant
1. Sélectionnez **Contrôle d’accès (IAM)** .
1. Sélectionnez **Ajouter** > **Ajouter une attribution de rôle** pour ouvrir la page Ajouter une attribution de rôle.
1. Attribuez le rôle suivant. Pour connaître les étapes détaillées, consultez [Attribuer des rôles Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md).
    
    | Paramètre | Valeur |
    | --- | --- |
    | Role | Propriétaire et rôle Propriétaire des données Blob du stockage |
    | Attribuer l’accès à | [USER |
    | Membres | Votre nom d’utilisateur |

    ![Page Ajouter une attribution de rôle dans le portail Azure.](../../includes/role-based-access-control/media/add-role-assignment-page.png)
1. Dans le volet de gauche, sélectionnez **Conteneurs** et créez un conteneur.
1. Attribuez au conteneur un nom de votre choix. Dans ce document, nous appellerons le conteneur **users**.
1. Acceptez le paramètre par défaut **Niveau d’accès public**, puis sélectionnez **Créer**.

### <a name="configure-access-to-the-storage-account-from-your-workspace"></a>Configuration de l’accès au compte de stockage à partir de votre espace de travail

Les identités managées pour votre espace de travail Azure Synapse ont peut-être déjà accès au compte de stockage. Vérifiez ce point en effectuant ces étapes :

1. Ouvrez le [Portail Azure](https://portal.azure.com) et le compte de stockage principal choisi pour votre espace de travail.
1. Sélectionnez **Contrôle d’accès (IAM)** .
1. Sélectionnez **Ajouter** > **Ajouter une attribution de rôle** pour ouvrir la page Ajouter une attribution de rôle.
1. Attribuez le rôle suivant. Pour connaître les étapes détaillées, consultez [Attribuer des rôles Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md).
    
    | Paramètre | Valeur |
    | --- | --- |
    | Role | Contributeur aux données Blob du stockage |
    | Attribuer l’accès à | MANAGEDIDENTITY |
    | Membres | myworkspace  |

    > [!NOTE]
    > Le nom de l’identité managée correspond également au nom de l’espace de travail.

    ![Page Ajouter une attribution de rôle dans le portail Azure.](../../includes/role-based-access-control/media/add-role-assignment-page.png)
1. Sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

* [Créer un pool SQL dédié](quickstart-create-sql-pool-studio.md) 
* [Créer un pool Apache Spark serverless](quickstart-create-apache-spark-pool-portal.md)
* [Utiliser un pool SQL serverless](quickstart-sql-on-demand.md)