---
title: Migration de Network Performance Monitor vers Moniteur de connexion
titleSuffix: Azure Network Watcher
description: Découvrez comment migrer de Network Performance Monitor vers Moniteur de connexion.
services: network-watcher
documentationcenter: na
author: vinynigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: vinigam
ms.openlocfilehash: 0ec16b16c8e71d764fb0fe21520eb407493ed8d7
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/30/2021
ms.locfileid: "113105360"
---
# <a name="migrate-to-connection-monitor-from-network-performance-monitor"></a>Migration de Network Performance Monitor vers Moniteur de connexion

> [!IMPORTANT]
> À compter du 1er juillet 2021, vous ne pourrez plus ajouter de tests dans un espace de travail existant ou activer un nouvel espace de travail avec Network Performance Monitor. Vous pouvez continuer à utiliser les tests créés avant le 1er juillet 2021. Pour réduire l’interruption de service de vos charges de travail actuelles, migrez vos tests de Network Performance Monitor vers le nouveau moniteur de connexion dans Azure Network Watcher avant le 29 février 2024.

En un clic et sans temps d’arrêt, vous pouvez migrer des tests de la solution Network Performance Monitor (NPM) vers la nouvelle fonctionnalité Moniteur de connexion améliorée. Pour en savoir plus sur les avantages, consultez [Moniteur de connexion](./connection-monitor-overview.md).


## <a name="key-points-to-note"></a>Points clés à noter

La migration produit les résultats suivants :

