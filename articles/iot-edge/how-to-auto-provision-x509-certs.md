---
title: Provisionner automatiquement des appareils avec le service Device Provisioning à l’aide de certificats X.509 – Azure IoT Edge | Microsoft Docs
description: Utiliser des certificats X.509 afin de tester le provisionnement automatique des appareils pour Azure IoT Edge avec le service Device Provisioning
author: kgremban
ms.author: kgremban
ms.date: 06/18/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: contperf-fy21q2
ms.openlocfilehash: 4095207fbc4fbbf4cb3f6463e3091717fcd3921a
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122532256"
---
# <a name="create-and-provision-an-iot-edge-device-using-x509-certificates"></a>Créer et provisionner un appareil IoT Edge à l’aide de certificats X.509

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Avec le [service Azure IoT Hub Device Provisioning (DPS)](../iot-dps/index.yml), vous pouvez provisionner automatiquement des appareils IoT Edge à l’aide de certificats X.509. Si vous ne connaissez pas le processus de provisionnement automatique, révisez la présentation du [provisionnement](../iot-dps/about-iot-dps.md#provisioning-process) avant de poursuivre.

Cet article vous montre comment créer une inscription au service Device Provisioning à l’aide de certificats X.509 sur un appareil IoT Edge, en procédant comme suit :

* Génération de certificats et de clés
* Création d’une inscription individuelle pour un appareil ou d’une inscription de groupe pour un ensemble d’appareils
* Installation du runtime IoT Edge et inscription de l’appareil auprès d’IoT Hub

Utiliser des certificats X.509 comme mécanisme d’attestation est un excellent moyen de mettre à l’échelle la production et de simplifier le provisionnement des appareils. Les certificats X.509 sont généralement organisés en une chaîne d’approbation de confiance. À partir d’un certificat racine auto-signé ou approuvé, chaque certificat de la chaîne signe le certificat inférieur suivant. Ce modèle crée une chaîne déléguée de confiance depuis le certificat racine jusqu’au certificat « feuille » final installé sur un appareil, en passant par chaque certificat intermédiaire.

## <a name="prerequisites"></a>Prérequis

* Un hub IoT actif.
* Un appareil physique ou virtuel faisant office d’appareil IoT Edge.
* La dernière version de [Git](https://git-scm.com/download/) installée.
* Une instance du service IoT Hub Device Provisioning dans Azure, liée à votre hub IoT.
  * Si vous ne disposez pas d’une instance du service Device Provisioning, suivez les instructions indiquées dans [Configurer le service IoT Hub Device Provisioning](../iot-dps/quick-setup-auto-provision.md).
  * Après avoir lancé l’exécution du service Device Provisioning, copiez la valeur de **Étendue de l’ID** à partir de la page de présentation. Vous utilisez cette valeur lorsque vous configurez le runtime IoT Edge.

## <a name="generate-device-identity-certificates"></a>Générer des certificats d’identité d’appareil

Le certificat d’identité d’appareil est un certificat feuille qui se connecte par le biais d’une chaîne de certificats de confiance au certificat d’autorité de certification X.509 supérieur. Le nom commun (CN) du certificat d’identité d’appareil doit être défini sur l’ID d’appareil que vous souhaitez que l’appareil ait dans votre hub IoT.

Les certificats d’identité d’appareil sont utilisés uniquement pour provisionner l’appareil IoT Edge et l’authentifier auprès d’Azure IoT Hub. Ils ne signent pas de certificats, contrairement aux certificats d’autorité de certification que l’appareil IoT Edge présente aux modules ou aux appareils de nœud terminal à des fins de vérification. Pour plus d’informations, consultez les [détails sur l’utilisation des certificats par Azure IoT Edge](iot-edge-certs.md).

Après avoir créé le certificat d’identité d’appareil, vous devez avoir deux fichiers : un fichier .cer ou .pem, qui contient la partie publique du certificat, et un fichier .cer ou .pem, qui comporte la clé privée du certificat. Si vous envisagez d’utiliser l’inscription de groupe dans le service Device Provisioning, vous avez également besoin de la partie publique d’un certificat d’autorité de certification racine ou intermédiaire dans la même chaîne de certificats de confiance.

Les fichiers suivants sont nécessaires pour configurer le provisionnement automatique avec X.509 :

* Le certificat d’identité d’appareil et son certificat de clé privée. Le certificat d’identité d’appareil est chargé dans DPS si vous créez une inscription individuelle. La clé privée est transmise au runtime IoT Edge.
* Un certificat de chaîne complète, qui doit comporter au moins l’identité d’appareil et les certificats intermédiaires qu’elle contient. Le certificat de chaîne complète est transmis au runtime IoT Edge.
* Un certificat d’autorité de certification racine ou intermédiaire issu de la chaîne d’approbation des certificats. Ce certificat est chargé dans DPS si vous créez une inscription de groupe.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
> [!NOTE]
> Il existe actuellement dans libiothsm une limitation empêchant l’utilisation de certificats qui expirent le 1er janvier 2038 ou après cette date.

:::moniker-end

### <a name="use-test-certificates-optional"></a>Utiliser des certificats de test (facultatif)

Si vous n’avez pas d’autorité de certification disponible pour créer des certificats d’identité et que vous souhaitez tester ce scénario, le dépôt Git Azure IoT Edge contient des scripts que vous pouvez utiliser pour générer des certificats de test. Ces certificats, conçus uniquement pour les tests de développement, ne doivent pas être utilisés en production.

Pour créer des certificats de test, suivez les étapes décrites dans [Créer des certificats de démonstration pour tester les fonctionnalités de l’appareil IoT Edge](how-to-create-test-certificates.md). Effectuez les deux sections nécessaires pour configurer les scripts de génération de certificat et créer un certificat d’autorité de certification racine. Ensuite, suivez les étapes permettant de créer un certificat d’identité d’appareil. Quand vous avez terminé, vous devez disposer de la chaîne de certificats et de la paire de clés suivantes :

Linux :

* `<WRKDIR>/certs/iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>/private/iot-edge-device-identity-<name>.key.pem`

Windows :

* `<WRKDIR>\certs\iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>\private\iot-edge-device-identity-<name>.key.pem`

Vous avez besoin de ces deux certificats sur l’appareil IoT Edge. Si vous envisagez d’utiliser l’inscription individuelle dans le service Device Provisioning, vous devez charger le fichier .cert.pem. Si vous envisagez d’utiliser l’inscription de groupe dans le service Device Provisioning, la même chaîne de certificats de confiance doit également comporter un certificat d’autorité de certification racine ou intermédiaire à charger. Si vous recourez à des certificats de démonstration, utilisez le certificat `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem` pour l’inscription de groupe.

## <a name="create-a-dps-individual-enrollment"></a>Créer une inscription individuelle dans le service Device Provisioning

Utilisez vos certificats et clés générés afin de créer une inscription individuelle dans le service Device Provisioning pour un seul appareil IoT Edge. Les inscriptions individuelles prennent la partie publique du certificat d’identité d’un appareil et la font correspondre avec le certificat sur l’appareil.

Si vous envisagez de provisionner plusieurs appareils IoT Edge, suivez les étapes de la section suivante, [Créer une inscription de groupe dans le service Device Provisioning](#create-a-dps-group-enrollment).

Lorsque vous créez une inscription auprès du service Device Provisioning, vous avez la possibilité de déclarer un **État initial du jumeau d’appareil**. Dans le jumeau d’appareil, vous pouvez définir des balises pour regrouper les appareils en fonction des métriques dont vous avez besoin dans votre solution, comme la région, l’environnement, l’emplacement ou le type d’appareil. Ces balises sont utilisées pour créer [des déploiements automatiques](how-to-deploy-at-scale.md).

Pour plus d’informations sur les inscriptions dans le service Device Provisioning, consultez [Guide pratique pour gérer les inscriptions d’appareils](../iot-dps/how-to-manage-enrollments.md).

   > [!TIP]
   > Dans l’interface de ligne de commande Azure, vous pouvez créer une [inscription](/cli/azure/iot/dps/enrollment) ou un [groupe d’inscriptions](/cli/azure/iot/dps/enrollment-group) et utiliser l’indicateur **compatible avec Edge** pour spécifier qu’un appareil, ou un groupe d’appareils, est un appareil IoT Edge.

1. Dans le [Portail Microsoft Azure](https://portal.azure.com), accédez à votre instance du service IoT Hub Device Provisioning.

1. Sous **Paramètres**, sélectionnez **Gérer les inscriptions**.

1. Sélectionnez **Ajouter une inscription individuelle** et suivez ces étapes pour configurer l’inscription :  

   * **Mécanisme** : sélectionnez **X.509**.

   * **Fichier .pem ou .cer du certificat principal** : chargez le fichier public à partir du certificat d’identité d’appareil. Si vous avez utilisé les scripts pour générer un certificat de test, choisissez le fichier suivant :

      `<WRKDIR>/certs/iot-edge-device-identity-<name>.cert.pem`

   * **ID de l’appareil IoT Hub** : Fournissez un ID pour votre appareil si vous le souhaitez. Vous pouvez utiliser l’ID d’appareil pour cibler un appareil individuel lors du déploiement de module. Si vous ne fournissez pas d’ID d’appareil, le nom commun (CN) dans le certificat X.509 est utilisé.

   * **Appareil IoT Edge** : Sélectionnez **Vrai** pour déclarer que cette inscription est un appareil IoT Edge.

   * **Sélectionnez les hubs IoT auxquels cet appareil peut être attribué** : choisissez le hub IoT lié auquel vous voulez connecter votre appareil. Vous pouvez choisir plusieurs hubs : l’appareil sera affecté à l’un d’entre eux en fonction de la stratégie d’allocation sélectionnée.

   * **État initial du jumeau d’appareil** : ajoutez une valeur d’étiquette destinée au jumeau d’appareil si vous le souhaitez. Vous pouvez utiliser des étiquettes pour cibler des groupes d’appareils lors du déploiement automatique. Par exemple :

      ```json
      {
         "tags": {
            "environment": "test"
         },
         "properties": {
            "desired": {}
         }
      }
      ```

1. Sélectionnez **Enregistrer**.

Maintenant qu’une inscription existe pour cet appareil, le runtime IoT Edge peut provisionner automatiquement l’appareil lors de l’installation. Passez à la section [Installer le runtime IoT Edge](#install-the-iot-edge-runtime) pour configurer votre appareil IoT Edge.

## <a name="create-a-dps-group-enrollment"></a>Créer une inscription de groupe dans le service Device Provisioning

Utilisez vos certificats et clés générés afin de créer une inscription de groupe dans le service Device Provisioning pour plusieurs appareils IoT Edge. Les inscriptions de groupe utilisent un certificat d’autorité de certification intermédiaire ou racine à partir de la chaîne de certificats de confiance utilisée pour générer les certificats d’identité d’appareil individuels.

Si vous envisagez de provisionner un seul appareil IoT Edge, suivez les étapes de la section précédente, [Créer une inscription individuelle dans le service Device Provisioning](#create-a-dps-individual-enrollment).

Lorsque vous créez une inscription auprès du service Device Provisioning, vous avez la possibilité de déclarer un **État initial du jumeau d’appareil**. Dans le jumeau d’appareil, vous pouvez définir des balises pour regrouper les appareils en fonction des métriques dont vous avez besoin dans votre solution, comme la région, l’environnement, l’emplacement ou le type d’appareil. Ces balises sont utilisées pour créer [des déploiements automatiques](how-to-deploy-at-scale.md).

### <a name="verify-your-root-certificate"></a>Vérifier votre certificat racine

Quand vous créez un groupe d’inscriptions, vous avez la possibilité d’utiliser un certificat vérifié. Vous pouvez vérifier un certificat avec le service Device Provisioning en prouvant que vous êtes propriétaire du certificat racine. Pour plus d’informations, consultez [Guide pratique pour effectuer la preuve de possession de certificats d’autorité de certification X.509](../iot-dps/how-to-verify-certificates.md).

1. Dans le [Portail Microsoft Azure](https://portal.azure.com), accédez à votre instance du service IoT Hub Device Provisioning.

1. Sélectionnez **Certificats** dans le menu de gauche.

1. Sélectionnez **Ajouter** pour ajouter un nouveau certificat.

1. Entrez un nom convivial pour votre certificat, puis accédez au fichier .cer ou .pem représentant la partie publique de votre certificat X.509.

   Si vous utilisez les certificats de démonstration, chargez le certificat `<wrkdir>/certs/azure-iot-test-only.root.ca.cert.pem`.

1. Sélectionnez **Enregistrer**.

1. Votre certificat doit maintenant être listé dans la page **Certificats**. Sélectionnez-le pour ouvrir les détails du certificat.

1. Sélectionnez **Générer le code de vérification**, puis copiez le code généré.

1. Que vous ayez fourni votre propre certificat d’autorité de certification ou utilisiez les certificats de démonstration, vous pouvez utiliser l’outil de vérification disponible dans le dépôt IoT Edge pour vérifier la preuve de possession. L’outil de vérification utilise votre certificat d’autorité de certification pour signer un nouveau certificat ayant le code de vérification fourni comme nom d’objet.

   * Windows :

     ```powershell
     New-CACertsVerificationCert "<verification code>"
     ```

   * Linux :

     ```bash
     ./certGen.sh create_verification_certificate <verification code>
     ```

1. Dans le portail Azure, dans la même page des détails du certificat, chargez le certificat de vérification qui vient d’être généré.

1. Sélectionnez **Vérifier**.

### <a name="create-enrollment-group"></a>Créer un groupe d’inscriptions

Pour plus d’informations sur les inscriptions dans le service Device Provisioning, consultez [Guide pratique pour gérer les inscriptions d’appareils](../iot-dps/how-to-manage-enrollments.md).

1. Dans le [Portail Microsoft Azure](https://portal.azure.com), accédez à votre instance du service IoT Hub Device Provisioning.

1. Sous **Paramètres**, sélectionnez **Gérer les inscriptions**.

1. Sélectionnez **Ajouter un groupe d’inscriptions** et suivez les étapes ci-après pour configurer l’inscription :

   * **Nom du groupe** : fournissez un nom facile à retenir pour cette inscription de groupe.

   * **Type d’attestation** : Sélectionnez **Certificate**.

   * **Appareil IoT Edge** : sélectionnez **VRAI**. Pour une inscription de groupe, tous les appareils doivent être de IoT Edge, ou aucun d’entre eux ne peut l’être.

   * **Type de certificat** : sélectionnez **Certificat d’autorité de certification** si vous avez un certificat d’autorité de certification vérifié stocké avec le service Device Provisioning, ou **Certificat intermédiaire** si vous souhaitez charger un nouveau fichier uniquement pour cette inscription.

   * **Certificat principal** : si vous avez choisi « Certificat d’autorité de certification » dans la dernière section, choisissez votre certificat dans la liste déroulante. Si vous avez choisi « Certificat intermédiaire », chargez le fichier public à partir d’un certificat d’autorité de certification dans la chaîne de certificats de confiance utilisée pour générer les certificats d’identité d’appareil.

   * **Sélectionnez les hubs IoT auxquels cet appareil peut être attribué** : choisissez le hub IoT lié auquel vous voulez connecter votre appareil. Vous pouvez choisir plusieurs hubs : l’appareil sera affecté à l’un d’entre eux en fonction de la stratégie d’allocation sélectionnée.

   * **État initial du jumeau d’appareil** : ajoutez une valeur d’étiquette destinée au jumeau d’appareil si vous le souhaitez. Vous pouvez utiliser des étiquettes pour cibler des groupes d’appareils lors du déploiement automatique. Par exemple :

      ```json
      {
         "tags": {
            "environment": "test"
         },
         "properties": {
            "desired": {}
         }
      }
      ```

1. Sélectionnez **Enregistrer**.

Maintenant qu’une inscription existe pour cet appareil, le runtime IoT Edge peut provisionner automatiquement l’appareil lors de l’installation. Passez à la section suivante pour configurer votre appareil IoT Edge.

## <a name="install-the-iot-edge-runtime"></a>Installer le runtime IoT Edge

Le runtime IoT Edge est déployé sur tous les appareils IoT Edge. Ses composants s’exécutent dans des conteneurs et vous permettent de déployer des conteneurs supplémentaires sur l’appareil, pour que vous puissiez exécuter du code en périphérie.

<!-- 1.1 -->
:::moniker range="=iotedge-2018-06"

Suivez les étapes appropriées pour installer Azure IoT Edge en fonction de votre système d’exploitation :

* [Installer IoT Edge pour Linux](how-to-install-iot-edge.md)
* [Installer IoT Edge pour des appareils Linux sur Windows](how-to-install-iot-edge-on-windows.md)
  * Ce scénario est la méthode recommandée pour exécuter IoT Edge sur des appareils Windows.
* [Installer IoT Edge avec des conteneurs Windows](how-to-install-iot-edge-windows-on-windows.md)

Une fois que IoT Edge est installé sur votre appareil, revenez à cet article pour provisionner l’appareil.

:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

Effectuez les étapes décrites dans [Installer le runtime Azure IoT Edge](how-to-install-iot-edge.md), puis revenez au présent article pour provisionner l’appareil.

:::moniker-end
<!-- end 1.2 -->

Le provisionnement X.509 avec le service Device Provisioning est uniquement pris en charge dans IoT Edge version 1.0.9 ou ultérieure.

## <a name="configure-the-device-with-provisioning-information"></a>Configurer l’appareil avec des informations de provisionnement

Une fois que le runtime est installé sur votre appareil, configurez l’appareil avec les informations qu’il utilise pour se connecter au service Device Provisioning (DPS) et à IoT Hub.

Préparez les informations suivantes :

* La valeur **Étendue de l’ID** du service Device Provisioning. Vous pouvez récupérer cette valeur à partir de la page de présentation de votre instance du service Device Provisioning dans le portail Azure.
* Le fichier de chaîne de certificats d’identité d’appareil sur l’appareil
* Le fichier de clé d’identité d’appareil sur l’appareil

# <a name="linux"></a>[Linux](#tab/linux)

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

1. Ouvrez le fichier de configuration sur l’appareil IoT Edge.

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

1. Recherchez la section des configurations de provisionnement dans le fichier. Supprimez les marques de commentaire des lignes pour l’approvisionnement de certificat DPS X.509, et assurez-vous que les autres lignes d’approvisionnement sont commentées.

   La ligne `provisioning:` ne doit pas être précédée d’un espace et les éléments imbriqués doivent être en retrait de deux espaces.

   ```yml
   # DPS X.509 provisioning configuration
   provisioning:
     source: "dps"
     global_endpoint: "https://global.azure-devices-provisioning.net"
     scope_id: "<SCOPE_ID>"
     attestation:
       method: "x509"
   #   registration_id: "<OPTIONAL REGISTRATION ID. LEAVE COMMENTED OUT TO REGISTER WITH CN OF identity_cert>"
       identity_cert: "<REQUIRED URI TO DEVICE IDENTITY CERTIFICATE>"
       identity_pk: "<REQUIRED URI TO DEVICE IDENTITY PRIVATE KEY>"
   #  always_reprovision_on_startup: true
   #  dynamic_reprovisioning: false
   ```

1. Mettez à jour les valeurs de `scope_id`, `identity_cert` et `identity_pk` avec vos informations sur l’appareil et le service Device Provisioning.

   Quand vous ajoutez le certificat X.509 et les informations de clé au fichier config.yaml, les chemins doivent être fournis en tant qu’URI de fichier. Par exemple :

   `file:///<path>/identity_certificate_chain.pem`
   `file:///<path>/identity_key.pem`

1. Si vous le souhaitez, fournissez le `registration_id` de l'appareil, qui doit correspondre au nom commun du certificat d'identité. Si vous laissez cette ligne commentée, le nom commun est appliqué automatiquement.

1. Si vous le souhaitez, utilisez les lignes `always_reprovision_on_startup` ou `dynamic_reprovisioning` pour configurer le comportement de reprovisionnement de votre appareil. Si un appareil est configuré pour être reprovisionné au démarrage, le reprovisionnement est toujours tenté avec le service DPS dans un premier temps, puis, en cas d’échec, au moyen de la sauvegarde de provisionnement. Si un appareil est configuré pour se reprovisionner dynamiquement lui-même, IoT Edge redémarre et exécute le reprovisionnement si un événement de reprovisionnement est détecté. Pour plus d’informations, consultez [Concepts du reprovisionnement d’appareils IoT Hub](../iot-dps/concepts-device-reprovision.md).

1. Enregistrez et fermez le fichier config.yaml.

1. Redémarrez le runtime IoT Edge afin qu’il récupère toutes les modifications de configuration que vous avez effectuées sur l’appareil.

   ```bash
   sudo systemctl restart iotedge
   ```

:::moniker-end
<!-- end 1.1. -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

1. Créez un fichier de configuration pour votre appareil à partir d’un fichier de modèle fourni dans le cadre de l’installation d’IoT Edge.

   ```bash
   sudo cp /etc/aziot/config.toml.edge.template /etc/aziot/config.toml
   ```

1. Ouvrez le fichier de configuration sur l’appareil IoT Edge.

   ```bash
   sudo nano /etc/aziot/config.toml
   ```

1. Recherchez la section **Provisioning** du fichier. Supprimez les marques de commentaire des lignes pour l’approvisionnement de certificat X.509, et assurez-vous que les autres lignes d’approvisionnement sont commentées.

   ```toml
   # DPS provisioning with X.509 certificate
   [provisioning]
   source = "dps"
   global_endpoint = "https://global.azure-devices-provisioning.net"
   id_scope = "<SCOPE_ID>"
   
   [provisioning.attestation]
   method = "x509"
   registration_id = "<REGISTRATION ID>"

   identity_cert = "<DEVICE IDENTITY CERTIFICATE>"

   identity_pk = "<DEVICE IDENTITY PRIVATE KEY>"
   ```

1. Mettez à jour la valeur de `id_scope` avec l'ID d'étendue que vous avez copié à partir de votre instance de DPS.

1. Fournissez un `registration_id` pour l'appareil ; il s'agira de l'ID de l'appareil dans IoT Hub. L'ID d'inscription doit correspondre au nom commun du certificat d'identité.

1. Mettez à jour les valeurs d'`identity_cert` et `identity_pk` avec les informations de votre certificat et de votre clé.

   La valeur du certificat d’identité peut être fournie sous la forme d’un URI de fichier ou être émise de manière dynamique à l’aide de l’autorité de certification EST ou locale. Supprimez les marques de commentaire d’une seule ligne, selon le format que vous choisissez d’utiliser.

   La valeur de la clé privée d’identité peut être fournie sous la forme d’un URI de fichier ou d’un URI PKCS#11. Supprimez les marques de commentaire d’une seule ligne, selon le format que vous choisissez d’utiliser.

   Si vous utilisez des URI PKCS#11, recherchez la section **PKCS#11** dans le fichier config et fournissez des informations sur votre configuration de PKCS#11.

1. Enregistrez et fermez le fichier.

1. Appliquez les modifications de configuration que vous avez apportées à IoT Edge.

   ```bash
   sudo iotedge config apply
   ```

:::moniker-end
<!-- end 1.2 -->

# <a name="linux-on-windows"></a>[Linux sur Windows](#tab/eflow)

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

Vous pouvez utiliser PowerShell ou Windows Admin Center pour provisionner votre appareil IoT Edge.

### <a name="powershell"></a>PowerShell

Pour PowerShell, exécutez la commande suivante en mettant à jour les valeurs d’espace réservé avec vos propres valeurs :

```powershell
Provision-EflowVm -provisioningType DPSX509 -scopeId <ID_SCOPE_HERE> -identityCertPath <ABSOLUTE_CERT_DEST_PATH_ON_WINDOWS_HOST> -identityPrivKeyPath <ABSOLUTE_PRIVATE_KEY_DEST_PATH_ON_WINDOWS_HOST>
```

### <a name="windows-admin-center"></a>Windows Admin Center

Pour Windows Admin Center, procédez comme suit :

1. Dans le volet **Approvisionnement de l’appareil Azure IoT Edge**, sélectionnez **Certificat X.509 (DPS)** dans le menu déroulant des méthodes d’approvisionnement.

1. Dans le [portail Azure](https://ms.portal.azure.com/), accédez à votre instance DPS.

1. Dans l’onglet **Vue d’ensemble**, copiez la valeur du paramètre **Étendue de l’ID**. Collez-la dans le champ ID de l’étendue dans Windows Admin Center.

1. Indiquez l’ID d’inscription de votre appareil dans le champ ID d’inscription dans Windows Admin Center.

1. Chargez vos fichiers de certificat et de clé privée.

1. Sélectionnez **Approvisionnement avec la méthode sélectionnée**.

   ![Sélectionnez Approvisionnement avec la méthode sélectionnée après avoir renseigné les champs requis pour l’approvisionnement avec certificat X.509](./media/how-to-install-iot-edge-on-windows/provisioning-with-selected-method-x509-certs.png)

1. Une fois l’approvisionnement terminé, sélectionnez **Terminer**. Vous êtes redirigé vers le tableau de bord principal. Un nouvel appareil, dont le type est `IoT Edge Devices`, doit maintenant apparaître dans la liste. Vous pouvez sélectionner l’appareil IoT Edge pour vous y connecter. Une fois sur la page de **vue d’ensemble**, vous pouvez consulter la **liste des modules IoT Edge** et l’**état IoT Edge** de votre appareil.

:::moniker-end
<!-- end 1.1. -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

>[!NOTE]
>Actuellement, il n’existe pas de prise en charge d’IoT Edge version 1.2 s’exécutant sur IoT Edge pour Linux sur Windows.

:::moniker-end
<!-- end 1.2 -->

# <a name="windows"></a>[Windows](#tab/windows)

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

1. Ouvrez une fenêtre PowerShell en mode administrateur. Veillez à utiliser une session AMD64 de PowerShell lors de l'installation d'IoT Edge, plutôt que PowerShell (x86).

1. La commande **Initialize-IoTEdge** configure le runtime IoT Edge sur votre ordinateur. La commande opère par défaut un provisionnement manuel avec les conteneurs Windows. Vous devez donc ajouter l’indicateur `-DpsX509` pour utiliser le provisionnement automatique avec l’authentification par certificat X.509.

   Remplacez les valeurs d’espace réservé `{scope_id}`, `{identity cert chain path}` et `{identity key path}` par les valeurs appropriées de votre instance du service Device Provisioning et les chemins des fichiers sur votre appareil.

   Ajoutez la valeur `-RegistrationId {registration_id}` si vous souhaitez définir l’ID d’appareil autrement que par le nom commun du certificat d’identité.

   Ajoutez le paramètre `-ContainerOs Linux` si vous utilisez des conteneurs Linux sur Windows.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -DpsX509 -ScopeId {scope ID} -X509IdentityCertificate {identity cert chain path} -X509IdentityPrivateKey {identity key path}
   ```

   >[!TIP]
   >Le fichier config stocke vos informations de certificat et de clé sous forme d’URI de fichier. Toutefois, la commande Initialize-IoTEdge gère cette étape de mise en forme pour vous ; vous pouvez donc fournir le chemin absolu des fichiers de certificat et de clé sur votre appareil.

:::moniker-end
<!-- end 1.1. -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

>[!NOTE]
>Actuellement, il n’existe pas de prise en charge d’IoT Edge version 1.2 s’exécutant sur Windows.

:::moniker-end
<!-- end 1.2 -->

---

## <a name="verify-successful-installation"></a>Vérifier la réussite de l’installation

Si le runtime a démarré correctement, vous pouvez accéder à votre IoT Hub et commencer à déployer des modules IoT Edge sur votre appareil.

Vous pouvez vérifier que l’inscription individuelle que vous avez créée dans le service Device Provisioning a été utilisée. Dans le Portail Azure, accédez à l’instance du service Device Provisioning. Ouvrez les détails de l’inscription pour l’inscription individuelle que vous avez créée. Notez que l’état de l’inscription est **Affecté** et que l’ID de l’appareil figure dans la liste.

Utilisez les commandes suivantes sur votre appareil pour vérifier que IoT Edge a été installé et démarré correctement.

# <a name="linux"></a>[Linux](#tab/linux)

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

Vérifiez l’état du service IoT Edge.

```cmd/sh
systemctl status iotedge
```

Consultez les journaux d’activité de service.

```cmd/sh
journalctl -u iotedge --no-pager --no-full
```

Répertoriez les modules en cours d’exécution.

```cmd/sh
iotedge list
```

:::moniker-end

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

Vérifiez l’état du service IoT Edge.

```cmd/sh
sudo iotedge system status
```

Consultez les journaux d’activité de service.

```cmd/sh
sudo iotedge system logs
```

Répertoriez les modules en cours d’exécution.

```cmd/sh
sudo iotedge list
```

:::moniker-end

# <a name="linux-on-windows"></a>[Linux sur Windows](#tab/eflow)

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

Connectez-vous à IoT Edge pour Linux sur une machine virtuelle Windows.

```powershell
Connect-EflowVM
```

Vérifiez l’état du service IoT Edge.

```cmd/sh
sudo systemctl status iotedge
```

Consultez les journaux d’activité de service.

```cmd/sh
sudo journalctl -u iotedge --no-pager --no-full
```

Répertoriez les modules en cours d’exécution.

```cmd/sh
sudo iotedge list
```

:::moniker-end

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

>[!NOTE]
>Actuellement, il n’existe pas de prise en charge d’IoT Edge version 1.2 s’exécutant sur IoT Edge pour Linux sur Windows.

:::moniker-end
<!-- end 1.2 -->

# <a name="windows"></a>[Windows](#tab/windows)

<!-- 1.1 -->
:::moniker range="=iotedge-2018-06"

Vérifiez l’état du service IoT Edge.

```powershell
Get-Service iotedge
```

Consultez les journaux d’activité de service.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

Répertoriez les modules en cours d’exécution.

```powershell
iotedge list
```

:::moniker-end

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

>[!NOTE]
>Actuellement, il n’existe pas de prise en charge d’IoT Edge version 1.2 s’exécutant sur Windows.

:::moniker-end
<!-- end 1.2 -->

---

## <a name="next-steps"></a>Étapes suivantes

Le processus d’inscription auprès du service Device Provisioning vous permet de définir l’ID d’appareil et les balises du jumeau d’appareil en même temps que vous provisionnez le nouvel appareil. Vous pouvez utiliser ces valeurs pour cibler des appareils individuels ou des groupes d’appareils avec la gestion d’appareils automatique. En savoir plus sur [Déployer et surveiller des modules IoT Edge à grande échelle à l’aide du portail Azure](how-to-deploy-at-scale.md) ou [d’Azure CLI](how-to-deploy-cli-at-scale.md).
