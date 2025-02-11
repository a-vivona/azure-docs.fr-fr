---
title: Réplication inter-région des volumes Azure NetApp Files | Microsoft Docs
description: Décrit le fonctionnement de la réplication inter-région, les paires de régions prises en charge, les objectifs de niveau de service, la durabilité des données et le modèle de coût.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/16/2021
ms.author: b-juche
ms.custom: references_regions
ms.openlocfilehash: 625df6ed5147b77a9829b6e7b3396f4855068d62
ms.sourcegitcommit: 05dd6452632e00645ec0716a5943c7ac6c9bec7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122534869"
---
# <a name="cross-region-replication-of-azure-netapp-files-volumes"></a>Réplication inter-région des volumes Azure NetApp Files

La fonctionnalité de réplication Azure NetApp Files fournit une protection des données via la réplication de volume inter-région. Vous pouvez répliquer de façon asynchrone des données à partir d’un volume Azure NetApp Files (source) d’une région vers un autre volume Azure NetApp Files (destination) d’une autre région.  Cette fonctionnalité vous permet de basculer votre application critique en cas de panne ou d’incident touchant l’ensemble de la région.

> [!IMPORTANT]
> La fonctionnalité de réplication inter-région est actuellement disponible en préversion. Vous devez soumettre une demande d’inscription à la liste d’attente pour accéder à la fonctionnalité via la [page de soumission de demande d’inscription à la liste d’attente de réplication inter-région Azure NetApp Files](https://aka.ms/anfcrrpreviewsignup). Attendez de recevoir un e-mail de confirmation officiel de l’équipe Azure NetApp Files avant d’utiliser la fonctionnalité de réplication inter-région.

## <a name="supported-cross-region-replication-pairs"></a><a name="supported-region-pairs"></a>Paires de réplication interrégion prises en charge

La réplication de volume Azure NetApp Files est prise en charge entre différentes [paires de régions Azure](../best-practices-availability-paired-regions.md#azure-regional-pairs) et des régions non appairées. La réplication de volume Azure NetApp Files est actuellement disponible entre les régions suivantes :  

### <a name="azure-regional-pairs"></a>Paires régionales Azure

* USA Est et USA Ouest
* USA Est 2 et USA Centre
* Australie Est et Australie Sud-Est
* Canada Centre et Canada Est
* Inde Sud et Inde Centre 
* Allemagne Centre-Ouest et Allemagne Nord
* Japon Est et Japon Ouest
* Europe Nord et Europe Ouest
* Royaume-Uni Sud et Royaume-Uni Ouest
* Émirats arabes unis Nord, Émirats arabes unis Centre
* Norvège Est et Norvège Ouest

### <a name="azure-regional-non-standard-pairs"></a>Paires non standard de régions Azure

*   USA Ouest 2 et USA Est
*   USA Centre Sud et USA Centre
*   USA Centre Sud et USA Est
*   USA Centre Sud et USA Est 2
*   USA Est et USA Est 2
*   USA Est 2 et USA Est 2
*   Australie Est et Asie Sud-Est 
*   Allemagne Centre-Ouest et Royaume-Uni Sud
*   Allemagne Centre-Ouest et Europe Ouest
*   Allemagne Centre-Ouest et France Centre

## <a name="service-level-objectives"></a>Objectifs de niveau de service

L’objectif de point de récupération (RPO) indique le moment où les données peuvent être récupérées. L’objectif de RPO est généralement inférieur à deux fois la planification de la réplication, mais peut varier. Dans certains cas, il peut dépasser le RPO cible en fonction de facteurs tels que la taille totale du jeu de données, le taux de modification, le pourcentage de remplacements de données et la bande passante de réplication disponible pour le transfert.   

* Pour une planification de réplication de 10 minutes, le RPO maximal est de 20 minutes.  
* Pour le calendrier de réplication horaire, le RPO typique est inférieur à deux heures.  
* Pour le calendrier de réplication quotidien, le RPO typique est inférieur à deux jours.  

L’objectif de délai de récupération (RTO), ou le temps d’arrêt maximal des applications professionnelles, est déterminé par des facteurs d’affichage de l’application et d’accès aux données sur le deuxième site. La partie de stockage du RTO pour rompre la relation d’homologation afin d’activer le volume de destination et de fournir l’accès aux données, en lecture et en écriture, dans le deuxième site, est censée durer une minute maximum.

## <a name="cost-model-for-cross-region-replication"></a>Modèle de coût pour la réplication inter-région  

Grâce à la réplication inter-région Azure NetApp Files, vous payez uniquement pour la quantité de données que vous répliquez. Il n’y a pas de frais de configuration ou d’utilisation minimale. Le prix de réplication est basé sur la fréquence de réplication et la région du volume de *destination* que vous choisissez pendant la configuration de la réplication initiale. Pour plus d’informations, consultez la page [Tarification Azure NetApp Files](https://azure.microsoft.com/pricing/details/netapp/).  

Les frais de capacité de stockage standard Azure NetApp Files s’appliquent au volume de destination de réplication (également appelé volume de *protection des données*). 

### <a name="pricing-examples"></a>Exemples de tarification

Le montant de réplication inter-région facturé par mois est basé sur la quantité de données répliquées via la fonctionnalité de réplication inter-région au cours de ce mois. La quantité de données répliquées est mesurée en Gio. Elle représente la somme des données répliquées entre deux régions pendant toutes les réplications régulières à partir des volumes sources vers les volumes de destination, et pendant toutes les réplications de resynchronisation des volumes de destination vers les volumes sources.

#### <a name="example-1-month-1-baseline-replication-and-incremental-replications"></a>Exemple 1 : Réplication de base et réplications incrémentielles du premier mois

Imaginons les situations suivantes :

* Votre volume *source* provient du niveau de service Azure NetApp Files *Premium*. Il a une taille de quota de volume de 1000 Gio et une taille de volume consommé de 500 Gio au début du premier jour du mois. Le volume se trouve dans la région *USA Centre Sud*.
* Votre volume de *destination* provient du niveau de service Azure NetApp Files *Standard*. Il se trouve dans la région *USA Est 2*.
* Vous avez configuré une réplication inter-région *horaire* entre les deux volumes ci-dessus. Par conséquent, le prix de la réplication est de 0,12 $ par Gio.
* Pour simplifier, supposons que votre volume source ait un changement de données constant de 0,5 Gio toutes les heures, mais que la taille du volume total consommé n'augmente pas (elle reste à 500 Gio). 

Après la configuration initiale, la réplication de base se produit immédiatement.  

* Quantité de données répliquées pendant la réplication de base : `500 GiB`
* Frais de réplication de base : `500 GiB * $0.12 = $60`

Après la réplication de base, seuls les blocs modifiés sont répliqués. Par conséquent, seules 0,5 Gio de données sont répliquées toutes les heures dans les réplications incrémentielles suivantes.

* Somme de la quantité de données répliquées sur les réplications incrémentielles pour un mois de 30 jours : `0.5 GiB * 24 hours * 30 days = 360 GiB`
* Frais de réplication incrémentielle : `360 GiB * $0.12 = $43.2`

À la fin du premier mois, le coût total de réplication inter-région est le suivant :  

*  Coût total de réplication inter-région du premier mois : `$60 + $43.2 = $103.2`

Les frais de capacité de stockage standard Azure NetApp Files s’appliquent au volume de destination. Toutefois, le volume de destination peut utiliser un niveau de stockage différent de la couche de volume source (et qui coûte moins cher).

#### <a name="example-2-month-2-incremental-replications-and-resync-replications"></a>Exemple 2 : Réplications incrémentielles et réplication de resynchronisation du deuxième mois  

Supposons que vous ayez un volume source, un volume de destination et une relation de réplication entre les deux configurés comme décrit dans l’exemple 1. Pour 29 jours du deuxième mois (un mois de 30 jours), les réplications horaires se sont produites comme prévu.

* Somme de la quantité de données répliquées sur les réplications incrémentielles pendant 29 jours : `0.5 GiB * 24 hours * 29 days = 348 GiB`

Supposons que le dernier jour du mois, un événement imprévu, tel qu’une panne, se produise dans la région source : vous basculez donc vers le volume de destination. Au bout de 2 heures, la région source est rétablie et vous avez effectué une réplication de resynchronisation du volume de destination vers le volume source. Pendant ces 2 heures, 0,8 Gio de données ont été modifiées sur le volume de destination. Il faut donc les resynchroniser avec la source.

* Somme de la quantité de données répliquées sur des réplications régulières pendant 22 heures le dernier jour : `0.5 GiB * 22 hours = 11 GiB`
* Quantité de données répliquées pendant une réplication de resynchronisation : `0.8 GiB`

Par conséquent, à la fin du deuxième mois, le coût total de réplication inter-région est le suivant :  

* Coût total de réplication inter-région du deuxième mois : `(348 GiB + 11 GiB + 0.8 GiB) * $0.12 = $43.18`

Les frais de capacité de stockage Azure NetApp Files normaux pour le deuxième s’appliquent au volume de destination.

## <a name="next-steps"></a>Étapes suivantes
* [Configuration requise et considérations pour la réplication inter-région](cross-region-replication-requirements-considerations.md)
* [Créer une réplication de volume](cross-region-replication-create-peering.md)
* [Afficher l’état d’intégrité de la relation de réplication](cross-region-replication-display-health-status.md)
* [Gérer la reprise d’activité après sinistre](cross-region-replication-manage-disaster-recovery.md)
* [Redimensionner un volume de destination de réplication entre régions](azure-netapp-files-resize-capacity-pools-or-volumes.md#resize-a-cross-region-replication-destination-volume)
* [Métriques de réplication de volume](azure-netapp-files-metrics.md#replication)
* [Supprimer des volumes ou des réplications de volume](cross-region-replication-delete.md)
* [Résoudre les problèmes de réplication inter-région](troubleshoot-cross-region-replication.md)