* Les agents locaux et les paramètres de pare-feu fonctionnent tels quels. Aucune modification n’est nécessaire. Les agents Log Analytics installés sur des machines virtuelles Azure doivent être remplacés par l’[extension Network Watcher](../virtual-machines/extensions/network-watcher-windows.md).
* Les tests existants sont mappés à Moniteur de connexion > Groupe de test > Format de test. En sélectionnant **Modifier**, vous pouvez afficher et modifier les propriétés du nouveau moniteur de connexion, télécharger un modèle pour y apporter des modifications et soumettre le modèle via Azure Resource Manager.
* Les agents envoient des données à l’espace de travail et aux métriques Log Analytics.
* Analyse des données :
   * **Données dans Log Analytics** : avant la migration, les données restent dans l’espace de travail dans lequel NPM est configuré dans la table NetworkMonitoring. Après la migration, les données sont placées dans les tables NetworkMonitoring, NWConnectionMonitorTestResult et NWConnectionMonitorPathResult, dans le même espace de travail. Une fois les tests désactivés dans NPM, les données sont uniquement stockées dans les tables NWConnectionMonitorTestResult et NWConnectionMonitorPathResult.
   * **Alertes, tableaux de bord et intégrations basés sur un journal** : vous devez modifier manuellement les requêtes en fonction des nouvelles tables NWConnectionMonitorTestResult et NWConnectionMonitorPathResult. Pour recréer les alertes dans les métriques, consultez [Surveillance de la connectivité réseau à l’aide de Moniteur de connexion](./connection-monitor-overview.md#metrics-in-azure-monitor).
* Pour la surveillance d’ExpressRoute :
    * **Perte et latence de bout en bout** : Moniteur de connexion s’en chargera, et cela sera plus facile que NPM, car les utilisateurs n’ont pas besoin de configurer les circuits et les Peerings à surveiller. Les circuits dans le chemin d’accès seront détectés automatiquement, les données seront disponibles dans les métriques (plus rapide que LA, qui était l’endroit où NPM stockait les résultats). La topologie fonctionnera également telle quelle.
    * **Mesures de la bande passante** : Avec le lancement des mesures liées à la bande passante, l’approche de NPM basée sur l’analytique des journaux d’activité n’était pas efficace pour la surveillance de la bande passante pour les clients d’ExpressRoute. Cette capacité est désormais disponible dans Moniteur de connexion.
    
## <a name="prerequisites"></a>Prérequis

* Assurez-vous que Network Watcher est activé dans votre abonnement et la région de l’espace de travail Log Analytics. Si vous ne l’avez pas fait, vous verrez une erreur indiquant « Avant de tenter une migration, activez l’extension Network Watcher dans l’abonnement de la sélection et l’emplacement de l’espace de travail LA sélectionné. »
* Si vous utilisez en tant que point de terminaison une machine virtuelle Azure appartenant à une autre région ou à un autre abonnement que l’espace de travail de Log Analytics, assurez-vous que Network Watcher est activé pour cet abonnement et cette région.   
* Les machines virtuelles Azure sur lesquelles les agents Log Analytics sont installés doivent être activées avec l’extension Network Watcher.

## <a name="migrate-the-tests"></a>Migrer les tests

Pour migrer les tests de Network Performance Monitor vers Moniteur de connexion, procédez comme suit :

1. Dans Network Watcher, sélectionnez **Moniteur de connexion**, puis sélectionnez l’onglet **Importer des tests de NPM**. 

    :::image type="content" source="./media/connection-monitor-2-preview/migrate-npm-to-cm-preview.png" alt-text="Migrer des tests de Network Performance Monitor vers le Moniteur de connexion" lightbox="./media/connection-monitor-2-preview/migrate-npm-to-cm-preview.png":::
    
1. Dans les listes déroulantes, sélectionnez votre abonnement et votre espace de travail, puis la fonctionnalité NPM que vous souhaitez migrer. 
1. Sélectionnez **Importer** pour migrer les tests.
* Si NPM n’est pas activé sur l’espace de travail, vous verrez une erreur indiquant « Aucune configuration NPM valide n’a été trouvée ». 
* Si aucun test n’existe dans la fonctionnalité que vous avez choisie à l’étape 2, vous verrez une erreur indiquant « L’espace de travail sélectionné n’a pas de configuration <feature> ».
* S’il n’y a pas de tests valides, vous verrez une erreur indiquant « L’espace de travail sélectionné n’a pas de tests valides »
* Vos tests peuvent contenir des agents qui ne sont plus actifs, mais qui ont peut-être été actifs dans le passé. Une erreur s’affiche indiquant que « Certains tests contiennent des agents qui ne sont plus actifs. Liste des agents inactifs : {0}. Ces agents ont peut-être été exécutés par le passé, mais ils sont arrêtés/ne sont plus en cours d’exécution. Activez les agents et migrez vers le moniteur de connexion. Cliquez sur Continuer pour migrer les tests qui ne contiennent pas d’agents qui ne sont pas actifs. »

Après le début de la migration, les modifications suivantes ont lieu : 
* Une nouvelle ressource de moniteur de connexion est créée.
   * Une seule analyse de connexion par région et un abonnement est créé. Pour les tests avec des agents locaux, le nouveau nom du moniteur de connexion est au format `<workspaceName>_"workspace_region_name"`. Pour les tests avec des agents Azure, le nouveau nom du moniteur de connexion est au format `<workspaceName>_<Azure_region_name>`.
   * Les données de surveillance sont désormais stockées dans le même espace de travail Log Analytics dans lequel NPM est activé, dans de nouvelles tables appelées NWConnectionMonitorTestResult et NWConnectionMonitorPathResult. 
   * Le nom du test est transféré en tant que nom du groupe de test. La description du test n’est pas migrée.
   * Les points de terminaison source et de destination sont créés et utilisés dans le groupe de test. Pour les agents locaux, les points de terminaison respectent le format `<workspaceName>_<FQDN of on-premises machine>`. La description de l’agent n’est pas migrée.
   * Le port de destination et l’intervalle de détection sont déplacés vers une configuration de test, sous les noms `TC_<protocol>_<port>` et `TC_<protocol>_<port>_AppThresholds`. Le protocole est défini sur la base des valeurs de port. Pour ICMP, les configurations de test sont nommées `TC_<protocol>` et `TC_<protocol>_AppThresholds` . Seuils de réussite et autres propriétés facultatives si le jeu est migré ; sinon, ces champs sont laissés vides.
   * Si les tests de migration contiennent des agents qui ne sont pas en cours d’exécution, vous devez activer les agents et les migrer à nouveau.
* Comme NPM n’est pas désactivé, les tests migrés peuvent continuer à envoyer des données aux tables NetworkMonitoring, NWConnectionMonitorTestResult et NWConnectionMonitorPathResult. Cette approche garantit que les alertes et intégrations basées sur un journal existantes ne sont pas affectées.
* Le moniteur de connexion nouvellement créé est visible dans Moniteur de connexion.

Après la migration, veillez à effectuer les opérations suivantes :
* Désactivez manuellement les tests dans NPM. Tant que vous ne le faites pas, vous continuerez d’être facturé pour ceux-ci. 
* Lorsque vous désactivez NPM, recréez vos alertes sur les tables NWConnectionMonitorTestResult et NWConnectionMonitorPathResult, ou utilisez des métriques. 
* Migrez les intégrations externes vers les tables NWConnectionMonitorTestResult et NWConnectionMonitorPathResult. Les intégrations externes sont, par exemple, les tableaux de bord dans Power BI et Grafana, ainsi que les intégrations avec les systèmes Security Information and Event Management (SIEM).

## <a name="common-errors-encountered"></a>Erreurs courantes rencontrées

Voici quelques-unes des erreurs courantes rencontrées lors de la migration : 

| Error  |    Motif   |
|---|---|
| Aucune configuration NPM valide n’a été trouvée. Accédez à l’interface utilisateur de NPM pour vérifier la configuration     |     Cette erreur se produit lorsque l’utilisateur sélectionne Importer des tests de NPM pour migrer les tests, mais que NPM n’est pas activé dans l’espace de travail   |
|L'espace de travail sélectionné n'a pas de configuration « Moniteur de connectivité de service »    |       Cette erreur se produit lorsque l’utilisateur effectue une migration des tests du moniteur de connectivité de service de NPM vers le Moniteur de connexion, mais qu’aucun test n’est configuré dans le moniteur de connectivité de service |
|L'espace de travail sélectionné n'a pas de configuration « Moniteur ExpressRoute »    |     Cette erreur se produit lorsque l’utilisateur migre des tests du moniteur ExpressRoute de NPM vers le Moniteur de connexion, mais qu’aucun test n’est configuré dans ExpressRoute Monitor  |
|L'espace de travail sélectionné n'a pas de configuration « Moniteur de performances »    |      Cette erreur se produit lorsque l’utilisateur migre des tests de l’analyseur de performances de NPM vers le Moniteur de connexion, mais qu’aucun test n’est configuré dans l’analyseur de performances |
|L’espace de travail sélectionné n’a pas de test « {0} » valide    |      Cette erreur se produit lorsque l’utilisateur effectue une migration des tests de NPM vers le Moniteur de connexion, mais qu’il n’existe aucun test valide dans la fonctionnalité choisie par l’utilisateur pour la migration  |
|Avant de tenter une migration, activez l'extension Network Watcher dans l'abonnement de la sélection et l'emplacement de l'espace de travail LA sélectionné      |      Cette erreur se produit lorsque l’utilisateur effectue une migration des tests de NPM vers le Moniteur de connexion et que l’extension de Network Watcher n’est pas activée dans l’espace de travail LA sélectionné. L’utilisateur doit activer l’extension NW avant de migrer des tests |
|Certains tests {1} contiennent des agents qui ne sont plus actifs. Liste des agents inactifs : {0}. Ces agents ont peut-être été exécutés par le passé, mais ils sont arrêtés/ne sont plus en cours d’exécution. Activez les agents et migrez vers le moniteur de connexion. Cliquez sur Continuer pour migrer les tests qui ne contiennent pas d’agents qui ne sont pas actifs       |    Cette erreur se produit lorsque l’utilisateur effectue une migration des tests de NPM vers le Moniteur de connexion et que certains tests sélectionnés contiennent des agents Network Watcher inactifs ou des agents NW qui ne sont plus actifs, mais qui ont été utilisés par le passé et qui ont été arrêtés. L’utilisateur peut désélectionner ces tests et continuer à sélectionner et à migrer les tests qui ne contiennent pas de tels agents inactifs  |
|Vos tests {1} contiennent des agents qui ne sont plus actifs. Liste des agents inactifs : {0}. Ces agents ont peut-être été exécutés par le passé, mais ils sont arrêtés/ne sont plus en cours d’exécution. Activez les agents et migrez vers le Moniteur de connexion     | Cette erreur se produit lorsque l’utilisateur effectue une migration des tests de NPM vers le Moniteur de connexion et que des tests sélectionnés contiennent des agents Network Watcher inactifs ou des agents NW qui ne sont plus actifs, mais qui ont été utilisés par le passé et qui ont été arrêtés. L’utilisateur doit activer les agents, puis continuer à migrer ces tests vers le Moniteur de connexion    |
|Une erreur s’est produite lors de l’importation des tests dans le Moniteur de connexion     |    Cette erreur se produit lorsque l’utilisateur tente de migrer des tests de NPM vers CM, mais en raison d’erreurs, la migration échoue |



## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le Moniteur de connexion, consultez :
* [Migrer vers le Moniteur de connexion à partir du Moniteur de connexion (classique)](./migrate-to-connection-monitor-from-connection-monitor-classic.md)
* [Créer un Moniteur de connexion à l’aide du portail Azure](./connection-monitor-create-using-portal.md)
