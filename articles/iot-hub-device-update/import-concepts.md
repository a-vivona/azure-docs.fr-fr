---
title: Comprendre l’importation de Device Update pour IoT Hub | Microsoft Docs
description: Concepts clés pour l’importation d’une nouvelle mise à jour dans Device Update pour IoT Hub.
author: andrewbrownmsft
ms.author: andbrown
ms.date: 2/10/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: 39a1acb596f4d56915aabfa0a64fc69d6e55aec2
ms.sourcegitcommit: 9339c4d47a4c7eb3621b5a31384bb0f504951712
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2021
ms.locfileid: "113769097"
---
# <a name="importing-updates-into-device-update-for-iot-hub"></a>Importation de mises à jour dans Device Update pour IoT Hub
Pour déployer une mise à jour sur des appareils à partir de Device Update pour IoT Hub, vous devez d’abord _importer_ cette mise à jour dans le service Device Update. Voici une vue d’ensemble de certains concepts importants à comprendre en ce qui concerne l’importation de mises à jour.

## <a name="limits-on-importing-updates"></a>Limites sur l’importation des mises à jour
Pour chaque instance Device Update pour IoT Hub, les limites suivantes sont appliquées :

* 5 fournisseurs par instance ADU
* 5 noms par fournisseur
* 25 versions par nom

Il existe également une limite de 100 versions de mise à jour cumulées (pour l’ensemble des fournisseurs et noms).

La taille de fichier maximale autorisée est de 800 Mo pour tout fichier individuel dans une soumission de mise à jour. Le total _combiné_ de tous les fichiers dans une même soumission de mise à jour ne doit pas non plus dépasser 800 Mo.

## <a name="import-manifest"></a>Manifeste d’importation

Un manifeste d’importation est un fichier JSON qui définit des informations importantes sur la mise à jour que vous importez. Vous allez soumettre à la fois votre manifeste d’importation et le ou les fichiers de mise à jour associés (par exemple, un package de mise à jour de microprogramme) dans le cadre du processus d’importation. Les métadonnées définies dans ce manifeste d’importation sont utilisées pour l’ingestion de la mise à jour. Certaines métadonnées sont également utilisées au moment du déploiement, par exemple, pour vérifier si une mise à jour a été correctement installée.

Le manifeste d’importation contient plusieurs éléments qui représentent des concepts importants de Device Update pour IoT Hub. Ces concepts sont indiqués ci-dessous.

### <a name="update-identity-update-id"></a>Identité de mise à jour (ID de mise à jour)

L’identité de mise à jour représente l’identificateur unique d’une mise à jour. Elle définit les propriétés importantes relatives à une mise à jour en cours d’importation. L’identité de mise à jour est composée de trois parties :
* Fournisseur : il s’agit de l’entité qui crée ou qui est directement responsable de la mise à jour. Souvent, il s’agit d’un nom d’entreprise.
* Nom : identificateur d’une classe de mises à jour. La classe peut être une chose quelconque que vous choisissez. Elle correspond souvent à un nom d’appareil ou de modèle.
* Version : il s’agit d’un numéro de version qui distingue cette mise à jour d’autres mises à jour dotées du même fournisseur et du même nom. Cette version est utilisée par le service Device Update pour IoT Hub et peut correspondre ou non à une version d’un composant logiciel individuel sur l’appareil. 

### <a name="compatibility"></a>Compatibilité

Pour simplifier les déploiements de mise à jour, Device Update pour IoT Hub compare les propriétés de compatibilité pour une mise à jour, qui sont définies dans le manifeste d’importation, avec les propriétés d’appareil correspondantes. Seules les mises à jour qui ont des propriétés correspondantes sont renvoyées et disponibles pour être déployées.

### <a name="installedcriteria"></a>InstalledCriteria

L’élément InstalledCriteria est utilisé par l’agent de mise à jour sur un appareil pour déterminer si une mise à jour a été installée avec succès.


## <a name="next-steps"></a>Étapes suivantes

Si vous êtes prêt, suivez le [Guide pratique d’importation](./import-update.md), qui vous guidera pas à pas tout au long du processus d’importation.


