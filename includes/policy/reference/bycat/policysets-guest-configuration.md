---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/03/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 8c559cd62092d3235d27a94c40cc39457b18a82e
ms.sourcegitcommit: e8b229b3ef22068c5e7cd294785532e144b7a45a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/04/2021
ms.locfileid: "123474749"
---
|Nom |Description |Stratégies |Version |
|---|---|---|---|
|[Auditer les machines avec des paramètres de sécurité de mot de passe non sécurisés](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordSettingsAINE.json) |Cette initiative permet de déployer les exigences de stratégie et d’auditer les machines avec des paramètres de sécurité de mot de passe non sécurisés. Pour plus d’informations sur les stratégies Guest Configuration, consultez [https://aka.ms/gcpol](../../../../articles/governance/policy/concepts/guest-configuration.md) |9 |1.0.0 |
|[Déployer les prérequis pour activer les stratégies Guest Configuration sur les machines virtuelles](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json) |Cette initiative ajoute une identité managée affectée par le système, et déploie l’extension Guest Configuration appropriée à la plateforme sur les machines virtuelles pouvant être supervisées par des stratégies Guest Configuration. Il s’agit d’un prérequis pour toutes les stratégies Guest Configuration, et il doit être affecté à l’étendue d’attribution des stratégies avant l’utilisation d’une stratégie Guest Configuration. Pour plus d’informations sur Guest Configuration, consultez [https://aka.ms/gcpol](../../../../articles/governance/policy/concepts/guest-configuration.md). |4 |1.0.0 |
|[\[Préversion\] : Les machines Windows doivent répondre aux impératifs de référence liés à la sécurité d’Azure Compute](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_AzureBaseline.json) |Cette initiative audite les machines Windows dont les paramètres ne correspondent pas à la base de référence de sécurité pour Azure Compute. Pour plus d’informations, consultez [https://aka.ms/gcpol](../../../../articles/governance/policy/concepts/guest-configuration.md). |29 |2.0.1-preview |