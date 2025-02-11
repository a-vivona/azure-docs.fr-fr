---
title: Support de Network File System 3.0 dans Stockage Blob Azure | Microsoft Docs
description: Le stockage Blob Azure prend désormais en charge le protocole NFS (Network File System) 3.0. Cette prise en charge permet aux clients Linux de monter un conteneur dans Stockage Blob à partir d’une machine virtuelle Azure ou d’un ordinateur qui s’exécute localement.
author: normesta
ms.subservice: blobs
ms.service: storage
ms.topic: conceptual
ms.date: 06/21/2021
ms.author: normesta
ms.reviewer: yzheng
ms.openlocfilehash: f350b4fad79a80a1bba0572c5ed906aa4aff1168
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/29/2021
ms.locfileid: "113005453"
---
# <a name="network-file-system-nfs-30-protocol-support-in-azure-blob-storage"></a>Support du protocole NFS (Network File System) 3.0 dans le service Stockage Blob Azure

Le stockage Blob Azure prend désormais en charge le protocole NFS (Network File System) 3.0. Cette prise en charge assure la compatibilité du système de fichiers Linux au niveau du stockage des objets, et permet aux clients Linux de monter un conteneur dans Stockage Blob à partir d’une machine virtuelle Azure ou d’un ordinateur local. 

Il a toujours été difficile d’exécuter des charges de travail héritées à grande échelle, telles que HPC (High Performance Computing), dans le cloud. L’une des raisons est que les applications utilisent souvent des protocoles de fichiers traditionnels tels que NFS ou SMB (Server Message Block) pour accéder aux données. En outre, les services de stockage cloud natifs sont axés sur le stockage d’objets qui ont un espace de noms plat et des métadonnées étendues, plutôt que des systèmes de fichiers qui fournissent un espace de noms hiérarchique et des opérations de métadonnées efficaces. 

Le Stockage Blob prend désormais en charge un espace de noms hiérarchique et, lorsqu’il est associé à la prise en charge du protocole NFS 3.0, Azure facilite grandement l’exécution d’applications héritées par-dessus le stockage d’objets cloud à grande échelle. 

## <a name="applications-and-workloads-suited-for-this-feature"></a>Applications et charges de travail adaptées à cette fonctionnalité

La fonctionnalité de protocole NFS 3.0 convient davantage au traitement des charges de travail à haut débit, à grande échelle et à lecture intensive, telles que le traitement multimédia, les simulations de risques et le séquençage génomique. Vous devez envisager d’utiliser cette fonctionnalité pour tout autre type de charge de travail qui utilise plusieurs lecteurs et de nombreux threads, ce qui nécessite une bande passante élevée. 

## <a name="nfs-30-and-the-hierarchical-namespace"></a>NFS 3.0 et l’espace de noms hiérarchique

La prise en charge du protocole NFS 3.0 impose que les blobs soient organisés dans un espace de noms hiérarchique. Vous pouvez activer un espace de noms hiérarchique lorsque vous créez un compte de stockage. La possibilité d’utiliser un espace de noms hiérarchique a été introduite par Azure Data Lake Storage Gen2. Il organise les objets (fichiers) selon une hiérarchie de répertoires et sous-répertoires, de la même façon que le système de fichiers sur votre ordinateur.  L’espace de noms hiérarchique est mis à l’échelle de façon linéaire, et ne dégrade pas la capacité ou les performances des données. Différents protocoles s’étendent à partir de l’espace de noms hiérarchique. Le protocole NFS 3.0 est l’un de ces protocoles disponibles.   

> [!div class="mx-imgBorder"]
> ![espace de noms hiérarchique](./media/network-protocol-support/hierarchical-namespace-and-nfs-support.png)
  
## <a name="data-stored-as-block-blobs"></a>Données stockées en tant qu’objets blob de blocs

Si vous activez la prise en charge du protocole NFS 3.0, toutes les données de votre compte de stockage sont stockées en tant qu’objets blob de blocs. Les objets blob de blocs sont optimisés pour traiter efficacement de grandes quantités de données à lecture intensive. Les objets blob de blocs sont composés de blocs. Chaque bloc est identifié par un ID de bloc. Un objet blob de blocs peut inclure jusqu’à 50 000 blocs. Chaque bloc dans un objet blob de blocs peut avoir une taille différente, jusqu’à la taille maximale autorisée pour la version de service utilisée par votre compte.

