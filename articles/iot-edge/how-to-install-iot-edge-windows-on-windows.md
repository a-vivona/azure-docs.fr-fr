---
title: Installer Azure IoT Edge pour Windows | Microsoft Docs
description: Installer Azure IoT Edge pour les conteneurs Windows sur des appareils Windows
author: kgremban
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 12/18/2020
ms.author: kgremban
monikerRange: iotedge-2018-06
ms.openlocfilehash: 676bd23dce7ca79f3640bdfa8f9b55e427390e91
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122523997"
---
# <a name="install-and-manage-azure-iot-edge-with-windows-containers"></a>Installer et gérer Azure IoT Edge avec des conteneurs Windows

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

Le runtime Azure IoT Edge est ce qui transforme un appareil en appareil IoT Edge. Une fois qu’un appareil est configuré avec le runtime IoT Edge, vous pouvez commencer à déployer une logique métier sur celui-ci à partir du cloud. Pour en savoir plus, consultez [Présentation du runtime Azure IoT Edge et de son architecture](iot-edge-runtime.md).

La configuration d’un appareil IoT Edge présente deux étapes. La première étape consiste à installer le runtime et ses dépendances. La deuxième étape consiste à connecter l’appareil à son identité dans le cloud et à configurer l’authentification avec IoT Hub.

Cet article répertorie les étapes à suivre pour installer le runtime d’Azure IoT Edge sur des conteneurs Windows. Si vous envisagez d’utiliser des conteneurs Linux sur un appareil Windows, consultez l’article [Azure IOT Edge pour Linux sur Windows](how-to-install-iot-edge-on-windows.md).

>[!NOTE]
>Azure IoT Edge pour conteneurs Windows ne sera pas pris en charge à partir de la version 1.2 d’Azure IoT Edge.
>
>Envisagez d’utiliser la nouvelle méthode pour exécuter IoT Edge sur des appareils Windows, [Azure IoT Edge pour Linux sur Windows](iot-edge-for-linux-on-windows.md).

## <a name="prerequisites"></a>Prérequis

