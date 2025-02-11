---
title: 'Tutoriel : Configurer un appareil Azure IoT Edge – Machine Learning sur IoT Edge'
description: Dans ce tutoriel, vous allez configurer une machine virtuelle Azure exécutant Linux en tant qu’appareil Azure IoT Edge servant de passerelle transparente.
author: kgremban
ms.author: kgremban
ms.date: 2/5/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: amqp
ms.openlocfilehash: 445163b0d33f404845a66b7d885815d6336dac46
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "121726365"
---
# <a name="tutorial-configure-an-azure-iot-edge-device"></a>Tutoriel : Configurer un appareil Azure IoT Edge

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

Dans cet article, nous configurons une machine virtuelle Azure exécutant Linux comme appareil Azure IoT Edge servant de passerelle transparente. La configuration d’une passerelle transparente permet aux appareils de se connecter à Azure IoT Hub à travers la passerelle sans savoir que la passerelle existe. De la même façon, l’utilisateur qui interagit avec les appareils dans IoT Hub n’a pas conscience de l’appareil de passerelle intermédiaire. À la fin, nous ajouterons des données d’analytique Edge à notre système en ajoutant des modules IoT Edge à la passerelle transparente.

>[!NOTE]
>Les concepts abordés dans ce tutoriel s’appliquent à toutes les versions d’IoT Edge, mais l’exemple d’appareil que vous créez pour tester le scénario exécute IoT Edge version 1.1.

Les étapes décrites dans cet article sont généralement effectuées par un développeur de cloud.

Dans cette section du tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer des certificats pour permettre à votre appareil de passerelle de se connecter de façon sécurisée à vos appareils en aval.
> * Créer un appareil IoT Edge.
> * Créer une machine virtuelle Azure pour simuler votre appareil IoT Edge.

## <a name="prerequisites"></a>Prérequis

Cet article fait partie d’une série décrivant l’utilisation d’Azure Machine Learning sur IoT Edge. Chaque article de la série s’appuie sur le travail réalisé dans le cadre de l’article précédent. Si vous êtes tombé directement sur cet article, consultez le [premier article](tutorial-machine-learning-edge-01-intro.md) de la série.

## <a name="create-certificates"></a>Créer des certificats

Pour qu’un appareil fonctionne comme une passerelle, il doit se connecter de manière sécurisée aux appareils en aval. IoT Edge vous permet d’utiliser une infrastructure à clé publique (PKI) pour configurer des connexions sécurisées entre les appareils. Dans ce cas, nous autorisons un appareil IoT en aval à se connecter à un appareil IoT Edge faisant office de passerelle transparente. Pour assurer un niveau raisonnable de sécurité, l’appareil en aval doit confirmer l’identité de l’appareil IoT Edge. Pour plus d’informations sur la façon dont les appareils IoT Edge utilisent des certificats, consultez [Détails d’utilisation des certificats Azure IoT Edge](iot-edge-certs.md).

Dans cette section, nous créons les certificats auto-signés à l’aide d’une image Docker que nous devons générer et exécuter. Nous avons choisi d’utiliser une image Docker pour effectuer cette étape, car elle réduit le nombre d’étapes nécessaires pour créer les certificats sur la machine de développement Windows. Pour comprendre ce que nous avons automatisé grâce à l’image Docker, consultez [Créer des certificats de démonstration pour tester des fonctionnalités d’appareils IoT Edge](how-to-create-test-certificates.md).

1. Connectez-vous à votre machine virtuelle de développement.
1. Créez un dossier avec le chemin et le nom **c:\edgeCertificates**.

1. S’il n’est pas déjà en cours d’exécution, démarrez **Docker pour Windows** à partir du menu Démarrer de Windows.

1. Ouvrez Visual Studio Code.

1. Sélectionnez **Fichier** > **Ouvrir le dossier** et choisissez **C:\\source\\IoTEdgeAndMlSample\\CreateCertificates**.

