---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 08/27/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 51699ea3cc78febed5f5fc62632aa65e41df8c24
ms.sourcegitcommit: dcf1defb393104f8afc6b707fc748e0ff4c81830
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2021
ms.locfileid: "123108350"
---
|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Le groupe de conteneurs Azure Container Instances doit être déployé dans un réseau virtuel](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8af8f826-edcb-4178-b35f-851ea6fea615) |Sécurisez les communications entre vos conteneurs avec les réseaux virtuels Azure. Lorsque vous spécifiez un réseau virtuel, les ressources au sein du réseau virtuel peuvent communiquer en toute sécurité et en privé entre elles. |Audit, Désactivé, Refus |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_VNET_Audit.json) |
|[Le groupe de conteneurs Azure Container Instances doit utiliser une clé gérée par le client pour le chiffrement](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0aa61e00-0a01-4a3c-9945-e93cffedf0e6) |Sécurisez vos conteneurs avec plus de flexibilité à l’aide de clés gérées par le client. Quand vous spécifiez une clé gérée par le client, cette clé est utilisée pour protéger et contrôler l’accès à la clé qui chiffre vos données. L’utilisation de clés gérées par le client fournit des fonctionnalités supplémentaires permettant de contrôler la rotation de la clé de chiffrement de clé ou d’effacer des données par chiffrement. |Audit, Désactivé, Refus |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_CMK_Audit.json) |
