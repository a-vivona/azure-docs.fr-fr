---
title: Comptes de Stockage Azure et Batch
description: Découvrez plus d’informations sur les comptes Azure Batch et leur utilisation du point de vue du développement.
ms.topic: conceptual
ms.date: 05/25/2021
ms.openlocfilehash: 3dd071a55e8605b8b5b1cf72c517c9304878e101
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/25/2021
ms.locfileid: "110452046"
---
# <a name="batch-accounts-and-azure-storage-accounts"></a>Comptes de Stockage Azure et Batch

Un compte Azure Batch est une entité identifiée de façon unique au sein du service Batch. La plupart des solutions Batch utilisent le [stockage Azure](../storage/index.yml) pour stocker des fichiers de ressources et des fichiers de sortie. Par conséquent, chaque compte Batch est généralement associé à un compte de stockage correspondant.

## <a name="batch-accounts"></a>Comptes Batch

L’ensemble du traitement et toutes les ressources sont associés à un compte Batch. Lorsque votre application effectue une requête auprès du service Batch, il authentifie la requête en utilisant le nom du compte Azure Batch, l’URL du compte et une clé d’accès ou bien un jeton de Azure Active Directory.

Vous pouvez exécuter plusieurs charges de travail Batch dans un compte Batch unique. Vous pouvez également répartir vos charges de travail entre plusieurs comptes Batch associés au même abonnement, mais situés dans des régions Azure différentes.

Vous pouvez créer un compte Batch à l’aide du [Portail Azure](batch-account-create-portal.md) ou par programme, par exemple avec la [bibliothèque .NET de gestion Batch](batch-management-dotnet.md). Lorsque vous créez le compte, vous pouvez associer un compte de stockage Azure pour le stockage des données d’entrée et de sortie ou des applications liées au travail.

[!INCLUDE [batch-account-mode-include](../../includes/batch-account-mode-include.md)]


## <a name="azure-storage-accounts"></a>Comptes de stockage Azure

La plupart des solutions Batch utilisent Stockage Azure pour stocker les [fichiers de ressources](resource-files.md) et les fichiers de sortie. Par exemple, vos tâches Batch (y compris les tâches standard, de démarrage, de préparation des travaux et de validation des travaux) spécifient généralement des fichiers de ressources se trouvant dans un compte de stockage. Les comptes de stockage stockent également les données traitées et toutes les données de sortie générées.

Batch prend en charge les types de comptes Stockage Azure suivants :

- Comptes Usage général v2 (GPv2)
- Comptes Usage général v1 (GPv1)
- Comptes de stockage d’objets blob (actuellement pris en charge pour les pools dans la configuration de la machine virtuelle)

Pour plus d’informations sur les comptes de stockage, consultez [Vue d’ensemble des comptes de stockage Azure](../storage/common/storage-account-overview.md).

Vous pouvez associer un compte de stockage à votre compte Batch lorsque vous créez le compte Batch. Vous avez également la possibilité de le faire ultérieurement. Prenez en compte vos exigences en termes de coûts et de performances lorsque vous choisissez un compte de stockage. Par exemple, les options de compte de stockage Blob et GPv2 prennent en charge des [limites d’extensibilité et de capacité](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/) plus importantes que celles des options GPv1. (Contactez le support technique Azure pour demander une augmentation de la limite de stockage.) Ces options de compte peuvent améliorer les performances des solutions Batch qui possèdent un certain nombre de tâches de lecture ou d’écriture s’exécutant simultanément sur le compte de stockage.

Lorsqu’un compte de stockage est lié à un compte batch, il est considéré comme étant le *compte autostorage*. Un compte autostorage est nécessaire si vous envisagez d’utiliser la fonctionnalité [packages d’application](batch-application-packages.md), car elle est utilisée pour stocker les fichiers .zip du package d’application. Elle peut également être utilisée pour les [fichiers de ressources de tâche](resource-files.md#storage-container-name-autostorage); étant donné que le compte autostorage est déjà lié au compte batch, cela évite de devoir utiliser des URL de signature d’accès partagé (SAS) pour accéder aux fichiers de ressources.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [nœuds et les pools](nodes-and-pools.md).
- Découvrez comment créer et manager des comptes Batch à l’aide du [portail Azure](batch-account-create-portal.md) ou de [Batch Management .NET](batch-management-dotnet.md).
- En savoir plus sur l’utilisation des [points de terminaison privés](private-connectivity.md) avec des comptes Azure Batch.
