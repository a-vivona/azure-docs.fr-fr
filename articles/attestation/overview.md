---
title: Vue d’ensemble d’Azure Attestation
description: Vue d’ensemble de Microsoft Azure Attestation, solution pour l’attestation des environnements d’exécution approuvés (TEE, Trusted Execution Environment).
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.custom: references_regions
ms.openlocfilehash: fe025fad4955095ef16b546b7d326d80b4aea15c
ms.sourcegitcommit: d137460f55a38a0e8f8b9e6594e480d5e5f662ed
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/22/2021
ms.locfileid: "112427581"
---
# <a name="microsoft-azure-attestation"></a>Microsoft Azure Attestation 

Microsoft Azure Attestation est une solution unifiée permettant de vérifier à distance la fiabilité d’une plateforme et l’intégrité des ressources binaires qui s’y exécutent. Le service prend en charge l’attestation des plateformes reposant sur des modules de plateforme sécurisée (TPM), en plus de la possibilité d’attester de l’état des environnements d’exécution de confiance (TEE), tels que les enclaves [Intel® SGX](https://www.intel.com/content/www/us/en/architecture-and-technology/software-guard-extensions.html) (Software Guard Extensions) et les enclaves de [sécurité basée sur la virtualisation](/windows-hardware/design/device-experiences/oem-vbs) (VBS). 

L’attestation est un processus permettant de démontrer que les binaires logiciels ont été correctement instanciés sur une plateforme approuvée. Les parties de confiance distantes peuvent alors avoir l’assurance que seuls des logiciels de ce type s’exécutent sur du matériel approuvé. Azure Attestation est un framework et un service orientés client unifiés pour l’attestation.

Azure Attestation permet des paradigmes de sécurité de pointe tels que l’[informatique confidentielle Azure](../confidential-computing/overview.md) et la protection de périphérie intelligente. Les clients ont demandé à pouvoir vérifier indépendamment la localisation d’un ordinateur, la posture d’une machine virtuelle sur cet ordinateur et l’environnement dans lequel les enclaves s’exécutent sur cette machine virtuelle. Azure Attestation permet de répondre à ces demandes des clients et à bien d’autres.

Azure Attestation reçoit des preuves des entités de calcul, les transforme en un ensemble de revendications, les valide par rapport à des stratégies configurables et génère des preuves cryptographiques pour les applications basées sur les revendications (par exemple, les parties de confiance et les autorités d’audit).

## <a name="use-cases"></a>Cas d'utilisation

Azure Attestation fournit des services d’attestation complets pour plusieurs environnements et des cas d’usage distincts.

### <a name="sgx-attestation"></a>Attestation SGX

SGX fait référence à l’isolation matérielle, qui est prise en charge sur certains modèles de processeur Intel. SGX permet au code de s’exécuter dans des compartiments assainis appelés enclaves SGX. Les autorisations d’accès et de mémoire sont ensuite gérées par le matériel pour garantir une surface d’attaque minimale avec une isolation appropriée.

Les applications clientes peuvent être conçues pour tirer parti des enclaves SGX, en y déléguant l’exécution des tâches sensibles à la sécurité. De telles applications peuvent ensuite utiliser Azure Attestation pour établir régulièrement une approbation dans l’enclave et utiliser sa capacité à accéder aux données sensibles.

Les processeurs Intel® Xeon® Scalable prennent uniquement en charge les [solutions d’attestation basées sur ECDSA](https://software.intel.com/content/www/us/en/develop/topics/software-guard-extensions/attestation-services.html#Elliptic%20Curve%20Digital%20Signature%20Algorithm%20(ECDSA)%20Attestation) pour l’attestation à distance des enclaves SGX. Avec le modèle d’attestation basé sur ECDSA, Azure Attestation prend en charge la validation des processeurs Intel® Xeon® E3 et des plateformes de serveur Intel® Xeon® évolutives. 

> [!NOTE]
> Pour effectuer l’attestation des plateformes de serveur à processeur Intel® Xeon® Scalable à l’aide d’Azure Attestation, les utilisateurs doivent installer [Azure DCAP version 1.10.0](https://github.com/microsoft/Azure-DCAP-Client) ou supérieure.

### <a name="open-enclave"></a>Open Enclave
[Open Enclave](https://openenclave.io/sdk/) (OE) est un ensemble de bibliothèques ciblant la création d’une seule abstraction d’enclavement unifiée permettant aux développeurs de créer des applications basées sur un environnement TEE. Il offre un modèle d’application sécurisé universel qui réduit les spécificités de la plateforme. Microsoft le considère comme une étape essentielle pour la démocratisation des technologies d’enclave basée sur le matériel, telles que SGX, et leur adoption croissante sur Azure.

OE standardise des exigences spécifiques en matière de vérification d’une preuve d’enclave. Cela fait d’OE un consommateur d’Azure Attestation approprié.

### <a name="tpm-attestation"></a>Attestation TPM 

L’attestation basée sur le Module de plateforme sécurisée (TPM) est essentielle pour fournir la preuve de l’état d’une plateforme. Le TPM joue le rôle de racine de confiance et de coprocesseur de sécurité pour fournir la validité du chiffrement aux mesures (preuve). Les appareils disposant d’un TPM peuvent s’appuyer sur l’attestation pour prouver que l’intégrité du démarrage n’est pas compromise, et utiliser les revendications pour détecter les capacités d’activation des états de fonctionnalités au démarrage. 

Les applications clientes peuvent être conçues pour tirer parti de l’attestation TPM en déléguant les tâches sensibles à la sécurité afin qu’elles aient lieu uniquement après qu’une plateforme a été validée comme étant sécurisée. De telles applications peuvent ensuite utiliser Azure Attestation pour établir régulièrement une approbation dans la plateforme et utiliser sa capacité à accéder aux données sensibles.

## <a name="azure-attestation-can-run-in-a-tee"></a>Azure Attestation peut s’exécuter dans un environnement TEE

Azure Attestation est essentiel aux scénarios d’informatique confidentielle, car il effectue les actions suivantes :

- Il vérifie si la preuve de l’enclave est valide.
- Il évalue la preuve de l’enclave par rapport à une stratégie définie par le client.
- Il gère et stocke les stratégies propres au locataire.
- Il génère et signe un jeton qui est utilisé par les parties de confiance pour interagir avec l’enclave.

Azure Attestation est conçu pour s’exécuter dans deux types d’environnements :
- Environnement TEE activé pour SGX
- Environnement non-TEE

Les clients d’Azure Attestation ont exprimé la nécessité pour Microsoft d’être opérationnellement hors de la base TCB (Trusted Computing Base). Cela permet d’empêcher les entités Microsoft telles que les administrateurs de machines virtuelles, les administrateurs d’hôtes et les développeurs Microsoft de modifier les demandes d’attestation, les stratégies et les jetons émis par Azure Attestation. Azure Attestation est également conçu pour s’exécuter dans un environnement TEE, où les fonctionnalités d’Azure Attestation telles que la validation de déclarations, la génération de jetons et la signature de jetons sont déplacées vers une enclave SGX.

## <a name="why-use-azure-attestation"></a>Pourquoi utiliser Azure Attestation

Azure Attestation demeure le premier choix pour l’attestation des environnements TEE, car il offre les avantages suivants : 

- Infrastructure unifiée pour l’attestation de divers environnements comme les TPM, les enclaves SGX et les enclaves VBS 
- Permet la création de fournisseurs d’attestations personnalisés et la configuration de stratégies pour limiter la génération de jetons
- Propose des fournisseurs partagés régionaux qui peuvent attester sans configuration de la part des utilisateurs
- Protège ses données durant leur utilisation avec une implémentation dans une enclave SGX
- Service haute disponibilité 

## <a name="business-continuity-and-disaster-recovery-bcdr-support"></a>Prise en charge de la BCDR (continuité de l’activité et reprise d’activité)

La [continuité d’activité et reprise d’activité](../best-practices-availability-paired-regions.md) (BCDR) pour Azure Attestation permet de limiter les interruptions de service résultant de problèmes de disponibilité ou d’événements de sinistre importants dans une région.

Les clusters déployés dans deux régions fonctionnent indépendamment dans des conditions normales. Dans le cas d’une défaillance ou d’une panne d’une région, voici ce qui se produit :

- La BCDR d’Azure Attestation fournit un basculement transparent dans lequel les clients n’ont pas besoin d’effectuer d’étape supplémentaire pour la récupération.
- [Azure Traffic Manager](../traffic-manager/index.yml) pour la région détecte que la sonde d’intégrité est détériorée et définit le point de terminaison sur la région associée
- Les connexions existantes ne fonctionnent pas et reçoivent une erreur serveur interne ou connaissent des problèmes de délai d’attente.
- Toutes les opérations du plan de contrôle sont bloquées. Les clients ne sont pas en mesure de créer des fournisseurs d’attestations dans la région primaire
- Toutes les opérations de plan de données, y compris les appels d’attestation et la configuration de stratégie, seront traitées par la région secondaire. Les clients peuvent continuer à travailler sur les opérations de plan de données avec l’URI d’origine correspondant à la région primaire

## <a name="next-steps"></a>Étapes suivantes
- Découvrez les [concepts de base d’Azure Attestation](basic-concepts.md).
- [Guide pratique pour créer et signer une stratégie d’attestation](author-sign-policy.md)
- [Configurer Azure Attestation à l’aide de PowerShell](quickstart-powershell.md)
