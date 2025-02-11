---
title: Meilleures pratiques pour l’hébergement d’une application cliente Kubernetes
titleSuffix: Azure Cache for Redis
description: Découvrez comment héberger une application cliente Kubernetes qui utilise Azure Cache pour Redis.
author: shpathak-msft
ms.service: cache
ms.topic: conceptual
ms.date: 08/25/2021
ms.author: shpathak
ms.openlocfilehash: 27117f8839c6ea0d98c2f15e01ee6a3ab3ec36c6
ms.sourcegitcommit: dcf1defb393104f8afc6b707fc748e0ff4c81830
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/27/2021
ms.locfileid: "123116277"
---
# <a name="kubernetes-hosted-client-application"></a>Application cliente hébergée par Kubernetes

## <a name="multiple-pods"></a>Plusieurs pods

Lorsque vous avez plusieurs pods se connectant à un serveur Redis, assurez-vous que les nouvelles connexions en provenance des pods sont créées de manière échelonnée. Si plusieurs pods démarrent en peu de temps sans échelonnement, cela provoque un pic soudain du nombre de connexions client créées. Le nombre élevé de connexions entraîne une charge élevée sur le serveur Redis et peut entraîner des délais d’attente.

Évitez le même scénario lors de l’arrêt de plusieurs pods en même temps. Un échec d’échelonnement de l’arrêt peut entraîner une chute abrupte du nombre de connexions conduisant à une sollicitation du processeur.

## <a name="sufficient-resources"></a>Ressources suffisantes

Assurez-vous que le nœud Kubernetes qui héberge le pod se connectant au serveur Redis dispose de suffisamment de mémoire, de processeur et de bande passante réseau.  

## <a name="noisy-neighbor-problem"></a>Problème de voisin bruyant

Attention au problème de *voisin bruyant*. Un pod exécutant le client peut être affecté par d’autres pods en cours d’exécution sur le même nœud, et limiter les connexions Redis ou les opérations d’E/S.