Lorsque votre application effectue une requête à l’aide du protocole NFS 3.0, cette requête est traduite en une combinaison d’opérations d’objets blob de blocs. Par exemple, les requêtes NFS 3.0 de lecture RPC (Remote Procedure Call) sont traduites en opérations [Get Blob](/rest/api/storageservices/get-blob). Les requêtes d’écriture RPC NFS 3.0 sont traduites en une combinaison d’opérations [Get Block List](/rest/api/storageservices/get-block-list), [Put Block](/rest/api/storageservices/put-block) et [Put Block List](/rest/api/storageservices/put-block-list).

## <a name="general-workflow-mounting-a-storage-account-container"></a>Workflow général : Monter un conteneur de compte de stockage

Vos clients Linux peuvent monter un conteneur dans le Stockage Blob à partir d’une machine virtuelle Azure ou d’un ordinateur local. Pour monter un conteneur de compte de stockage, vous devez effectuer les opérations suivantes.

1. Créez un réseau virtuel Azure (Vnet).

2. Configurez la sécurité du réseau.

3. Créez et configurez un compte de stockage qui accepte le trafic uniquement à partir du réseau virtuel.

4. Créez un conteneur dans le compte de stockage.

5. Montez le conteneur.

Pour obtenir des instructions pas à pas, consultez [Monter le stockage Blob à l’aide du protocole NFS (Network File System) 3.0](network-file-system-protocol-support-how-to.md).

## <a name="network-security"></a>Sécurité du réseau

Le trafic doit provenir d’un réseau virtuel. Un réseau virtuel permet aux clients de se connecter en toute sécurité à votre compte de stockage. La seule façon de sécuriser les données dans votre compte consiste à utiliser un réseau virtuel et d’autres paramètres de sécurité réseau. Tous les autres outils utilisé pour sécuriser les données, notamment l’autorisation de clé de compte, la sécurité Azure Active Directory (AD) et les listes de contrôle d’accès (ACL), ne sont pas encore pris en charge dans les comptes sur lesquels la prise en charge du protocole NFS 3.0 est activée. 

Pour en savoir plus, consultez les [recommandations de sécurité réseau pour le stockage Blob](security-recommendations.md#networking).

### <a name="supported-network-connections"></a>Connexions réseau prises en charge

Un client peut se connecter via un point de terminaison public ou [un point de terminaison privé](../common/storage-private-endpoints.md) à partir de l’un des emplacements réseau suivants :

- Le réseau virtuel que vous configurez pour votre compte de stockage. 

  Dans cet article, nous ferons référence à ce réseau virtuel en tant que *réseau virtuel principal*. Pour en savoir plus, consultez [Autoriser l'accès à partir d'un réseau virtuel](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Un réseau virtuel associé qui se trouve dans la même région que le réseau virtuel principal.

  Vous devrez configurer votre compte de stockage pour autoriser l’accès à ce réseau virtuel associé. Pour en savoir plus, consultez [Autoriser l'accès à partir d'un réseau virtuel](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Un réseau local connecté à votre réseau virtuel principal à l’aide d'une [passerelle VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md) ou d’une [passerelle ExpressRoute](../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md). 

  Pour en savoir plus, consultez [Configuration de l’accès à partir de réseaux locaux](../common/storage-network-security.md#configuring-access-from-on-premises-networks).

- Un réseau local connecté à un réseau associé.

  Cette opération peut être effectuée avec une [passerelle VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md) ou une [passerelle ExpressRoute](../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md) ainsi qu’avec le [transit de la passerelle](/azure/architecture/reference-architectures/hybrid-networking/vnet-peering#gateway-transit). 

> [!IMPORTANT]
> Si vous vous connectez à partir d’un réseau local, assurez-vous que votre client autorise les communications sortantes via les ports 111 et 2048. Le protocole NFS 3.0 utilise ces ports.

<a id="azure-storage-features-not-yet-supported"></a>

## <a name="known-issues-and-limitations"></a>Problèmes connus et limitations

Consultez l’article [Problèmes connus](network-file-system-protocol-known-issues.md) pour obtenir une liste complète des problèmes et limitations avec la version actuelle du support NFS 3.0.

## <a name="pricing"></a>Tarifs

Consultez la page [Tarifs du stockage Blob Azure](https://azure.microsoft.com/pricing/details/storage/blobs/) pour connaitre le coût du stockage et de la transaction des données. 

## <a name="see-also"></a>Voir aussi

- [Monter le stockage Blob à l’aide du protocole NFS (Network File System) 3.0](network-file-system-protocol-support-how-to.md)
- [Considérations relatives au niveau de performance de NFS (Network File System) 3.0 dans le Stockage Blob Azure](network-file-system-protocol-support-performance.md)
- [Comparer l’accès avec NFS à Azure Files, Stockage Blob et NetApp Files Azure](../common/nfs-comparison.md)
