---
title: Comment exécuter le runtime d’intégration auto-hébergé dans un conteneur Windows
description: Apprenez à exécuter le runtime d’intégration auto-hébergé dans un conteneur Windows.
ms.author: lle
author: lrtoyou1223
ms.service: data-factory
ms.subservice: integration-runtime
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/05/2020
ms.openlocfilehash: 0c1452368af6e3bd3083b3b7ecf505d2d911b6b6
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122641403"
---
# <a name="how-to-run-self-hosted-integration-runtime-in-windows-container"></a>Comment exécuter le runtime d’intégration auto-hébergé dans un conteneur Windows

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment exécuter le runtime d’intégration auto-hébergé dans un conteneur Windows.
Azure Data Factory fournit la prise en charge officielle du conteneur Windows pour l’exécution du runtime d’intégration auto-hébergé. Vous pouvez télécharger le code source de la build Docker et combiner le processus de génération et d’exécution dans votre propre pipeline de livraison continue. 

## <a name="prerequisites"></a>Conditions préalables requises 
- [Configuration requise pour un conteneur Windows](/virtualization/windowscontainers/deploy-containers/system-requirements)
- Docker, version 2.3 et ultérieure 
- Version 5.2.7713.1 et ultérieures du runtime d’intégration auto-hébergé 
## <a name="get-started"></a>Prise en main 
1.  Installez Docker et activez le conteneur Windows 
2.  Télécharger le code source à partir de https://github.com/Azure/Azure-Data-Factory-Integration-Runtime-in-Windows-Container
3.  Téléchargez la version la plus récente de SHIR dans le dossier « SHIR » 
4.  Ouvrez votre dossier dans l’interpréteur de commandes : 
```console
cd "yourFolderPath"
```

5.  Générez l’image de Docker pour Windows : 
```console
docker build . -t "yourDockerImageName" 
```
6.  Exécutez le conteneur Docker : 
```console
docker run -d -e NODE_NAME="irNodeName" -e AUTH_KEY="IR_AUTHENTICATION_KEY" -e ENABLE_HA=true -e HA_PORT=8060 "yourDockerImageName"    
```
> [!NOTE]
> AUTH_KEY est obligatoire pour cette commande. NODE_NAME, ENABLE_HA et HA_PORT sont facultatifs. Si vous ne définissez pas de valeur, la commande utilise les valeurs par défaut. La valeur par défaut est false pour ENABLE_HA et 8060 pour HA_PORT.

## <a name="container-health-check"></a>Vérification de l’intégrité du conteneur 
Après une période de démarrage de 120 secondes, le vérificateur d’intégrité s’exécute toutes les 30 secondes. Il fournit l’état d’intégrité du runtime d’intégration au moteur du conteneur. 

## <a name="limitations"></a>Limites
Actuellement, nous ne prenons pas en charge les fonctionnalités ci-dessous lors de l’exécution du runtime d’intégration auto-hébergé dans un conteneur Windows :
- Serveur proxy HTTP 
- Communication nœud à nœud chiffrée avec un certificat TLS/SSL 
- Création et importation de sauvegarde 
- Service démon 
- Mise à jour automatique 

### <a name="next-steps"></a>Étapes suivantes
- Étudiez les [concepts de runtime d’intégration dans Azure Data Factory](./concepts-integration-runtime.md).
- Apprenez à [créer un runtime d’intégration auto-hébergé sur le portail Azure](./create-self-hosted-integration-runtime.md).