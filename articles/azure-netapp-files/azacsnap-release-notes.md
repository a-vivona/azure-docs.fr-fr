---
title: Notes de publication pour l’outil Azure Application Consistent Snapshot pour Azure NetApp Files | Microsoft Docs
description: Fournit des notes de publication pour l’outil Azure Application Consistent Snapshot que vous pouvez utiliser avec Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/27/2021
ms.author: phjensen
ms.openlocfilehash: fd8d4e1bfed60aa8f3eae4d4d3c033247ab1268d
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579903"
---
# <a name="release-notes-for-azure-application-consistent-snapshot-tool"></a>Notes de publication pour l'outil Azure Application Consistent Snapshot

Cette page répertorie les principales modifications apportées à AzAcSnap pour fournir de nouvelles fonctionnalités ou résoudre les défauts.

## <a name="may-2021"></a>Mai-2021

### <a name="azacsnap-v501-build-2021052414837---patch-update-to-v50"></a>AzAcSnap v 5.0.1 (Build : 20210524,14837) - Mise à jour corrective de v5.0

AzAcSnap v 5.0.1 (Build : 20210524,14837) est fournie en tant que mise à jour corrective de la branche v5.0 avec les correctifs et améliorations suivants :

- Amélioration de la gestion du code de sortie.  Dans certains cas, le code de sortie 0 (zéro) a été émis même en cas d’échec d’exécution alors qu’il aurait dû être différent de zéro.  Les codes de sortie doivent maintenant avoir uniquement la valeur zéro si l’exécution de `azacsnap` s’est terminée avec succès et une valeur différente de zéro en cas d’échec.  En outre, la gestion des erreurs internes de AzAcSnap a été étendue pour capturer et émettre le code de sortie des commandes externes (par exemple, hdbsql, ssh) exécuté par AzAcSnap, s’il s’agit de la cause de l’échec.

Téléchargez la [version la plus récente](https://aka.ms/azacsnapdownload) du programme d’installation et consultez la [procédure de démarrage](azacsnap-get-started.md).

## <a name="april-2021"></a>Avril 2021

### <a name="azacsnap-v50-build-202104216349---ga-released-21-april-2021"></a>AzAcSnap v5.0 (Build : 20210421.6349) - Version GA (21 avril 2021)

AzAcSnap v5.0 (Build : 20210421.6349) a été mis à la disposition générale. Cette build a bénéficié des correctifs et améliorations suivants :

- Le délai de nouvelle tentative hdbsql (pour attendre une réponse de SAP HANA) est automatiquement défini sur la moitié du paramètre « savePointAbortWaitSeconds » pour éviter les situations de concurrence.  Le paramètre « savePointAbortWaitSeconds » peut être modifié directement dans le fichier de configuration JSON et doit être au minimum de 600 secondes.

## <a name="march-2021"></a>Mars 2021

### <a name="azacsnap-v50-preview-build2021031830771"></a>AzAcSnap v5.0 Preview (Build:20210318.30771)

AzAcSnap v5.0 Preview (Build:20210318.30771) a été publié avec les améliorations et correctifs suivants :

- Suppression de la nécessité d’ajouter l’utilisateur AZACSNAP dans les bases de données des locataires SAP HANA ; voir la section [Activer la communication avec SAP HANA](azacsnap-installation.md#enable-communication-with-sap-hana).
- Correctif pour autoriser une [restauration](azacsnap-cmd-ref-restore.md) avec des volumes configurés avec la QoS manuelle.
- Ajout du contrôle mutex pour limiter les connexions SSH pour Azure Large Instance.
- Correction du programme d’installation pour gérer les noms de chemin d’accès avec des espaces et d’autres problèmes connexes.
- En vue de la prise en charge d’autres serveurs de base de données, le paramètre facultatif « --hanasid » a été remplacé par « --dbsid ».

## <a name="next-steps"></a>Étapes suivantes

- [Bien démarrer avec l’outil Azure Application Consistent Snapshot](azacsnap-get-started.md)
