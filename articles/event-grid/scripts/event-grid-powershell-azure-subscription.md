---
title: Azure PowerShell - S’abonner à un abonnement Azure
description: Cet article fournit un exemple de script Azure PowerShell qui montre comment s’abonner à des événements Event Grid pour un abonnement Azure.
ms.devlang: powershell
ms.topic: sample
ms.date: 07/22/2021
ms.openlocfilehash: b14d1a07725b015b436f48dc03f0c21ce1e74226
ms.sourcegitcommit: 7d63ce88bfe8188b1ae70c3d006a29068d066287
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/22/2021
ms.locfileid: "114467413"
---
# <a name="subscribe-to-events-for-an-azure-subscription-with-powershell"></a>S’abonner aux événements d’un abonnement Azure avec PowerShell

Ce script crée un abonnement Event Grid aux événements d’un abonnement Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script---stable"></a>Exemple de script : stable

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/event-grid/subscribe-to-azure-subscription/subscribe-to-azure-subscription.ps1 "Subscribe to Azure subscription")]

## <a name="sample-script---preview-module"></a>Exemple de script - module en préversion

Cet exemple de script nécessite le module Event Grid. Pour l’installer, exécutez `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`

[!INCLUDE [requires-azurerm](../../../includes/requires-azurerm.md)]

[!code-powershell[main](../../../powershell_scripts/event-grid/subscribe-to-azure-subscription-preview/subscribe-to-azure-subscription-preview.ps1 "Subscribe to Azure subscription")]

## <a name="script-explanation"></a>Explication du script

Ce script utilise la commande suivante pour créer l’abonnement aux événements. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzEventGridSubscription](/powershell/module/az.eventgrid/new-azeventgridsubscription) | Créer un abonnement Event Grid. |

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation des applications gérées, consultez [Vue d’ensemble des applications gérées Azure](../overview.md).
* Pour plus d’informations sur PowerShell, consultez la [documentation Azure PowerShell](/powershell/azure/get-started-azureps).
