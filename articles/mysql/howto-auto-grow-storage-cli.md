---
title: Augmenter automatiquement le stockage - Azure CLI - Azure Database pour MySQL
description: Cet article explique comment vous pouvez activer la croissance automatique du stockage avec Azure CLI dans Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: a296a4e74e445a067e6b308eeffea126312d1b5d
ms.sourcegitcommit: 8b38eff08c8743a095635a1765c9c44358340aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/30/2021
ms.locfileid: "122641135"
---
# <a name="auto-grow-azure-database-for-mysql-storage-using-the-azure-cli"></a>Activer la croissance automatique pour un serveur Azure Database pour MySQL avec Azure CLI

[!INCLUDE[applies-to-mysql-single-server](includes/applies-to-mysql-single-server.md)]
Cet article explique comment vous pouvez configurer l’augmentation d’un stockage de serveur Azure Database pour MySQL sans affecter la charge de travail.

Le serveur [qui atteint la limite de stockage](./concepts-pricing-tiers.md#reaching-the-storage-limit) est placé en lecture seule. Si la croissance automatique du stockage est activée pour les serveurs avec moins de 100 Go de stockage provisionnés, la taille du stockage provisionné augmente de 5 Go dès que l’espace de stockage disponible est inférieur à 1 Go ou à 10 % du stockage provisionné (selon la valeur la plus élevée). Pour les serveurs avec plus de 100 Go de stockage approvisionnés, la taille de stockage approvisionnée augmente de 5 % lorsque l’espace de stockage libre est inférieur à 10 Go de taille de stockage approvisionnée. Les limites de stockage maximales spécifiées [ici](./concepts-pricing-tiers.md#storage) s’appliquent.

## <a name="prerequisites"></a>Prérequis

Pour suivre cette procédure :

- Vous avez besoin d’un [serveur Azure Database pour MySQL](quickstart-create-mysql-server-database-using-azure-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article demande la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="enable-mysql-server-storage-auto-grow"></a>Activer la croissance automatique du stockage du serveur MySQL

Activez le stockage à croissance automatique du serveur sur un serveur existant avec la commande suivante :

```azurecli-interactive
az mysql server update --name mydemoserver --resource-group myresourcegroup --auto-grow Enabled
```

Activez le stockage à croissance automatique du serveur sur un nouveau serveur avec la commande suivante :

```azurecli-interactive
az mysql server create --resource-group myresourcegroup --name mydemoserver  --auto-grow Enabled --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 5.7
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez [comment créer des alertes sur des métriques](howto-alert-on-metric.md).
