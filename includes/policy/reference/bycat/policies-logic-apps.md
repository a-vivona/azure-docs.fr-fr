---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/03/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 18d2911eef94570d2d942b8d6c641f30c5ceadb6
ms.sourcegitcommit: e8b229b3ef22068c5e7cd294785532e144b7a45a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/04/2021
ms.locfileid: "123476938"
---
|Nom<br /><sub>(Portail Azure)</sub> |Description |Effet(s) |Version<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[L’environnement de service d’intégration Logic Apps doit être chiffré avec des clés gérées par le client](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1fafeaf6-7927-4059-a50a-8eb2a7a6f2b5) |Effectuez des déploiements dans un environnement de service d’intégration pour gérer le chiffrement au repos des données Logic Apps avec des clés gérées par le client. Par défaut, les données client sont chiffrées avec des clés gérées par le service. Cependant, des clés gérées par le client sont généralement nécessaires pour répondre aux standards de la conformité réglementaire. Les clés gérées par le client permettent de chiffrer les données avec une clé Azure Key Vault que vous avez créée et dont vous êtes le propriétaire. Vous avez le contrôle total et la responsabilité du cycle de vie des clés, notamment leur permutation et leur gestion. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_ISEWithCustomerManagedKey_AuditDeny.json) |
|[Logic Apps doit être déployé dans un environnement de service d’intégration](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdc595cb1-1cde-45f6-8faf-f88874e1c0e1) |Le déploiement de Logic Apps dans un environnement de service d’intégration au sein d’un réseau virtuel déverrouille les fonctionnalités avancées Logic Apps de sécurité et de réseau, et vous permet de mieux contrôler votre configuration réseau. Pour en savoir plus : [https://aka.ms/integration-service-environment](../../../../articles/logic-apps/connect-virtual-network-vnet-isolated-environment.md). Le déploiement dans un environnement de service d’intégration permet également de chiffrer les données avec des clés gérées par le client, ce qui améliore la protection des données en vous permettant de gérer vos clés de chiffrement. Cette configuration permet souvent de répondre aux exigences de conformité. |Audit, Refuser, Désactivé |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_LogicAppsInISE_AuditDeny.json) |
|[Les journaux de ressources dans Logic Apps doivent être activés](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34f95f76-5386-4de7-b824-0d8478470c9d) |Auditez l’activation des journaux de ressources. Permet de recréer les pistes d’activité à utiliser à des fins d’investigation en cas d’incident de sécurité ou de compromission du réseau. |AuditIfNotExists, Désactivé |[5.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_AuditDiagnosticLog_Audit.json) |