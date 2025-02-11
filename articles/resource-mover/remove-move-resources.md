---
title: Supprimer des ressources d’une collection de déplacements dans Azure Resource Mover
description: Découvrez comment supprimer des ressources d’une collection de déplacements dans Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: how-to
ms.date: 02/22/2020
ms.author: raynew
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a655e8f0a4e4a6d44ce8960b45991cf6e394e2db
ms.sourcegitcommit: 30e3eaaa8852a2fe9c454c0dd1967d824e5d6f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/22/2021
ms.locfileid: "112454279"
---
# <a name="manage-move-collections-and-resource-groups"></a>Gérer les collections de déplacement et les groupes de ressources

Cet article explique comment supprimer des ressources d’une collection de déplacement ou supprimer une collection de déplacement/un groupe de ressources dans [Azure Resource Mover](overview.md). Les collections de déplacement sont utilisées lors du déplacement de ressources Azure entre régions Azure.

## <a name="remove-a-resource-portal"></a>Suppression d’une ressource (portail)

Vous pouvez supprimer des ressources dans une collection de déplacement, dans le portail du déplaceur de ressources, comme suit :

1. Dans **Entre régions**, sélectionnez toutes les ressources que vous souhaitez supprimer de la collection, puis sélectionnez **Supprimer**. 

    ![Bouton à sélectionner pour supprimer](./media/remove-move-resources/portal-select-resources.png)

2. Dans **Supprimer des ressources**, cliquez sur **Supprimer**.

    ![Bouton permettant de supprimer des ressources d’une collection de déplacements](./media/remove-move-resources/remove-portal.png)

## <a name="remove-a-move-collectionresource-group-portal"></a>Supprimer une collection de déplacement/un groupe de ressources (portail)

Vous pouvez supprimer une collection de déplacement/un groupe de ressources dans le portail.

1. Suivez les instructions de la procédure ci-dessus pour supprimer des ressources de la collection. Si vous supprimez un groupe de ressources, assurez-vous qu’il ne contient aucune ressource.
2. Supprimez la collection de déplacement ou le groupe de ressources.  

## <a name="remove-a-resource-powershell"></a>Supprimer une ressource (PowerShell)

À l’aide des applets de commande PowerShell, vous pouvez supprimer une ressource unique d’un MoveCollection, ou supprimer plusieurs ressources.

### <a name="remove-a-single-resource"></a>Sélectionner une ressource unique

Supprimez une ressource (dans notre exemple le réseau virtuel *psdemorm-vnet*) comme suit :

```azurepowershell-interactive
# Remove a resource using the resource ID
Remove-AzResourceMoverMoveResource -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -Name "psdemorm-vnet"
```
**Sortie après l’exécution de l’applet de commande**

![Texte de sortie après la suppression d’une ressource d’une collection de déplacement](./media/remove-move-resources/powershell-remove-single-resource.png)

### <a name="remove-multiple-resources"></a>Supprimer plusieurs ressources

Supprimez plusieurs ressources comme suit :

1. Validez les dépendances :

    ```azurepowershell-interactive
    $resp = Invoke-AzResourceMoverBulkRemove -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemorm-vnet') -ValidateOnly
    ```

    **Sortie après l’exécution de l’applet de commande**

    ![Texte de sortie après la suppression de plusieurs ressources d’une collection de déplacement](./media/remove-move-resources/remove-multiple-validate-dependencies.png)

2. Récupérez les ressources dépendantes qui doivent être supprimées (ainsi que notre exemple de réseau virtuel psdemorm-vnet) :

    ```azurepowershell-interactive
    $resp.AdditionalInfo[0].InfoMoveResource
    ```

    **Sortie après l’exécution de l’applet de commande**

    ![Texte de sortie après la récupération des ressources dépendantes qui doivent être supprimées](./media/remove-move-resources/remove-multiple-get-dependencies.png)


3. Supprimez toutes les ressources ainsi que le réseau virtuel :

    
    ```azurepowershell-interactive
    Invoke-AzResourceMoverBulkRemove -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('PSDemoVM','psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg')
    ```

    **Sortie après l’exécution de l’applet de commande**

    ![Texte de sortie après la suppression de toutes les ressources d’une collection de déplacement](./media/remove-move-resources/remove-multiple-all.png)


## <a name="remove-a-collection-powershell"></a>Supprimer une collection (PowerShell)

Supprimez une collection de déplacement complète de l’abonnement, comme suit :

1. Suivez les instructions ci-dessus pour supprimer des ressources de la collection à l’aide de PowerShell.
2. Supprimez une collection comme suit :

    ```azurepowershell-interactive
    Remove-AzResourceMoverMoveCollection -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"
    ```

    **Sortie après l’exécution de l’applet de commande**
    
    ![Texte de sortie après la suppression d’une collection de déplacement](./media/remove-move-resources/remove-collection.png)

## <a name="vm-resource-state-after-removing"></a>État des ressources de la machine virtuelle après suppression

Ce qui se passe lorsque vous supprimez une ressource de machine virtuelle d’une collection de déplacements dépend de l’état de la ressource, comme le tableau le résume.