* Un appareil Windows

  IoT Edge avec des conteneurs Windows requiert Windows version 1809/Build 17763, qui est la [build prise en charge à long terme de Windows](/windows/release-information/)la plus récente. Veillez à consulter la [liste de systèmes pris en charge](support.md#operating-systems) pour obtenir la liste des références SKU prises en charge.

* Un [ID d’appareil inscrit](how-to-register-device.md)

  Si vous avez inscrit votre appareil avec l’authentification par clé symétrique, préparez la chaîne de connexion de l’appareil.

  Si vous avez inscrit votre appareil avec l’authentification par certificat auto-signé X.509, vous devez disposer au moins de l’un des certificats d’identité que vous avez utilisés pour inscrire l’appareil et la clé privée correspondante sur votre appareil.

## <a name="install-a-container-engine"></a>Installer un moteur de conteneur

Azure IoT Edge s’appuie sur un runtime de conteneur compatible avec OCI, comme [Moby](https://github.com/moby/moby). Un moteur basé sur Moby qui est inclus dans le script d’installation. Aucune étape supplémentaire n’est nécessaire pour installer le moteur.

## <a name="install-the-iot-edge-security-daemon"></a>Installer le démon de sécurité IoT Edge

1. Exécutez PowerShell ISE en tant qu’administrateur.

   Utilisez une session AMD64 de PowerShell, et non PowerShell (x86). Si vous ne savez pas quel type de session vous utilisez, exécutez la commande suivante :

   ```powershell
   (Get-Process -Id $PID).StartInfo.EnvironmentVariables["PROCESSOR_ARCHITECTURE"]
   ```

2. Exécutez la commande [Deploy-IoTEdge](reference-windows-scripts.md#deploy-iotedge) qui effectue les tâches suivantes :

   * Vérifie que votre ordinateur Windows est sous une version prise en charge.
   * Active la fonctionnalité des conteneurs.
   * Télécharge le moteur Moby et le runtime IoT Edge.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Deploy-IoTEdge
   ```

3. Redémarrez votre appareil si vous y êtes invité.

Quand vous installez IoT Edge sur un appareil, vous pouvez utiliser des paramètres supplémentaires pour modifier le processus, à savoir :

* Diriger le trafic pour qu’il transite par un serveur proxy,
* Pointer le programme d’installation vers un répertoire local pour une installation hors connexion.

Pour plus d’informations sur ces paramètres supplémentaires, consultez [Scripts PowerShell pour IoT Edge sur conteneurs Windows](reference-windows-scripts.md).

## <a name="provision-the-device-with-its-cloud-identity"></a>Provisionnement de l’appareil avec son identité cloud

Maintenant que le moteur de conteneur et le runtime d’IoT Edge sont installés sur votre appareil, vous pouvez passer à l’étape suivante, à savoir configurer l’appareil avec ses informations d’identité et d’authentification cloud.

Choisissez la section suivante en fonction du type d’authentification que vous souhaitez utiliser :

* [Option n°1 : Authentification avec des clés symétriques](#option-1-authenticate-with-symmetric-keys)
* [Option n°2 : Authentification avec des certificats X.509](#option-2-authenticate-with-x509-certificates)

### <a name="option-1-authenticate-with-symmetric-keys"></a>Option 1 : Authentification avec des clés symétriques

À ce stade, le runtime IoT Edge est installé sur votre appareil Windows, et vous devez configurer l’appareil avec ses informations d’identité et d’authentification cloud.

Cette section décrit la procédure à suivre pour approvisionner un appareil avec une authentification par clé symétrique. Vous devez avoir inscrit votre appareil dans IoT Hub et récupéré la chaîne de connexion à partir des informations de l’appareil. Si ce n’est pas le cas, suivez les étapes décrites dans [Inscrire un appareil IoT Edge dans IoT Hub](how-to-register-device.md).

1. Sur l’appareil IoT Edge, exécutez PowerShell en tant qu’administrateur.

2. Utilisez la commande [Initialize-IoTEdge](reference-windows-scripts.md#initialize-iotedge) pour configurer le runtime IoT Edge sur votre ordinateur. Par défaut, la commande est une commande d’approvisionnement manuel avec des conteneurs Windows.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -ManualConnectionString -ContainerOs Windows
   ```

   * Si vous avez téléchargé le script IoTEdgeSecurityDaemon.ps1 sur votre appareil pour une installation hors connexion ou l’installation d’une version spécifique, veillez à référencer la copie locale du script.

      ```powershell
      . <path>/IoTEdgeSecurityDaemon.ps1
      Initialize-IoTEdge -ManualX509
      ```

3. Lorsque vous y êtes invité, spécifiez la chaîne de connexion de l’appareil que vous avez récupérée dans la section précédente. La chaîne de connexion de l’appareil associe l’appareil physique à un ID d’appareil dans IoT Hub et fournit des informations d’authentification.

   La chaîne de connexion se présente au format suivant et ne doit pas contenir de guillemets : `HostName={IoT hub name}.azure-devices.net;DeviceId={device name};SharedAccessKey={key}`

Quand vous provisionnez un appareil manuellement, vous pouvez utiliser des paramètres supplémentaires pour changer le processus, à savoir :

* Diriger le trafic pour qu’il transite par un serveur proxy,
* Déclarer une image conteneur d’agent Edge spécifique, et fournir des informations d’identification si l’image se trouve dans un registre privé

Pour plus d’informations sur ces paramètres supplémentaires, consultez [Scripts PowerShell pour IoT Edge sur conteneurs Windows](reference-windows-scripts.md).

### <a name="option-2-authenticate-with-x509-certificates"></a>Option n°2 : Authentification avec des certificats X.509

À ce stade, le runtime IoT Edge est installé sur votre appareil Windows, et vous devez configurer l’appareil avec ses informations d’identité et d’authentification cloud.

Cette section décrit la procédure à suivre pour approvisionner un appareil avec une authentification par certificat X.509. Vous devez avoir inscrit votre appareil dans IoT Hub, en indiquant les empreintes correspondant au certificat et à la clé privée situés sur votre appareil IoT Edge. Si ce n’est pas le cas, suivez les étapes décrites dans [Inscrire un appareil IoT Edge dans IoT Hub](how-to-register-device.md).

1. Sur l’appareil IoT Edge, exécutez PowerShell en tant qu’administrateur.

2. Utilisez la commande [Initialize-IoTEdge](reference-windows-scripts.md#initialize-iotedge) pour configurer le runtime IoT Edge sur votre ordinateur.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -ManualX509
   ```

   * Si vous avez téléchargé le script IoTEdgeSecurityDaemon.ps1 sur votre appareil pour une installation hors connexion ou l’installation d’une version spécifique, veillez à référencer la copie locale du script.

      ```powershell
      . <path>/IoTEdgeSecurityDaemon.ps1
      Initialize-IoTEdge -ManualX509
      ```

3. À l’invite, fournissez les informations suivantes :

   * **IotHubHostName** : nom d’hôte de l’hub IoT auquel l’appareil doit se connecter. Par exemple, `{IoT hub name}.azure-devices.net`.
   * **DeviceId** : ID que vous avez indiqué lors de l’inscription de l’appareil.
   * **X509IdentityCertificate** : chemin absolu d’un certificat d’identité sur l’appareil. Par exemple, `C:\path\identity_certificate.pem`.
   * **X509IdentityPrivateKey** : chemin absolu du fichier de clé privée pour le certificat d’identité fourni. Par exemple, `C:\path\identity_key.pem`.

Quand vous provisionnez un appareil manuellement, vous pouvez utiliser des paramètres supplémentaires pour changer le processus, à savoir :

* Diriger le trafic pour qu’il transite par un serveur proxy,
* Déclarer une image conteneur d’agent Edge spécifique, et fournir des informations d’identification si l’image se trouve dans un registre privé

Pour plus d’informations sur ces paramètres supplémentaires, consultez [Scripts PowerShell pour IoT Edge sur conteneurs Windows](reference-windows-scripts.md).

## <a name="offline-or-specific-version-installation-optional"></a>Installation d’une version spécifique hors connexion (facultatif)

Les étapes de cette section concernent les scénarios non couverts par les étapes d’installation standard. Cela peut inclure :

* Installer IoT Edge en mode hors connexion
* Installer une version Release candidate
* Installer une version autre que la version la plus récente

Lors de l’installation, trois fichiers sont téléchargés :

* Un script PowerShell, qui contient les instructions d’installation
* Le fichier cab Microsoft Azure IoT Edge, contenant le démon de sécurité IoT Edge (iotedged), le moteur du conteneur Moby et Moby CLI
* Programme d’installation du package Visual C++ Redistribuable (Runtime VC)

Si votre appareil est hors connexion pendant l’installation, ou si vous souhaitez installer une version spécifique d’IoT Edge, vous pouvez télécharger à l’avance ces fichiers sur l’appareil. Quand il est temps d’effectuer l’installation, pointez le script d’installation sur le répertoire contenant les fichiers téléchargés. Le programme d’installation commence par vérifier le répertoire, puis télécharge uniquement les composants qui ne s’y trouvent pas. Si tous les fichiers sont disponibles hors connexion, vous pouvez procéder à l’installation sans connexion Internet.

1. Pour accéder aux derniers fichiers d’installation IoT Edge ainsi qu’aux versions précédentes, voir les [publications d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases).

2. Recherchez la version que vous souhaitez installer, puis téléchargez les fichiers suivants de la section **Ressources** des notes de publication vers votre appareil IoT :

   * IoTEdgeSecurityDaemon.ps1
   * Microsoft-Azure-IoTEdge-amd64.cab du canal de distribution 1.1.

   Il est important d’utiliser le script PowerShell de la même version que le fichier .cab que vous utilisez, car les fonctionnalités changent pour prendre en charge les fonctionnalités de chaque version.

3. Si le fichier .cab que vous avez téléchargé est doté d’un suffixe d’architecture, renommez le fichier uniquement **Microsoft-Azure-IoTEdge.cab**.

4. Éventuellement, téléchargez un programme d’installation pour Visual C++ Redistributable. Par exemple, le script PowerShell utilise cette version : [vc_redist.x64. exe](https://download.microsoft.com/download/0/6/4/064F84EA-D1DB-4EAA-9A5C-CC2F0FF6A638/vc_redist.x64.exe). Enregistrez le programme d’installation dans le même dossier que les fichiers IoT Edge sur votre appareil.

5. Pour installer avec des composants hors connexion, [effectuez un appel de source de type « dot source »](/powershell/module/microsoft.powershell.core/about/about_scripts#script-scope-and-dot-sourcing) de la copie locale du script PowerShell.

6. Exécutez la commande [Deploy-IoTEdge](reference-windows-scripts.md#deploy-iotedge) avec le paramètre `-OfflineInstallationPath`. Indiquez le chemin d’accès absolu au répertoire de fichiers. Par exemple,

   ```powershell
   . <path>\IoTEdgeSecurityDaemon.ps1
   Deploy-IoTEdge -OfflineInstallationPath <path>
   ```

   La commande de déploiement utilise les composants trouvés dans le répertoire de fichiers local fourni. S’il manque le fichier .cab ou le programme d'installation de Visual C++, elle tente de les télécharger.

## <a name="update-iot-edge"></a>Mettre à jour IoT Edge

Utilisez la commande `Update-IoTEdge` pour mettre à jour le démon de sécurité. Le script extrait automatiquement la dernière version du démon de sécurité.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Update-IoTEdge
```

La commande Update-IoTEdge supprime et met à jour le démon de sécurité de l’appareil, ainsi que les deux images conteneur du runtime. Le fichier config.yaml est conservé sur l’appareil, de même que les données du moteur de conteneur Moby. Le fait de conserver les informations de configuration évite d’avoir à indiquer de nouveau la chaîne de connexion ou le service Device Provisioning de votre appareil lors du processus de mise à jour.

Si vous souhaitez mettre à jour vers une version spécifique du démon de sécurité, recherchez la version du canal de distribution 1.1 que vous souhaitez cibler à partir de [Versions d’Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases). Dans cette version, téléchargez le fichier **Microsoft-Azure-IoTEdge.cab**. Utilisez ensuite le paramètre `-OfflineInstallationPath` pour pointer vers l’emplacement du fichier local. Par exemple :

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Update-IoTEdge -OfflineInstallationPath <absolute path to directory>
```

>[!NOTE]
>Le paramètre `-OfflineInstallationPath` recherche un fichier nommé **Microsoft-Azure-IoTEdge.cab** dans le répertoire fourni. Renommez le fichier pour supprimer le suffixe d’architecture, le cas échéant.

Si vous souhaitez mettre à jour un appareil hors connexion, recherchez la version que vous souhaitez cibler parmi les [versions Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases). Dans cette version, téléchargez les fichiers *IoTEdgeSecurityDaemon.ps1* et *Microsoft-Azure-IoTEdge.cab*. Il est important d’utiliser le script PowerShell de la même version que le fichier .cab que vous utilisez, car les fonctionnalités changent pour prendre en charge les fonctionnalités de chaque version.

Si le fichier .cab que vous avez téléchargé est doté d’un suffixe d’architecture, renommez le fichier uniquement **Microsoft-Azure-IoTEdge.cab**.

Pour mettre à jour avec des composants hors connexion, [effectuez un appel de source de type « dot source »](/powershell/module/microsoft.powershell.core/about/about_scripts#script-scope-and-dot-sourcing) de la copie locale du script PowerShell. Ensuite, utilisez le paramètre `-OfflineInstallationPath` dans la commande `Update-IoTEdge` et spécifiez le chemin absolu du répertoire des fichiers. Par exemple,

```powershell
. <path>\IoTEdgeSecurityDaemon.ps1
Update-IoTEdge -OfflineInstallationPath <path>
```

Pour plus d’informations sur les options de mise à jour, utilisez la commande `Get-Help Update-IoTEdge -full` ou consultez [Script PowerShell pour IoT Edge pour conteneurs Windows](reference-windows-scripts.md).

## <a name="uninstall-iot-edge"></a>Désinstaller IoT Edge

Si vous souhaitez supprimer l’installation d’IoT Edge de votre appareil, utilisez les commandes suivantes.

Si vous souhaitez supprimer l’installation d’IoT Edge de votre appareil Windows, utilisez la commande [Uninstall-IoTEdge](reference-windows-scripts.md#uninstall-iotedge) à partir d’une fenêtre PowerShell d’administration. Cette commande supprime le runtime IoT Edge, ainsi que votre configuration existante et les données du moteur Moby.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
Uninstall-IoTEdge
```

Pour plus d’informations sur les options de désinstallation, utilisez la commande `Get-Help Uninstall-IoTEdge -full`.

## <a name="next-steps"></a>Étapes suivantes

Passez à [Déployer des modules IoT Edge](how-to-deploy-modules-portal.md) pour savoir comment déployer des modules sur votre appareil.
