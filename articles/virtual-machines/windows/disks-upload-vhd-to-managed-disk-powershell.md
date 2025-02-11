---
title: Charger un disque dur virtuel sur Azure ou copier un disque d’une région à l’autre - Azure PowerShell
description: Découvrez comment charger un disque dur virtuel sur un disque managé Azure et comment copier un disque managé d’une région à l’autre avec Azure PowerShell via le chargement direct.
author: roygara
ms.author: rogarana
ms.date: 06/29/2021
ms.topic: how-to
ms.service: storage
ms.tgt_pltfrm: linux
ms.subservice: disks
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9bec6aaf34691adf03dad9204b93af006008c7a0
ms.sourcegitcommit: 82d82642daa5c452a39c3b3d57cd849c06df21b0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2021
ms.locfileid: "113356151"
---
# <a name="upload-a-vhd-to-azure-or-copy-a-managed-disk-to-another-region---azure-powershell"></a>Charger un disque dur virtuel sur Azure ou copier un disque managé dans une autre région - Azure PowerShell

[!INCLUDE [disks-upload-vhd-to-disk-intro](../../../includes/disks-upload-vhd-to-disk-intro.md)]

## <a name="prerequisites"></a>Prérequis

- Téléchargez la dernière [version d’AzCopy v10](../../storage/common/storage-use-azcopy-v10.md#download-and-install-azcopy).
- [Installez le module Azure PowerShell](/powershell/azure/install-Az-ps).
- Si vous envisagez de charger un VHD à partir d’un emplacement local : Un VHD de taille fixe qui a été [préparé pour Azure](prepare-for-upload-vhd-image.md) et stocké localement.
- Ou un disque managé dans Azure, si vous envisagez d’effectuer une action de copie.

## <a name="getting-started"></a>Prise en main

Si vous préférez charger des disques via une interface graphique utilisateur, vous pouvez utiliser l’Explorateur Stockage Azure. Pour plus d’informations, consultez : [Utiliser l’Explorateur Stockage Azure pour gérer des disques managés Azure](../disks-use-storage-explorer-managed-disks.md)

Pour charger votre disque dur virtuel sur Azure, vous devez créer un disque managé vide configuré pour ce processus de chargement. Avant de créer ce disque, vous devez prendre connaissance des informations ci-dessous.

Ce type de disque managé présente deux états uniques :

- ReadyToUpload, qui signifie que le disque est prêt à recevoir un chargement, mais qu’aucune [signature d’accès partagé](../../storage/common/storage-sas-overview.md) (SAP) n’a été générée.
- ActiveUpload, qui signifie que le disque est prêt à recevoir un chargement et que la SAS a été générée.

> [!NOTE]
> Dans l'un ou l'autre de ces états, le disque managé est facturé au [tarif HDD Standard](https://azure.microsoft.com/pricing/details/managed-disks/), quel que soit le type de disque. Par exemple, un P10 est facturé comme un S10. Ceci s’applique jusqu’à ce que `revoke-access` soit appelé sur le disque managé, ce qui est nécessaire pour joindre le disque à une machine virtuelle.

## <a name="create-an-empty-managed-disk"></a>Créer un disque managé vierge

Avant de pouvoir créer un disque dur standard vide pour le chargement, vous devez connaître la taille de fichier (en octets) du disque dur virtuel que vous voulez charger. L'exemple de code vous la fournira, mais si vous souhaitez effectuer cette opération vous-même, vous pouvez utiliser : `$vhdSizeBytes = (Get-Item "<fullFilePathHere>").length`. Cette valeur est utilisée lors de la spécification du paramètre **-UploadSizeInBytes**.

Ensuite, à partir de votre interpréteur de commandes local, créez un disque HDD Standard vierge pour le chargement en spécifiant le paramètre **Upload** dans **-CreateOption** ainsi que le paramètre **-UploadSizeInBytes** dans la cmdlet [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig). Ensuite, appelez [New-AzDisk](/powershell/module/az.compute/new-azdisk) pour créer le disque.

Remplacez `<yourdiskname>`, `<yourresourcegroupname>`et `<yourregion>`, puis exécutez les commandes suivantes :

> [!TIP]
> Si vous créez un disque de système d’exploitation, ajoutez `-HyperVGeneration '<yourGeneration>'` à `New-AzDiskConfig`.

```powershell
$vhdSizeBytes = (Get-Item "<fullFilePathHere>").length

$diskconfig = New-AzDiskConfig -SkuName 'Standard_LRS' -OsType 'Windows' -UploadSizeInBytes $vhdSizeBytes -Location '<yourregion>' -CreateOption 'Upload'

New-AzDisk -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>' -Disk $diskconfig
```

Si vous souhaitez charger un disque SSD Premium ou Standard, remplacez **Standard_LRS** par **Premium_LRS** ou par **StandardSSD_LRS**. Les disques Ultra ne sont pas encore pris en charge.

Maintenant que vous avez créé un disque managé vide configuré pour le processus de chargement, vous pouvez y charger un disque dur virtuel. Pour charger un disque dur virtuel sur le disque, il vous faut une signature d’accès partagé accessible en écriture afin de pouvoir la référencer en tant que destination pour votre chargement.

Pour générer une signature d’accès partagé accessible en écriture de votre disque managé vide, remplacez `<yourdiskname>` et `<yourresourcegroupname>`, puis utilisez les commandes suivantes :

```powershell
$diskSas = Grant-AzDiskAccess -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>' -DurationInSecond 86400 -Access 'Write'

$disk = Get-AzDisk -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>'
```

## <a name="upload-a-vhd"></a>Télécharger un disque dur virtuel

Maintenant que vous disposez d'une SAS pour votre disque managé vierge, vous pouvez l'utiliser pour définir votre disque managé en tant que destination de votre commande de chargement.

Utilisez AzCopy v10 pour charger votre fichier de disque dur virtuel local sur un disque managé en spécifiant l'URI de la SAS que vous avez générée.

Ce chargement présente le même débit que le disque [HDD Standard](../disks-types.md#standard-hdd) correspondant. Par exemple, pour une taille correspondant à S4, vous aurez un débit allant jusqu'à 60 Mio/s. Mais pour une taille correspondant à S70, le débit ira jusqu’à 500 Mio/s.

```
AzCopy.exe copy "c:\somewhere\mydisk.vhd" $diskSas.AccessSAS --blob-type PageBlob
```

Lorsque vous avez terminé le chargement et que vous n’avez plus rien à écrire sur le disque, révoquez la SAS. La révocation de la SAS changera l'état du disque managé et vous permettra de le joindre à une machine virtuelle.

Remplacez `<yourdiskname>` et `<yourresourcegroupname>`, puis exécutez la commande suivante :

```powershell
Revoke-AzDiskAccess -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>'
```

## <a name="copy-a-managed-disk"></a>Copier un disque managé

Le chargement direct simplifie également le processus de copie d’un disque managé. Vous pouvez effectuer une copie au sein d’une même région ou copier votre disque managé vers une autre région.

Le script suivant effectue cette opération pour vous. Le processus est similaire aux étapes décrites précédemment, à quelques différences près puisque vous utilisez un disque existant.

> [!IMPORTANT]
> Vous devez ajouter un décalage de 512 quand vous fournissez la taille en octets d’un disque managé d’Azure. En effet, Azure omet le pied de page lors du retour de la taille du disque. Si vous ne le faites pas, la copie échouera. Le script suivant s’en charge pour vous.

Remplacez `<sourceResourceGroupHere>`, `<sourceDiskNameHere>`, `<targetDiskNameHere>`, `<targetResourceGroupHere>`, `<yourOSTypeHere>` et `<yourTargetLocationHere>` (la valeur d'emplacement pourrait par exemple être uswest2) par vos valeurs, puis exécutez le script suivant afin de copier un disque managé.

> [!TIP]
> Si vous créez un disque de système d’exploitation, ajoutez -HyperVGeneration ’<yourGeneration>’ à `New-AzDiskConfig`.

```powershell

$sourceRG = <sourceResourceGroupHere>
$sourceDiskName = <sourceDiskNameHere>
$targetDiskName = <targetDiskNameHere>
$targetRG = <targetResourceGroupHere>
$targetLocate = <yourTargetLocationHere>
#Expected value for OS is either "Windows" or "Linux"
$targetOS = <yourOSTypeHere>

$sourceDisk = Get-AzDisk -ResourceGroupName $sourceRG -DiskName $sourceDiskName

# Adding the sizeInBytes with the 512 offset, and the -Upload flag
$targetDiskconfig = New-AzDiskConfig -SkuName 'Standard_LRS' -osType $targetOS -UploadSizeInBytes $($sourceDisk.DiskSizeBytes+512) -Location $targetLocate -CreateOption 'Upload'

$targetDisk = New-AzDisk -ResourceGroupName $targetRG -DiskName $targetDiskName -Disk $targetDiskconfig

$sourceDiskSas = Grant-AzDiskAccess -ResourceGroupName $sourceRG -DiskName $sourceDiskName -DurationInSecond 86400 -Access 'Read'

$targetDiskSas = Grant-AzDiskAccess -ResourceGroupName $targetRG -DiskName $targetDiskName -DurationInSecond 86400 -Access 'Write'

azcopy copy $sourceDiskSas.AccessSAS $targetDiskSas.AccessSAS --blob-type PageBlob

Revoke-AzDiskAccess -ResourceGroupName $sourceRG -DiskName $sourceDiskName

Revoke-AzDiskAccess -ResourceGroupName $targetRG -DiskName $targetDiskName 
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez chargé un disque dur virtuel sur un disque managé, vous pouvez attacher votre disque à une machine virtuelle et commencer à l’utiliser.

Pour découvrir comment joindre un disque de données à une machine virtuelle, consultez l’article suivant : [Joindre un disque de données à une machine virtuelle Windows à l'aide de PowerShell](attach-disk-ps.md). Pour utiliser le disque comme disque de système d’exploitation, consultez [Créer une machine virtuelle Windows à partir d’un disque spécialisé](create-vm-specialized.md#create-the-new-vm).
