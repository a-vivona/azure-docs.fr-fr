---
author: fvneerden
ms.service: azure-video-analyzer
ms.topic: include
ms.date: 05/05/2021
ms.author: faneerde
ms.openlocfilehash: 5f8fd87a6eaa04cde7f5edde20b5debbe872c2f8
ms.sourcegitcommit: cc099517b76bf4b5421944bd1bfdaa54153458a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/09/2021
ms.locfileid: "113559559"
---
Vous allez utiliser le module périphérique Video Analyzer pour détecter les mouvements dans le flux vidéo en direct entrant et envoyer des événements à IoT Hub. Pour voir ces événements, effectuez les étapes suivantes :

1. Ouvrez le volet Explorateur dans Visual Studio Code, puis recherchez Azure IoT Hub dans l’angle inférieur gauche.
1. Développez le nœud **Appareils**.
1. Cliquez avec le bouton droit sur **avasample-iot-edge-device**, puis sélectionnez **Démarrer la supervision du point de terminaison d’événements intégré**.

   ![Démarrer la supervision d’un point de terminaison d’événement intégré](../../../media/vscode-common-screenshots/start-monitoring.png)

    [!INCLUDE [provide-builtin-endpoint](../../common-includes/provide-builtin-endpoint.md)]
