---
title: Définir la version de Redis pour Azure Cache pour Redis (préversion)
description: Découvrez comment configurer la version de Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 09/30/2020
ms.openlocfilehash: 23bc9f92f405fe29aa43b266c0b18b8620e1d18c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/26/2021
ms.locfileid: "110463083"
---
# <a name="set-redis-version-for-azure-cache-for-redis-preview"></a>Définir la version de Redis pour Azure Cache pour Redis (préversion)
Dans cet article, vous allez apprendre à configurer la version logicielle Redis à utiliser avec votre instance de cache. Azure Cache pour Redis offre la version principale la plus récente de ReDis et au moins une version précédente. Il mettra à jour ces versions régulièrement à mesure que de nouvelles versions de Redis seront publiées. Vous pouvez choisir entre les deux versions disponibles. N’oubliez pas que votre cache sera automatiquement mis à niveau vers la version suivante si la version qu’il utilise actuellement n’est plus prise en charge.

> [!NOTE]
> Redis 6 est actuellement en préversion. À ce stade, Redis 6 ne prend pas en charge le clustering, la redondance de zone, la liste de contrôle d’accès, PowerShell, Azure CLI, Terraform et la géo-réplication entre un cache Redis 4.0 et un cache 6.0. La version Redis ne peut pas non plus être modifiée une fois qu’un cache a été créé. 
>

> [!IMPORTANT]
> Une fois que Redis 6.0 est mis à la disposition générale (GA), Redis 6.0 est la version par défaut pour les nouveaux caches. Vous avez toujours la possibilité de créer des caches Redis 4.0 et vous serez en mesure de mettre à niveau vos caches Redis 4.0 vers des caches Redis 6.0 à la disponibilité générale. 
>

## <a name="prerequisites"></a>Prérequis
* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)

## <a name="create-a-cache"></a>Création d'un cache
Pour créer un cache, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Créer une ressource**.
  
1. Dans la page **Nouvelle**, sélectionnez **Bases de données**, puis **Azure Cache pour Redis**.

    :::image type="content" source="media/cache-create/new-cache-menu.png" alt-text="Sélectionnez Azure Cache pour Redis.":::
   
1. Dans la page **Général**, configurez les paramètres du nouveau cache.
   
    | Paramètre      | Valeur suggérée  | Description |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Abonnement** | Sélectionnez votre abonnement. | Abonnement sous lequel créer cette nouvelle instance d’Azure Cache pour Redis. | 
    | **Groupe de ressources** | Sélectionnez un groupe de ressources ou choisissez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources. | Nom du groupe de ressources dans lequel créer votre cache et d’autres ressources. En plaçant toutes les ressources de votre application dans un seul groupe de ressources, vous pouvez facilement les gérer ou les supprimer ensemble. | 
    | **Nom DNS** | Entrez un nom globalement unique. | Le nom du cache doit être une chaîne de 1 à 63 caractères ne contenant que des chiffres, des lettres et des traits d’union. Le nom doit commencer et se terminer par un chiffre ou une lettre, et ne peut pas contenir de traits d’union consécutifs. Le *nom d’hôte* de votre instance de cache sera *\<DNS name>.redis.cache.windows.net*. | 
    | **Lieu** | Sélectionnez un emplacement. | Choisissez une [Région](https://azure.microsoft.com/regions/) proche d’autres services qui utiliseront votre cache. |
    | **Type de cache** | Sélectionnez [un niveau et une taille de cache](https://azure.microsoft.com/pricing/details/cache/). |  Le niveau tarifaire détermine la taille, les performances et les fonctionnalités disponibles pour le cache. Pour plus d’informations, consultez [Présentation du cache Azure pour Redis](cache-overview.md). |
   
1. Dans la page **Avancé**, choisissez une version de Redis à utiliser.
   
    :::image type="content" source="media/cache-how-to-version/select-redis-version.png" alt-text="Version de Redis.":::

1. Cliquez sur **Créer**. 
   
    La création du cache prend un certain temps. Vous pouvez surveiller la progression dans la page **Vue d’ensemble** du Azure Cache pour Redis. Lorsque **État** indique **En cours d’exécution**, le cache est prêt pour utilisation.

    > [!NOTE]
    > À ce stade, la version de Redis ne peut pas être modifiée une fois qu’un cache a été créé.
    >

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les fonctionnalités d’Azure Cache pour Redis.

> [!div class="nextstepaction"]
> [Niveaux de service Premium Azure Cache pour Redis](cache-overview.md#service-tiers)
