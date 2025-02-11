---
title: Déployer sur IoT Edge pour Linux sur Windows - Azure
description: Cet article fournit des conseils sur la façon de déployer sur un appareil IoT Edge pour Linux sur Windows.
ms.topic: how-to
ms.date: 06/01/2021
ms.openlocfilehash: e80721375cf4b0c912fe47ec76c2cebe92359f90
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122532415"
---
# <a name="deploy-to-an-iot-edge-for-linux-on-windows-eflow-device"></a>Déployer sur un appareil IoT Edge pour Linux sur Windows (EFLOW)

Dans cet article, vous allez découvrir comment déployer Azure Video Analyzer sur un appareil de périphérie qui dispose d’[IoT Edge pour Linux sur Windows (EFLOW)](../../iot-edge/iot-edge-for-linux-on-windows.md). Une fois que vous aurez fini de suivre les étapes de ce document, vous pourrez exécuter un [pipeline](pipeline.md) qui détecte les mouvements dans une vidéo et émet les événements correspondants vers le hub IoT dans le cloud. Vous pourrez ensuite utiliser le pipeline dans des scénarios avancés et apporter la puissance d’Azure Video Analyzer à votre appareil IoT Edge basé sur Windows.

## <a name="prerequisites"></a>Prérequis 

* Un compte Azure disposant d’un abonnement actif. Si vous n’en avez pas déjà un, [créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* [Visual Studio Code](https://code.visualstudio.com/) sur votre machine de développement. Vérifiez que vous disposez de l’[extension Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Lisez [Qu’est-ce qu’Azure IoT Edge pour Linux sur Windows](../../iot-edge/iot-edge-for-linux-on-windows.md).

## <a name="deployment-steps"></a>Étapes du déploiement

Ce qui suit décrit le flux global du document. En 5 étapes simples, vous devez être prêt à exécuter Azure Video Analyzer sur un appareil Windows disposant d’EFLOW :

![Diagramme d’IoT Edge pour Linux sur Windows (EFLOW).](./media/deploy-iot-edge-linux-on-windows/eflow.png)

1. [Installez EFLOW](../../iot-edge/how-to-install-iot-edge-on-windows.md) sur votre appareil Windows à l’aide de PowerShell.


1. Une fois EFLOW configuré, saisissez la commande `Connect-EflowVm` dans PowerShell (avec des privilèges administratifs) pour vous connecter. Cela fera apparaître un terminal bash dans PowerShell pour contrôler la machine virtuelle EFLOW, où vous pourrez exécuter des commandes Linux, notamment des utilitaires comme Top et Nano. 

    > [!TIP] 
    > Pour quitter la machine virtuelle EFLOW, saisissez `exit` dans le terminal.

1. Connectez-vous à la machine virtuelle EFLOW via PowerShell et saisissez la commande suivante :

    `bash -c "$(curl -sL https://aka.ms/ava-edge/prep_device)"`

    Le module Azure Video Analyzer s’exécute sur l’appareil de périphérie avec des comptes d’utilisateurs locaux non privilégiés. Il a aussi besoin de certains dossiers locaux pour le stockage des données de configuration d’application. Enfin, pour ce guide pratique, nous tirons profit d’un [simulateur RTSP](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) qui relaie un flux vidéo en temps réel vers le module AVA à des fins d’analyse. Ce simulateur prend comme entrée des fichiers vidéo préenregistrés à partir d’un répertoire d’entrée. 

    Le script de préparation d’appareil utilisé ci-dessus permet d’automatiser ces tâches, ce qui vous permet de créer en une seule commande tous les dossiers d’entrée et de configuration pertinents, les fichiers d’entrée vidéo et les comptes d’utilisateur avec privilèges. Une fois la commande terminée, les dossiers suivants auront normalement été créés sur votre appareil Edge. 

    * `/home/localedgeuser/samples`
    * `/home/localedgeuser/samples/input`
    * `/var/lib/videoanalyzer`
    * `/var/media`

    Notez la présence des fichiers vidéo (*.mkv) dans le dossier /home/localedgeuser/samples/input, qui servent de fichiers d’entrée à analyser. 
1. Une fois que le périphérique est configuré, qu’il est inscrit auprès du hub et qu’il s’exécute correctement avec les structures de dossiers appropriées créées, l’étape suivante consiste à configurer les ressources Azure supplémentaires ci-dessous et à déployer le module AVA. Le modèle de déploiement suivant s’occupe de la création des ressources :

    [![Déployer sur Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)
    
    Le processus de déploiement prend environ 20 minutes. À la fin de l’opération, certaines ressources Azure sont déployées dans l’abonnement Azure, notamment :

    * Compte Video Analyzer : ce service cloud est utilisé pour inscrire le module de périphérie Video Analyzer ainsi que pour la lecture des vidéos enregistrées et l’analytique vidéo.
    * Compte de stockage : pour stocker les vidéos enregistrées et l’analytique vidéo.
    * Identité managée : il s’agit de l’identité managée affectée par l’utilisateur qui permet de gérer l’accès au compte de stockage ci-dessus.
    * IoT Hub : fait office de hub de messagerie centralisé pour la communication bidirectionnelle entre votre application IoT, les modules IoT Edge et les appareils qu'il gère.

    Dans le modèle, quand vous êtes invité à indiquer si vous avez besoin d’un appareil de périphérie, choisissez l’option Use an existing edge device (Utiliser un appareil de périphérie existant), car vous avez déjà créé l’appareil et le hub IoT. Vous êtes également invité à entrer le nom du hub IoT et l’ID d’appareil IoT Edge au cours des étapes suivantes.  
    
    ![Utiliser un appareil existant](./media/deploy-iot-edge-linux-on-windows/use-existing-device.png) 

    Une fois terminé, vous pouvez vous reconnecter à la machine virtuelle EFLOW et exécuter la commande suivante.

    **`sudo iotedge list`**

    Les quatre modules suivants doivent être déployés et en cours d’exécution sur votre appareil Edge. Notez que le script de création de ressources déploie le module AVA ainsi que les modules IoT Edge (edgeAgent et edgeHub) et un module de simulateur RTSP pour fournir le flux vidéo RTSP simulé.
    
    ![Modules déployés](./media/vscode-common-screenshots/avaedge-module.png)
1. Une fois les modules déployés et configurés, vous êtes prêt à exécuter votre premier pipeline AVA sur EFLOW. Vous pouvez exécuter un pipeline de détection de mouvement simple comme celui décrit ci-dessous, et visualiser les résultats en exécutant les étapes suivantes :

    ![Solution Video Analyzer basée sur la détection de mouvement](./media/get-started-detect-motion-emit-events/motion-detection.svg)

    1. [Configurez](get-started-detect-motion-emit-events.md#prepare-to-monitor-the-modules) l’extension Azure IoT Tools.
    1. Définissez le pipelineTopology, instanciez le livePipeline, puis activez-le via ces [appels de méthode directe](get-started-detect-motion-emit-events.md#use-direct-method-calls).
    1. [Observez les résultats](get-started-detect-motion-emit-events.md#observe-results) sur le hub.
    1. Appelez des [méthodes de nettoyage](get-started-detect-motion-emit-events.md#deactivate-the-live-pipeline).
    1. Supprimez vos ressources si vous n’en avez plus besoin.

        > [!IMPORTANT]
        > Les ressources qui ne sont pas supprimées peuvent rester actives et occasionner des coûts Azure. Veillez à supprimer les ressources que vous n’avez pas l’intention d’utiliser.
        
## <a name="next-steps"></a>Étapes suivantes

* Essayez la détection de mouvement avec l’enregistrement de vidéos pertinentes dans le cloud. Suivez les étapes décrites dans le guide de démarrage rapide sur [la détection de mouvement et l’enregistrement de clips vidéo](detect-motion-record-video-edge-devices.md).
* Exécutez l’[IA sur la vidéo en direct](analyze-live-video-use-your-model-http.md#overview) (vous pouvez ignorer la configuration des prérequis, car elle a déjà été effectuée ci-dessus).
* Utilisez notre [extension VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.live-video-analytics-edge) pour voir les pipelines supplémentaires.
* Utilisez une [caméra IP](https://en.wikipedia.org/wiki/IP_camera) qui prend en charge RTSP au lieu d’utiliser le simulateur RTSP. Vous trouverez les caméras IP qui prennent RTSP en charge dans la page des [produits conformes ONVIF](https://www.onvif.org/conformant-products/). Recherchez les appareils conformes aux profils G, S ou T.
