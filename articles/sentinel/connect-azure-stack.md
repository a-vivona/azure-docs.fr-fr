---
title: Intégrer vos machines virtuelles Azure Stack Hub à Azure Sentinel | Microsoft Docs
description: Cet article vous montre comment approvisionner l’extension de machine virtuelle Azure Monitor, Update and Configuration Management sur des machines virtuelles Azure Stack Hub et commencer à les superviser avec Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/23/2019
ms.author: yelevin
ms.openlocfilehash: 38b595e4f9b2582f3baefecabf6285ee36436ad6
ms.sourcegitcommit: 2eac9bd319fb8b3a1080518c73ee337123286fa2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123260065"
---
# <a name="connect-azure-stack-hub-virtual-machines-to-azure-sentinel"></a>Connecter des machines virtuelles Azure Stack Hub à Azure Sentinel

Avec Azure Sentinel, vous pouvez superviser vos machines virtuelles s’exécutant sur Azure et Azure Stack Hub à un même emplacement. Pour intégrer vos machines Azure Stack à Azure Sentinel, vous devez d’abord ajouter l’extension de machine virtuelle à vos machines virtuelles Azure Stack Hub existantes. 

Après avoir connecté des machines Azure Stack Hub, vous pouvez faire votre choix parmi une galerie de tableaux de bord qui mettent en avant les insights provenant de vos sources de données. Ces tableaux de bord peuvent être aisément adaptés à vos besoins.

## <a name="add-the-virtual-machine-extension"></a>Ajouter l’extension de machine virtuelle 

Ajoutez l’extension de machine virtuelle **Azure Monitor, Update and Configuration Management** aux machines virtuelles en cours d’exécution sur votre système Azure Stack Hub. 

1. Dans un nouvel onglet de navigateur, connectez-vous à votre [portail Azure Stack Hub](/azure-stack/user/azure-stack-use-portal#access-the-portal).

1. Accédez à la page **Machines virtuelles**, puis sélectionnez la machine virtuelle que vous souhaitez protéger avec Azure Sentinel. Pour plus d’informations sur la création d’une machine virtuelle sur Azure Stack Hub, consultez [Créer une machine virtuelle Windows Server avec le portail Azure Stack Hub](/azure-stack/user/azure-stack-quick-windows-portal) ou [Créer une machine virtuelle serveur Linux sur le portail Azure Stack Hub](/azure-stack/user/azure-stack-quick-linux-portal).

1. Sélectionnez **Extensions**. La liste des extensions de machine virtuelle installées sur cette machine virtuelle s’affiche.

1. Sélectionnez l’onglet **Ajouter**. Le panneau de menu **Nouvelle ressource** s’ouvre, affichant la liste des extensions de machine virtuelle disponibles. 

1. Sélectionnez l’extension **Azure Monitor, Update and Configuration Management**, puis choisissez **Créer**. La fenêtre de configuration **Installer l’extension** s’ouvre.

   ![Paramètres d’Azure Monitor, Update, and Configuration Management](./media/connect-azure-stack/azure-monitor-extension-fix.png)  

   >[!NOTE]
   > Si l’extension **Azure Monitor, Update and Configuration Management** n’est pas listée dans votre place de marché, contactez votre opérateur Azure Stack Hub pour qu’elle soit disponible.

1. Dans le menu Azure Sentinel, sélectionnez **Paramètres d’espace de travail** suivi d’**Avancé**, puis copiez l’**ID d'espace de travail** et la **Clé de l’espace de travail (clé primaire)** . 

1. Dans la fenêtre **Installer l’extension** d’Azure Stack Hub, collez-les dans les champs indiqués, puis sélectionnez **OK**.

1. Une fois l’installation de l’extension terminée, son état s’affiche en tant qu’**Approvisionnement réussi**. Il peut s’écouler une heure avant que la machine virtuelle n’apparaisse dans le portail Azure Sentinel.

Pour plus d’informations sur l’installation et la configuration de l’agent pour Windows, consultez la page [Connecter des ordinateurs Windows](../azure-monitor/agents/agent-windows.md#install-agent-using-setup-wizard).

Pour plus d’informations sur la résolution des problèmes lié à l’agent Linux, consultez [Résoudre les problèmes liés à l’agent Linux Azure Log Analytics](../azure-monitor/agents/agent-linux-troubleshoot.md).

Dans le portail Azure Sentinel sur Azure, la fenêtre **Machines Virtuelles** affiche une vue d’ensemble de toutes les machines virtuelles et de tous les ordinateurs ainsi que du statut correspondant. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer l’extension de la machine virtuelle par le biais du portail Azure Stack Hub.

Pour supprimer l’extension :

1. Ouvrez le **portail Azure Stack Hub**.

1. Accédez à **Machines virtuelles**, sélectionnez la machine virtuelle dont vous souhaitez supprimer l’extension.

1. Sélectionnez **Extensions**, puis sélectionnez l’extension **Microsoft.EnterpriseCloud.Monitoring**.

1. Sélectionnez **Désinstaller**, puis confirmez votre sélection.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure Sentinel, voir les articles suivants :

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](detect-threats-built-in.md).
- Diffusez des données en streaming depuis des [appliances Common Event Format](connect-common-event-format.md) dans Azure Sentinel.