1. Dans le volet **Explorer**, cliquez avec le bouton droit sur **dockerfile** et sélectionnez **Générer l’image**.

1. Dans la boîte de dialogue, acceptez la valeur par défaut du nom et de l’étiquette de l’image : **createcertificates: latest**.

    ![Capture d’écran montrant la création de certificats dans Visual Studio Code.](media/tutorial-machine-learning-edge-05-configure-edge-device/create-certificates.png)

1. Attendez la fin de la génération.

    > [!NOTE]
    > Vous pouvez voir un avertissement indiquant qu’une clé publique est manquante. Vous pouvez ignorer cet avertissement. De même, vous verrez un avertissement de sécurité qui vous recommande de vérifier ou de réinitialiser les autorisations sur votre image. Vous pouvez l’ignorer pour cette image.

1. Dans la fenêtre de terminal Visual Studio Code, exécutez le conteneur createcertificates.

    ```cmd
    docker run --name createcertificates --rm -v c:\edgeCertificates:/edgeCertificates createcertificates /edgeCertificates
    ```

1. Docker vous invite à accéder au lecteur **c:\\** . Sélectionnez **Partager**.

1. À l’invite, indiquez vos informations d’identification.

1. Une fois que le conteneur a fini son exécution, recherchez les fichiers suivants dans **c:\\edgeCertificates** :

    * c:\\edgeCertificates\\certs\\azure-iot-test-only.root.ca.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device-full-chain.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pfx
    * c:\\edgeCertificates\\private\\new-edge-device.key.pem

## <a name="upload-certificates-to-azure-key-vault"></a>Charger des certificats dans Azure Key Vault