###  <a name="remove-vm-state"></a>Supprimer l’état de la machine virtuelle
**État de la ressource** | **Machine virtuelle** | **Mise en réseau**
--- | --- | --- 
**Ajouté à la collection de déplacement** | Supprimer de la collection de déplacement. | Supprimer de la collection de déplacement. 
**Dépendances résolues/préparation en attente** | Supprimer de la collection de déplacement  | Supprimer de la collection de déplacement. 
**Préparation en cours**<br/> (ou tout autre état en cours) | L’opération de suppression échoue avec une erreur.  | L’opération de suppression échoue avec une erreur.
**Échec de la préparation** | Supprimez de la collection de déplacement.<br/>Supprimez tout ce qui a été créé dans la région cible, y compris les disques de réplication. <br/><br/> Les ressources d’infrastructure créées pendant le déplacement doivent être supprimées manuellement. | Supprimez de la collection de déplacement.  
**Lancement du déplacement en attente** | Supprimer de la collection de déplacement.<br/><br/> Supprimez tout ce qui a été créé dans la région cible, y compris les machines virtuelles, les disques de réplication, etc.  <br/><br/> Les ressources d’infrastructure créées pendant le déplacement doivent être supprimées manuellement. | Supprimer de la collection de déplacement.
**Impossible de lancer le déplacement** | Supprimer de la collection de déplacement.<br/><br/> Supprimez tout ce qui a été créé dans la région cible, y compris les machines virtuelles, les disques de réplication, etc.  <br/><br/> Les ressources d’infrastructure créées pendant le déplacement doivent être supprimées manuellement. | Supprimer de la collection de déplacement.
**Validation en attente** | Nous vous recommandons d’ignorer le déplacement pour que les ressources cibles soient supprimées en premier.<br/><br/> La ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là. | Nous vous recommandons d’ignorer le déplacement pour que les ressources cibles soient supprimées en premier.<br/><br/> La ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là. 
**Échec de la validation** | Nous vous recommandons d’ignorer le déplacement afin que les ressources cibles soient supprimées en premier.<br/><br/> La ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là. | Nous vous recommandons d’ignorer le déplacement pour que les ressources cibles soient supprimées en premier.<br/><br/> La ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là.
**Abandon terminé** | La ressource revient à l’état **Lancement du déplacement en attente**.<br/><br/> Elle est supprimée de la collection de déplacement, ainsi que tout ce qui a été créé sur la cible : les disques de réplication, le coffre, etc.  <br/><br/> Les ressources d’infrastructure créées pendant le déplacement doivent être supprimées manuellement. <br/><br/> Les ressources d’infrastructure créées pendant le déplacement doivent être supprimées manuellement. |  La ressource revient à l’état **Lancement du déplacement en attente**.<br/><br/> Elle est supprimée de la collection de déplacements.
**Échec de l’abandon** | Nous vous recommandons d’ignorer les déplacements afin que les ressources cibles soient supprimées en premier.<br/><br/> Après cela, la ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là. | Nous vous recommandons d’ignorer les déplacements afin que les ressources cibles soient supprimées en premier.<br/><br/> Après cela, la ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là.
**Suppression de la source en attente** | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région cible.  | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région cible.
**Échec de la suppression de la source** | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région cible. | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région cible.
**Déplacement terminé** | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région source ou cible. |  Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région source ou cible.

## <a name="sql-resource-state-after-removing"></a>État de la ressource SQL après suppression

Ce qui se passe lorsque vous supprimez une ressource Azure SQL d’une collection de déplacements dépend de l’état de la ressource, comme le tableau le résume.

**État de la ressource** | **SQL** 
--- | --- 
**Ajouté à la collection de déplacement** | Supprimer de la collection de déplacement. 
**Dépendances résolues/préparation en attente** | Supprimer de la collection de déplacement 
**Préparation en cours**<br/> (ou tout autre état en cours)  | L’opération de suppression échoue avec une erreur. 
**Échec de la préparation** | Supprimer de la collection de déplacement<br/><br/>Ne supprimez rien de ce qui a été créé dans la région cible. 
**Lancement du déplacement en attente** |  Supprimer de la collection de déplacement<br/><br/>Ne supprimez rien de ce qui a été créé dans la région cible. La base de données SQL existe à ce stade et sera supprimée. 
**Impossible de lancer le déplacement** | Supprimer de la collection de déplacement<br/><br/>Ne supprimez rien de ce qui a été créé dans la région cible. La base de données SQL existe à ce stade et doit être supprimée. 
**Validation en attente** | Nous vous recommandons d’ignorer le déplacement pour que les ressources cibles soient supprimées en premier.<br/><br/> La ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là.
**Échec de la validation** | Nous vous recommandons d’ignorer le déplacement pour que les ressources cibles soient supprimées en premier.<br/><br/> La ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là. 
**Abandon terminé** |  La ressource revient à l’état **Lancement du déplacement en attente**.<br/><br/> Elle est supprimée de la collection de déplacement, ainsi que tout ce qui a été créé sur la cible, y compris les bases de données SQL. 
**Échec de l’abandon** | Nous vous recommandons d’ignorer les déplacements afin que les ressources cibles soient supprimées en premier.<br/><br/> Après cela, la ressource revient à l’état **Lancement du déplacement en attente** et vous pouvez continuer de là. 
**Suppression de la source en attente** | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région cible. 
**Échec de la suppression de la source** | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région cible. 
**Déplacement terminé** | Supprimé de la collection de déplacements.<br/><br/> Ne supprime rien de ce qui a été créé dans la région source ou cible.

## <a name="next-steps"></a>Étapes suivantes

Essayez de [déplacer une machine virtuelle](tutorial-move-region-virtual-machines.md) vers une autre région avec Resource Mover.
