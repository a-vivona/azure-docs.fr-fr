---
title: Journalisation pour HSM managé Azure
description: Utilisez ce tutoriel pour démarrer avec la journalisation pour HSM managé.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 03/30/2021
ms.author: mbaldwin
ms.openlocfilehash: 029913125104f1c4eaa6ed627feae6a4ec207bd9
ms.sourcegitcommit: 351279883100285f935d3ca9562e9a99d3744cbd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2021
ms.locfileid: "112379212"
---
# <a name="managed-hsm-logging"></a>Journalisation pour HSM managé 

Une fois que vous avez créé un ou plusieurs HSM managés, vous voulez probablement superviser qui accède à ces derniers, par quel moyen et quand. Pour ce faire, vous pouvez activer la journalisation, ce qui permet d’enregistrer les informations dans un compte de stockage Azure que vous fournissez. Un conteneur nommé **insights-logs-auditevent** est automatiquement créé pour le compte de stockage spécifié. Vous pouvez utiliser ce même compte de stockage pour collecter les journaux de plusieurs HSM managés.

Vous pouvez accéder aux informations de journalisation 10 minutes (au maximum) après l’opération sur le HSM managé. Dans la plupart des cas, ce sera plus rapide.  C’est à vous de gérer vos journaux d’activité dans votre compte de stockage :

* Utilisez les méthodes de contrôle d’accès Azure standard pour sécuriser vos journaux en limitant qui y a accès.
* Supprimez les journaux d’activité que vous ne souhaitez plus conserver dans votre compte de stockage.

Utilisez ce tutoriel pour démarrer avec la journalisation pour HSM managé. Vous allez créer un compte de stockage, activer la journalisation et interpréter les informations de journal collectées.  

> [!NOTE]
> Ce tutoriel n’inclut pas d’instructions pour créer des HSM managés ni des clés. Cet article fournit des instructions Azure CLI pour la mise à jour de la journalisation des diagnostics.

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous devez disposer des éléments suivants :

* Un abonnement à Microsoft Azure. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI version 2.25.0 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).
* HSM managé dans votre abonnement. Consultez [Démarrage rapide : Provisionner et activer un HSM managé à l’aide d’Azure CLI](quick-create-cli.md) pour provisionner et activer un HSM managé.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="connect-to-your-azure-subscription"></a>Connectez-vous à un abonnement Azure

La première étape de la configuration de la journalisation de clés consiste à faire pointer Azure CLI vers le HSM managé à journaliser.

```azurecli-interactive
az login
```

Pour plus d’informations sur les options de connexion par le biais de l’interface CLI, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli)

Vous devrez peut-être spécifier l’abonnement que vous avez utilisé pour créer votre HSM managé. Entrez la commande ci-après pour afficher les abonnements de votre compte :

## <a name="identify-the-managed-hsm-and-storage-account"></a>Identifier le HSM managé et le compte de stockage

```azurecli-interactive
hsmresource=$(az keyvault show --hsm-name ContosoMHSM --query id -o tsv)
storageresource=$(az storage account show --name ContosoMHSMLogs --query id -o tsv)
```

## <a name="enable-logging"></a>Activation de la journalisation

Pour activer la journalisation du HSM managé, utilisez la commande **az monitor diagnostic-settings create** ainsi que les variables que nous avons créées pour le nouveau compte de stockage et le HSM managé. Nous allons également définir l’indicateur **-Enabled** sur **$true** et la catégorie sur **AuditEvent** (la seule catégorie pour la journalisation pour HSM managé) :

Cette sortie confirme que la journalisation est à présent activée pour votre HSM managé et qu’elle enregistrera les informations dans votre compte de stockage.

Si vous le souhaitez, vous pouvez définir une stratégie de rétention pour vos journaux, par exemple la suppression automatique des anciens journaux. Par exemple, définissez une stratégie de rétention en paramétrant l’indicateur **-RetentionEnabled** sur la valeur **$true** et le paramètre **-RetentionInDays** sur la valeur **90** afin que les journaux antérieurs à 90 jours soient automatiquement supprimés.

```azurecli-interactive
az monitor diagnostic-settings create --name ContosoMHSM-Diagnostics --resource $hsmresource --logs '[{"category": "AuditEvent","enabled": true}]' --storage-account $storageresource
```

Éléments consignés :

