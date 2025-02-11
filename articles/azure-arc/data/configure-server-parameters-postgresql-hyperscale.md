---
title: Configurer des paramètres de serveur du moteur Postgres pour votre groupe de serveurs PostgreSQL Hyperscale sur Azure Arc
titleSuffix: Azure Arc-enabled data services
description: Configurer des paramètres de serveur du moteur Postgres pour votre groupe de serveurs PostgreSQL Hyperscale sur Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 07/30/2021
ms.topic: how-to
ms.openlocfilehash: e634bcc7d07cfba4016c8f2db323e78e9beda92a
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122532063"
---
# <a name="set-the-database-engine-settings-for-azure-arc-enabled-postgresql-hyperscale"></a>Définir les paramètres du moteur de base de données pour PostgreSQL Hyperscale activé par Azure Arc

Ce document décrit les étapes permettant de définir les paramètres du moteur de base de données de votre groupe de serveurs PostgreSQL Hyperscale sur des valeurs personnalisées (autres que par défaut). Pour plus d’informations sur les paramètres du moteur de base de données qui peuvent être définis et leur valeur par défaut, consultez la documentation de PostgreSQL [ici](https://www.postgresql.org/docs/current/runtime-config.html).

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

> [!NOTE]
> La préversion ne prend pas en charge la définition des paramètres suivants : 
>
> - `archive_command`
> - `archive_timeout`
> - `log_directory`
> - `log_file_mode`
> - `log_filename`
> - `restore_command`
> - `shared_preload_libraries`
> - `synchronous_commit`
> - `ssl`
> - `wal_level`

## <a name="syntax"></a>Syntaxe

Le format général de la commande permettant de configurer les paramètres du moteur de base de données est le suivant :

```azurecli
az postgres arc-server edit -n <server group name>, [{--engine-settings, -e}] [{--replace-settings , --re}] {'<parameter name>=<parameter value>, ...'} --k8s-namespace <namespace> --use-k8s
```

## <a name="show-current-custom-values"></a>Afficher les valeurs personnalisées actuelles

### <a name="with-azure-data-cli-azdata-command"></a>Avec une commande [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

```azurecli
az postgres arc-server show -n <server group name> --k8s-namespace <namespace> --use-k8s
```

Exemple :

```azurecli
az postgres arc-server show -n postgres01 --k8s-namespace <namespace> --use-k8s 
```

cette commande retourne les spécifications du groupe de serveurs dans lequel vous pouvez voir les paramètres que vous avez définis. S’il n’existe pas de section engine\settings, cela signifie que tous les paramètres s’exécutent sur la base de leur valeur par défaut :

```console
 "
...
engine": {
      "settings": {
        "default": {
          "autovacuum_vacuum_threshold": "65"
        }
      }
    },
...
```

### <a name="with-kubectl-command"></a>Avec la commande kubectl

Procédez comme suit.

1. Récupérez le type de définition de ressource personnalisée pour votre groupe de serveurs

   Exécutez :

   ```azurecli
   az postgres arc-server show -n <server group name> --k8s-namespace <namespace> --use-k8s
   ```

   Exemple :

   ```azurecli
   az postgres arc-server show -n postgres01 --k8s-namespace <namespace> --use-k8s
   ```

   cette commande retourne les spécifications du groupe de serveurs dans lequel vous pouvez voir les paramètres que vous avez définis. S’il n’existe pas de section engine\settings, cela signifie que tous les paramètres s’exécutent sur la base de leur valeur par défaut :

   ```output
   > {
     >"apiVersion": "arcdata.microsoft.com/v1alpha1",
     >"**kind**": "**postgresql-12**",
     >"metadata": {
       >"creationTimestamp": "2020-08-25T14:32:23Z",
       >"generation": 1,
       >"name": "postgres01",
       >"namespace": "arc",  
   ```

   Dans les résultats de sortie, recherchez le champ `kind` et réservez la valeur, par exemple : `postgresql-12`.

2. Décrivez la ressource personnalisée Kubernetes correspondant à votre groupe de serveurs 

   Le format général de la commande est le suivant :

   ```console
   kubectl describe <kind of the custom resource> <server group name> -n <namespace name>
   ```

   Exemple :

   ```console
   kubectl describe postgresql-12 postgres01
   ```

   Si des valeurs personnalisées sont définies pour les paramètres du moteur, celui-ci les retourne. Exemple :

   ```output
   Engine:
   ...
    Settings:
      Default:
        autovacuum_vacuum_threshold:  65
   ```

   Si vous n’avez pas défini de valeurs personnalisées pour l’un des paramètres du moteur, la section Paramètres du moteur du `resultset` est vide comme suit :

   ```output
   Engine:
   ...
       Settings:
         Default:
   ```

## <a name="set-custom-values-for-engine-settings"></a>Définir des valeurs personnalisées pour les paramètres du moteur

Les commandes ci-dessous définissent les paramètres du nœud coordinateur et des nœuds Worker de votre PostgresSQL Hyperscale sur les mêmes valeurs. Il n’est pas encore possible de définir des paramètres par rôle dans votre groupe de serveurs. Autrement dit, il n’est pas encore possible de configurer un paramètre donné sur une valeur spécifique sur le nœud coordinateur et sur une autre valeur sur les nœuds Worker.

### <a name="set-a-single-parameter"></a>Définir un paramètre unique

```azurecli
az postgres arc-server edit -n <server group name> --engine-settings  <parameter name>=<parameter value> --k8s-namespace <namespace> --use-k8s
```

Exemple :

```azurecli
az postgres arc-server edit -n postgres01 --engine-settings  shared_buffers=8MB --k8s-namespace <namespace> --use-k8s
```

### <a name="set-multiple-parameters-with-a-single-command"></a>Définir plusieurs paramètres avec une seule commande

```azurecli
az postgres arc-server edit -n <server group name> --engine-settings  '<parameter name>=<parameter value>, <parameter name>=<parameter value>, --k8s-namespace <namespace> --use-k8s...'
```

Exemple :

```azurecli
az postgres arc-server edit -n postgres01 --engine-settings  'shared_buffers=8MB, max_connections=50' --k8s-namespace <namespace> --use-k8s
```

### <a name="reset-a-parameter-to-its-default-value"></a>Rétablir la valeur par défaut d’un paramètre

Pour rétablir la valeur par défaut d’un paramètre, définissez-le sans indiquer de valeur. 

Exemple :

```azurecli
az postgres arc-server edit -n postgres01 --k8s-namespace <namespace> --use-k8s --engine-settings  shared_buffers=
```

### <a name="reset-all-parameters-to-their-default-values"></a>Rétablir les valeurs par défaut de tous les paramètres

```azurecli
az postgres arc-server edit -n <server group name> --engine-settings  '' -re --k8s-namespace <namespace> --use-k8s
```

Exemple :

```azurecli
az postgres arc-server edit -n postgres01 --engine-settings  '' -re --k8s-namespace <namespace> --use-k8s
```

## <a name="special-considerations"></a>Considérations spéciales

### <a name="set-a-parameter-which-value-contains-a-comma-space-or-special-character"></a>Définir un paramètre dont la valeur contient une virgule, une espace ou un caractère spécial

```azurecli
az postgres arc-server edit -n <server group name> --engine-settings  '<parameter name>="<parameter value>"' --k8s-namespace <namespace> --use-k8s
```

Exemple :

```azurecli
az postgres arc-server edit -n postgres01 --engine-settings  'custom_variable_classes = "plpgsql,plperl"' --k8s-namespace <namespace> --use-k8s
```

### <a name="pass-an-environment-variable-in-a-parameter-value"></a>Passer une variable d’environnement dans une valeur de paramètre

Pour que la variable d’environnement ne soit pas résolue avant sa définition, elle doit être encapsulée dans "’’".

Par exemple : 

```azurecli
az postgres arc-server edit -n postgres01 --engine-settings  'search_path = "$user"' --k8s-namespace <namespace> --use-k8s
```

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [effectuer un scale-out (ajout de nœuds Worker)](scale-out-in-postgresql-hyperscale-server-group.md) de votre groupe de serveurs
- Découvrez comment [effectuer un scale-up ou un scale-down (augmentation/diminution de la mémoire/des vCores)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) de votre groupe de serveurs
