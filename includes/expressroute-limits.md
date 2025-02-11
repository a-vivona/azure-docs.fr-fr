---
title: Fichier Include
description: Fichier Include
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: include
ms.date: 01/12/2021
ms.author: duau
ms.custom: include file
ms.openlocfilehash: 4648693bc0dd7cebd8f8aec1cf878a8c3abfc49e
ms.sourcegitcommit: 0ede6bcb140fe805daa75d4b5bdd2c0ee040ef4d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2021
ms.locfileid: "122636439"
---
| Ressource | Limite |
| --- | --- |
| Circuits ExpressRoute par abonnement |50 |
| Circuits ExpressRoute par région par abonnement, avec Azure Resource Manager |10 |
| Nombre maximal d’itinéraires IPv4 publiés vers le peering privé Azure avec ExpressRoute Standard |4 000 |
| Nombre maximal d’itinéraires IPv4 publiés vers le peering privé Azure avec le module complémentaire ExpressRoute Premium |10 000 |
| Nombre maximal d’itinéraires IPv6 publiés vers le peering privé Azure avec ExpressRoute Standard |100 |
| Nombre maximal d’itinéraires IPv6 publiés vers le peering privé Azure avec le module complémentaire ExpressRoute Premium |100 |
| Nombre maximal d'itinéraires IPv4 publiés depuis le peering privé Azure à partir de l’espace d’adressage de réseau virtuel pour une connexion ExpressRoute | 1 000 |
| Nombre maximal d'itinéraires IPv6 publiés depuis le peering privé Azure à partir de l’espace d’adressage de réseau virtuel pour une connexion ExpressRoute | 1 000 |
| Nombre maximal d’itinéraires publiés vers le peering Microsoft avec ExpressRoute Standard |200 |
| Nombre maximal d’itinéraires publiés vers le peering Microsoft avec le module complémentaire ExpressRoute Premium |200 |
| Nombre maximal de circuits ExpressRoute liés au même réseau virtuel dans le même emplacement de peering |4 |
| Nombre maximal de circuits ExpressRoute liés au même réseau virtuel dans différents emplacements de peering |16 (Pour plus d'informations, consultez [Références SKU de passerelles](../articles/expressroute/expressroute-about-virtual-network-gateways.md#aggthroughput).) |
| Nombre de liens de réseau virtuel autorisés par circuit ExpressRoute |Consultez le tableau [Nombre de réseaux virtuels par circuit ExpressRoute](#vnetpercircuit).  |

#### <a name="number-of-virtual-networks-per-expressroute-circuit"></a><a name="vnetpercircuit"></a> Nombre de réseaux virtuels par circuit ExpressRoute
| **Taille du circuit** | **Nombre de liens de réseau virtuel pour la version standard** | **Nombre de liens de réseau virtuel avec le module complémentaire Premium** |
| --- | --- | --- |
| 50 Mbits/s |10 |20 |
| 100 Mbits/s |10 |25 |
| 200 Mbits/s |10 |25 |
| 500 Mbits/s |10 |40 |
| 1 Gbit/s |10 |50 |
| 2 Gbit/s |10 |60 |
| 5 Gbit/s |10 |75 |
| 10 Gbits/s |10 |100 |
| 40 Gbits/s* |10 |100 |
| 100 Gbits/s* |10 |100 |

**ExpressRoute Direct 100 Gbits/s uniquement*

> [!NOTE]
> Les connexions Global Reach sont décomptées par rapport à la limite de connexions de réseau virtuel par circuit ExpressRoute. Par exemple, un circuit Premium de 10 Gbits/s autorise 5 connexions Global Reach et 95 connexions aux passerelles ExpressRoute ou 95 connexions Global Reach et 5 connexions aux passerelles ExpressRoute, ou toute autre combinaison jusqu’à la limite de 100 connexions pour le circuit.