* Toutes les requêtes d’API REST authentifiées, ce qui inclut les requêtes ayant échoué suite à des autorisations d’accès, des erreurs système ou des requêtes incorrectes.
* Les opérations de plan managées sur la ressource HSM managé elle-même, notamment la création, la suppression et la mise à jour d’attributs comme des balises.
* Opérations liées au domaine de sécurité, telles que l’initialisation et le téléchargement, l’initialisation de la récupération, le chargement
* Opérations de sauvegarde complète du HSM, de restauration et de restauration sélective
* Opérations de gestion des rôles comme créer/afficher/supprimer des attributions de rôles et créer/afficher/supprimer des définitions de rôle personnalisées
* Opérations sur les clés, notamment :
  * Création, modification ou suppression des clés.
  * Signature, vérification, chiffrement, déchiffrement, enveloppement et désenveloppement de clés, listage de clés.
  * Sauvegarde de clés, restauration, vidage
* les requêtes non authentifiées qui génèrent une réponse 401. Il s’agit notamment des requêtes dépourvues de jeton du porteur, dont le format est incorrect, qui ont expiré ou qui comportent un jeton non valide.  

## <a name="access-your-logs"></a>Accéder à vos journaux d’activité

Les journaux pour HSM managé sont stockés dans le conteneur **insights-logs-auditevent** du compte de stockage que vous avez fourni. Pour visualiser les journaux, vous devez télécharger des objets blob. Pour plus d’informations sur le stockage Azure, consultez [Créer, télécharger et lister des objets blob avec Azure CLI](../../storage/blobs/storage-quickstart-blobs-cli.md).

Les objets blob individuels sont stockés sous forme de texte, au format JSON. Examinons un exemple d’entrée du journal. L’exemple ci-dessous illustre l’entrée de journal quand une demande de création d’une sauvegarde complète est envoyée au HSM managé.

```json
[
  {
    "TenantId": "766eaf62-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "time": "2020-08-31T19:52:39.763Z",
    "resourceId": "/SUBSCRIPTIONS/A1BA9AAA-xxxx-xxxx-xxxx-xxxxxxxxxxxx/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS/MICROSOFT.KEYVAULT/MANAGEDHSMS/CONTOSOMHSM",
    "operationName": "BackupCreate",
    "operationVersion": "7.0",
    "category": "AuditEvent",
    "resultType": "Success",
    "properties": {
        "PoolType": "M-HSM",
        "sku_Family": "B",
        "sku_Name": "Standard_B1"
    },
    "durationMs": 488,
    "callerIpAddress": "X.X.X.X",
    "identity": "{\"claim\":{\"appid\":\"04b07795-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"http_schemas_microsoft_com_identity\":{\"claims\":{\"objectidentifier\":\"b1c52bf0-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"}},\"http_schemas_xmlsoap_org_ws_2005_05_identity\":{\"claims\":{\"upn\":\"admin@contoso.com\"}}}}",
    "clientInfo": "azsdk-python-core/1.7.0 Python/3.8.2 (Linux-4.19.84-microsoft-standard-x86_64-with-glibc2.29) azsdk-python-azure-keyvault/7.2",
    "correlationId": "8806614c-ebc3-11ea-9e9b-00155db778ad",
    "subnetId": "(unknown)",
    "httpStatusCode": 202,
    "PoolName": "mhsmdemo",
    "requestUri": "https://ContosoMHSM.managedhsm.azure.net/backup",
    "resourceGroup": "ContosoResourceGroup",
    "resourceProvider": "MICROSOFT.KEYVAULT",
    "resource": "ContosoMHSM",
    "resourceType": "managedHSMs"
  }
]
```



## <a name="use-azure-monitor-logs"></a>Utiliser les journaux d’activité Azure Monitor

Vous pouvez utiliser la solution Key Vault dans les journaux Azure Monitor pour consulter les journaux **AuditEvent** pour HSM managé. Dans les journaux Azure Monitor, vous utilisez des requêtes de journaux pour analyser les données et obtenir les informations dont vous avez besoin.

Pour plus d’informations, notamment sur la procédure de configuration correspondante, consultez [Azure Key Vault dans Azure Monitor](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrir les [bonnes pratiques](best-practices.md) pour provisionner et utiliser un HSM managé
- Découvrir la [procédure de sauvegarde et de restauration](backup-restore.md) d’un HSM managé
