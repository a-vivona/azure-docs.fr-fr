---
title: Offres de services managés dans la Place de marché Azure
description: Proposez à vos clients des services de gestion Azure Lighthouse par le biais d’offres de services gérés sur la Place de marché Azure.
ms.date: 05/11/2021
ms.topic: conceptual
ms.openlocfilehash: 10b32445fcf6d014219dd8559c9c1ac9b2905044
ms.sourcegitcommit: e2fa73b682a30048907e2acb5c890495ad397bd3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/16/2021
ms.locfileid: "114391673"
---
# <a name="managed-service-offers-in-azure-marketplace"></a>Offres de services managés dans la Place de marché Azure

Cet article décrit le type d’offre **Services gérés** sur la [Place de marché Azure](https://azuremarketplace.microsoft.com). Les offres de services gérés vous permettent d’offrir des services de gestion des ressources à des clients par le biais [d’Azure Lighthouse](../overview.md). Vous pouvez mettre ces offres à la disposition de tous les clients potentiels ou uniquement d’un ou plusieurs clients spécifiques. Dans la mesure où vous facturez directement les clients pour les coûts liés à ces services managés, Microsoft ne facture aucuns frais.

## <a name="understand-managed-service-offers"></a>Comprendre les offres de services gérés

Les offres de services gérés simplifient le processus d’intégration des clients pour Azure Lighthouse. Quand un client achète une offre dans la Place de marché Azure, il peut spécifier les abonnements et/ou les groupes de ressources à intégrer.

Après cela, les utilisateurs de votre organisation seront en mesure de travailler sur ces ressources à partir de votre locataire de gestion avec la [gestion des ressources déléguées Azure](architecture.md), en fonction de l’accès que vous avez défini lors de la création de l’offre. Cette opération s'effectue par le biais d'un manifeste spécifiant les utilisateurs, groupes et principaux de service Azure Active Directory (Azure AD) qui auront accès aux ressources du client, ainsi que les [rôles](tenants-users-roles.md) déterminant leur niveau d'accès.

> [!NOTE]
> Les offres de services managés peuvent ne pas être disponibles dans Azure Government et d’autres clouds nationaux.

## <a name="public-and-private-offers"></a>Offres publiques et privées

Chaque offre de service managé comprend un ou plusieurs plans. Les plans peuvent être privés ou publics.

Si vous souhaitez limiter votre offre à des clients spécifiques, vous pouvez publier un plan privé. Dans ce cas, le plan ne peut être acheté que pour les ID d’abonnement que vous spécifiez. Pour plus d’informations, voir [Offres privées](../../marketplace/private-offers.md).

> [!NOTE]
> Les offres privées ne sont pas prises en charge avec les abonnements souscrits via un revendeur participant au programme des fournisseurs de solutions cloud (CSP).

Les plans publics vous permettent de promouvoir vos services auprès de nouveaux clients. Ils sont généralement plus appropriés lorsque vous n’avez besoin que d’un accès limité au locataire du client. Une fois que vous avez établi une relation avec un client, si celui-ci décide d’accorder un accès supplémentaire à votre organisation, vous pouvez soit publier un nouveau plan privé exclusivement pour ce client, soit [intégrer celui-ci pour un accès supplémentaire à l’aide de modèles Resource Manager](../how-to/onboard-customer.md).

Le cas échéant, vous pouvez inclure des plans publics et privés dans la même offre.

> [!IMPORTANT]
> Une fois que vous avez publié un plan public, vous ne pouvez plus le changer en plan privé. Pour contrôler les clients qui peuvent accepter votre offre et déléguer des ressources, utilisez un plan privé. Avec un plan public, vous ne pouvez pas limiter la disponibilité à des clients spécifiques ou à un certain nombre de clients (en revanche, vous pouvez arrêter complètement la vente du plan si vous le souhaitez). Vous pouvez [supprimer l’accès à une délégation](../how-to/remove-delegation.md) après qu’un client a accepté une offre uniquement si vous avez inclus une **Autorisation** avec la **Définition de rôle** définie sur [Inscription des services managés, attribution Supprimer le rôle](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) lors de la publication de l’offre. Vous pouvez également contacter le client et lui demander de [supprimer votre accès](../how-to/view-manage-service-providers.md#remove-service-provider-offers).

## <a name="publish-managed-service-offers"></a>Publier des offres de services gérés

Pour savoir comment publier une offre de service managé, consultez [Publier une offre de services gérés sur la place de marché Azure](../how-to/publish-managed-services-offers.md).

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur l’[architecture](architecture.md) Azure Lighthouse et les [expériences de gestion multilocataires](cross-tenant-management-experience.md).
- [Publiez des offres de services managés](../how-to/publish-managed-services-offers.md) sur la Place de marché Azure.