Pour stocker nos certificats de manière sécurisée et les rendre accessibles sur plusieurs appareils, nous les chargeons dans Azure Key Vault. Comme vous pouvez le voir dans la liste précédente, nous avons deux types de fichiers de certificat : PFX et PEM. Nous traiterons les fichiers PFX comme des certificats Key Vault à charger dans Key Vault. Les fichiers PEM sont en texte brut, et nous les traiterons comme des secrets Key Vault. Nous utiliserons l’instance de Key Vault associée à l’espace de travail Azure Machine Learning que nous avons créé en exécutant les [notebooks Jupyter](tutorial-machine-learning-edge-04-train-model.md#run-the-jupyter-notebooks).

1. Depuis le [portail Azure](https://portal.azure.com), accédez à votre espace de travail Azure Machine Learning.

1. Dans la page de présentation de l’espace de travail Azure Machine Learning, recherchez le nom pour **Coffre de clés**.

    ![Capture d’écran montrant la copie du nom du coffre de clés.](media/tutorial-machine-learning-edge-05-configure-edge-device/find-key-vault-name.png)

1. Sur votre machine de développement, chargez les certificats dans Key Vault. Remplacez **\<subscriptionId\>** et **\<keyvaultname\>** par vos informations de ressource.

    ```powershell
    c:\source\IoTEdgeAndMlSample\CreateCertificates\upload-keyvaultcerts.ps1 -SubscriptionId <subscriptionId> -KeyVaultName <keyvaultname>
    ```

1. Si vous y êtes invité, connectez-vous à Azure.

1. Le script s’exécute pendant quelques minutes et génère la liste des nouvelles entrées Key Vault.

    ![Capture d’écran qui montre la sortie du script Key Vault.](media/tutorial-machine-learning-edge-05-configure-edge-device/key-vault-entries-output.png)

## <a name="register-an-iot-edge-device"></a>Enregistrer un appareil IoT Edge

Pour connecter un appareil Azure IoT Edge à un hub IoT, nous devons d’abord inscrire l’appareil auprès du hub. Nous prenons la chaîne de connexion de l’identité d’appareil dans le cloud et l’utilisons pour configurer le runtime sur notre appareil IoT Edge. Une fois que l’appareil configuré se connecte au hub, nous pouvons déployer des modules et envoyer des messages. Nous pouvons aussi modifier la configuration de l’appareil IoT Edge physique en changeant son identité d’appareil correspondante dans IoT Hub.

Pour ce tutoriel, nous allons inscrire la nouvelle identité d’appareil à l’aide de Visual Studio Code. Vous pouvez également effectuer ces étapes en utilisant le portail Azure ou Azure CLI. Quelle que soit la méthode choisie, il est important de récupérer la chaîne de connexion de votre appareil IoT Edge. La chaîne de connexion de l’appareil se trouve dans la page de détails de votre appareil sur le portail Azure.

1. Sur votre machine de développement, ouvrez Visual Studio Code.

1. Développez le cadre **Azure IoT Hub** dans la vue **Explorer** de Visual Studio Code.

1. Cliquez sur le bouton de sélection et sélectionnez **Créer un appareil IoT Edge**.

1. Donnez un nom à l’appareil. Pour des raisons pratiques, nous utilisons le nom **aaTurbofanEdgeDevice** afin qu’il s’affiche en haut de la liste des appareils.

1. Le nouvel appareil apparaît dans la liste des appareils.

    ![Capture d’écran montrant une vue de l’appareil dans Visual Studio Code Explorer.](media/tutorial-machine-learning-edge-05-configure-edge-device/iot-hub-devices-list.png)

## <a name="deploy-an-azure-virtual-machine"></a>Déployer une machine virtuelle Azure

Nous utilisons une machine virtuelle Ubuntu 18.04 LTS sur laquelle est installé et configuré le runtime IoT Edge. Le déploiement utilise un [modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md) qui se trouve dans le dépôt du projet [iotedge-vm-deploy](https://github.com/Azure/iotedge-vm-deploy). Il provisionne l’appareil IoT Edge que vous avez inscrit à l’étape précédente à l’aide de la chaîne de connexion que vous avez fournie dans le modèle.

Vous pouvez déployer la machine virtuelle à l’aide du portail Azure ou d’Azure CLI. Nous allons vous montrer les étapes à effectuer dans le portail Azure. Pour plus d’informations, consultez [Exécuter Azure IoT Edge sur des machines virtuelles Ubuntu](how-to-install-iot-edge-ubuntuvm.md).

### <a name="deploy-using-deploy-to-azure-button"></a>Déployer à l’aide du Déployer sur Azure

1. Si vous souhaitez utiliser le modèle ARM `iotedge-vm-deploy` pour déployer votre machine virtuelle Ubuntu 18.04 LTS, cliquez sur le bouton ci-dessous :

    [![Bouton déployer sur Azure pour iotedge-vm-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fiotedge-vm-deploy%2Fmaster%2FedgeDeploy.json)

1. Dans la fenêtre qui vient de s’ouvrir, renseignez les champs de formulaire disponibles.

   | Champ | Description |
   | - | - |
   | **Abonnement** | Abonnement Azure actif dans lequel déployer la machine virtuelle. |
   | **Groupe de ressources** | Groupe de ressources existant ou nouvellement créé pour contenir la machine virtuelle et ses ressources associées. |
   | **DNS Label Prefix** | Valeur obligatoire de votre choix, utilisée pour préfixer le nom d’hôte de la machine virtuelle. |
   | **Nom de l’utilisateur administrateur** | Nom de l’utilisateur qui sera doté de privilèges root sur le déploiement. |
   | **Chaîne de connexion de l’appareil** | [Chaîne de connexion d’appareil](./how-to-register-device.md) pour un appareil créé dans votre [IoT Hub](../iot-hub/about-iot-hub.md) prévu. |
   | **Taille de machine virtuelle** | [Taille](../cloud-services/cloud-services-sizes-specs.md) de la machine virtuelle à déployer
   | **Version du système d’exploitation Ubuntu** | Version du système d’exploitation Ubuntu à installer sur la machine virtuelle de base. |
   | **Lieu** | [Région géographique](https://azure.microsoft.com/global-infrastructure/locations/) dans laquelle déployer la machine virtuelle. Par défaut, il s’agit de l’emplacement du groupe de ressources sélectionné. |
   | **Type d'authentification** | Choisissez **sshPublicKey** ou **mot de passe** selon votre préférence. |
   | **Mot de passe ou clé d’administrateur** | Valeur de la clé publique SSH ou du mot de passe en fonction du choix du type d’authentification. |

1. Lorsque tous les champs sont renseignés, cochez la case située au bas de la page pour accepter les conditions d’utilisation? Ensuite, sélectionnez **Vérifier + créer** puis **Créer** pour commencer le déploiement.

1. Dans le portail Azure, accédez à votre machine virtuelle. Vous pouvez la trouver par le biais de votre groupe de ressources ou en sélectionnant **Machines virtuelles** sous **Services Azure** dans la page d’accueil du portail.

1. Notez le **nom DNS** de votre machine virtuelle. Vous en aurez besoin pour vous connecter à votre machine virtuelle.

## <a name="connect-to-your-iot-edge-device"></a>Se connecter à votre appareil IoT Edge

1. Ouvrez une invite de commandes et utilisez la commande suivante pour vous connecter à votre machine virtuelle. Entrez votre nom d’utilisateur et votre nom DNS en fonction de la section précédente.

    ```bash
    ssh <adminUsername>@<DNS_name>
    ```

1. Quand vous êtes invité à valider l’authenticité de l’hôte, entrez **yes** (oui) et sélectionnez **Enter** (Entrée).

1. Quand vous y êtes invité, indiquez votre mot de passe.

1. Ubuntu affiche un message de bienvenue, puis une invite de type `<username>@<machinename>:~$`.

## <a name="download-key-vault-certificates"></a>Télécharger les certificats de coffre de clés

Plus haut dans cet article, nous avons chargé des certificats dans Key Vault afin de les rendre disponibles pour notre appareil IoT Edge et notre appareil de nœud terminal. L’appareil de nœud terminal est un appareil en aval qui utilise l’appareil IoT Edge comme passerelle pour communiquer avec IoT Hub.

Nous décrivons l’appareil de nœud terminal plus loin dans le tutoriel. Dans cette section, téléchargez les certificats sur l’appareil IoT Edge.

1. À partir de la session SSH sur la machine virtuelle Linux, connectez-vous à Azure avec Azure CLI.

    ```azurecli
    az login
    ```

1. Vous êtes invité à ouvrir un navigateur sur une page de [connexion de l’appareil Microsoft](https://microsoft.com/devicelogin) et à fournir un code unique. Vous pouvez effectuer ces étapes sur votre machine locale. Fermez la fenêtre de navigateur dès que vous êtes authentifié.

1. Quand l’authentification est correcte, la machine virtuelle Linux se connecte et liste vos abonnements Azure.

1. Sélectionnez l’abonnement Azure à utiliser pour les commandes Azure CLI.

    ```azurecli
    az account set --subscription <subscriptionId>
    ```

1. Créez un répertoire sur la machine virtuelle pour les certificats.

    ```bash
    sudo mkdir /edgeMlCertificates
    ```

1. Téléchargez les certificats que vous avez stockés dans le coffre de clés : new-edge-device-full-chain.cert.pem, new-edge-device.key.pem et azure-iot-test-only.root.ca.cert.pem.

    ```azurecli
    key_vault_name="<key vault name>"
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-full-chain-cert-pem -f /edgeMlCertificates/new-edge-device-full-chain.cert.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-key-pem -f /edgeMlCertificates/new-edge-device.key.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name azure-iot-test-only-root-ca-cert-pem -f /edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem
    ```

## <a name="update-the-iot-edge-device-configuration"></a>Mettre à jour la configuration de l’appareil IoT Edge

Le runtime IoT Edge utilise le fichier /etc/iotedge/config.yaml pour conserver sa configuration. Nous devons mettre à jour trois informations dans ce fichier :

* **Certificats** : Certificats à utiliser pour les connexions établies avec les appareils en aval
* **Nom d’hôte** : Nom de domaine complet (FQDN) de l’appareil IoT Edge de la machine virtuelle.

Mettez à jour les certificats et le nom d’hôte en modifiant directement le fichier config.yaml.

1. Ouvrez le fichier config.yaml.

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

1. Mettez à jour la section des certificats du fichier config.yaml en supprimant le **#** du début et en définissant le chemin pour que le fichier ressemble à l’exemple suivant :

    ```yaml
    certificates:
      device_ca_cert: "/edgeMlCertificates/new-edge-device-full-chain.cert.pem"
      device_ca_pk: "/edgeMlCertificates/new-edge-device.key.pem"
      trusted_ca_certs: "/edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem"
    ```

    Vérifiez que la ligne **certificates:** n’est pas précédée d’un espace blanc et que chaque certificat imbriqué est mis en retrait de deux espaces.

    Dans nano, cliquez avec le bouton droit pour coller le contenu de votre Presse-papiers à la position actuelle du curseur. Pour remplacer la chaîne, utilisez les touches de direction pour accéder à la chaîne à remplacer, supprimez la chaîne et cliquez avec le bouton droit pour coller à partir de la mémoire tampon.

1. Dans le portail Azure, accédez à votre machine virtuelle. Copiez le nom DNS (FQDN de la machine) à partir de la section **Vue d’ensemble**.

1. Collez le FQDN dans la section du nom d’hôte du fichier config.yaml. Vérifiez que le nom est tout en minuscules.

    ```yaml
    hostname: '<machinename>.<region>.cloudapp.azure.com'
    ```

1. Enregistrez et fermez le fichier en sélectionnant les touches **Ctrl+X**, **Y**, puis **Entrée**.

1. Redémarrez le démon IoT Edge.

    ```bash
    sudo systemctl restart iotedge
    ```

1. Vérifiez l’état du démon IoT Edge. Après la commande, entrez **:q** pour quitter.

    ```bash
    systemctl status iotedge
    ```

## <a name="troubleshooting"></a>Dépannage

Si vous voyez des erreurs (texte en couleur avec le préfixe « \[ERROR\] ») dans l’état, examinez les journaux du démon pour avoir des informations détaillées sur l’erreur.

   ```bash
   journalctl -u iotedge --no-pager --no-full
   ```

Pour plus d’informations sur la résolution des erreurs, consultez la page de [résolution des problèmes](troubleshoot.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Ce tutoriel fait partie d’une série où chaque article s’appuie sur le travail effectué dans les articles précédents. Ne nettoyez pas les ressources avant d’avoir terminé le dernier tutoriel.

## <a name="next-steps"></a>Étapes suivantes

Nous venons de terminer la configuration d’une machine virtuelle Azure comme passerelle transparente d’IoT Edge. Nous avons d’abord généré des certificats de test que nous avons chargés dans Key Vault. Ensuite, nous avons utilisé un script et un modèle Resource Manager pour déployer la machine virtuelle avec l’image Ubuntu Server 16.04 LTS + runtime Azure IoT Edge de Place de marché Azure. À l’aide de la machine virtuelle opérationnelle, nous nous sommes connectés via SSH. Ensuite, nous nous sommes connectés à Azure et avons téléchargé des certificats à partir de Key Vault. Nous avons effectué plusieurs mises à jour de la configuration du runtime IoT Edge en mettant à jour le fichier config.yaml.

Passez à l’article suivant pour générer des modules IoT Edge.

> [!div class="nextstepaction"]
> [Créer et déployer des modules IoT Edge personnalisés](tutorial-machine-learning-edge-06-custom-modules.md)
