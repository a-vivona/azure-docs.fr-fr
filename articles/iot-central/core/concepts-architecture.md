---
title: Concepts d’architecture dans Azure IoT Central | Microsoft Docs
description: Cet article présente les concepts clés relatifs à l’architecture d’Azure IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 12/19/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 46b8cdc7fa33c8ddd382decb49eaa148093c99fe
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122562921"
---
# <a name="azure-iot-central-architecture"></a>Architecture d’Azure IoT Central

Cet article fournit une vue d’ensemble des concepts clés de l’architecture d’Azure IoT Central.

## <a name="devices"></a>Appareils

Les appareils échangent des données avec votre application Azure IoT Central. Un appareil peut :

- Envoyer des mesures, comme de la télémétrie.
- Synchroniser des paramètres avec votre application.

Dans Azure IoT Central, les données qu’un appareil peut échanger avec votre application sont spécifiées dans un modèle d’appareil. Pour plus d’informations sur les modèles d’appareil, consultez [Modèles d’appareil](concepts-device-templates.md).

Pour plus d’informations sur la façon dont les appareils se connectent à votre application Azure IoT Central, consultez [Connectivité des appareils](concepts-get-connected.md).

## <a name="azure-iot-edge-devices"></a>Appareil Azure IoT Edge

En plus des appareils créés à l’aide des [kits de développement logiciel (SDK) Azure IoT](https://github.com/Azure/azure-iot-sdks), vous pouvez également connecter des [appareils Azure IoT Edge](../../iot-edge/about-iot-edge.md) à une application IoT Central. IoT Edge vous permet d’exécuter l’intelligence Cloud et une logique personnalisée directement sur les appareils IoT gérés par IoT Central. Le runtime IoT Edge vous permet d’effectuer les opérations suivantes :

- Installer et mettre à jour des charges de travail sur l’appareil.
- Tenir à jour les normes de sécurité IoT Edge sur l’appareil.
- Garantir que les modules IoT Edge sont toujours en cours d’exécution.
- Envoyer des rapports d’intégrité du module dans le cloud pour la supervision à distance.
- Gérer la communication entre les appareils de nœud terminal en aval et un appareil IoT Edge, entre les modules sur un appareil IoT Edge, ainsi qu’entre un appareil IoT Edge et le cloud.

![Azure IoT Central avec Azure IoT Edge](./media/concepts-architecture/iotedge.png)

IoT Central offre les fonctionnalités suivantes sur les appareils IoT Edge :

- Modèles d’appareils pour décrire les fonctionnalités d'un appareil IoT Edge, par exemple :
  - Capacité de chargement du manifeste de déploiement, qui vous aide à gérer un manifeste pour une flotte d’appareils.
  - Modules qui fonctionnent sur l’appareil IoT Edge.
  - La télémétrie envoyée par chaque module.
  - Les propriétés signalées par chaque module.
  - Les commandes auxquelles chaque module répond.
  - Les relations entre un appareil de passerelle IoT Edge et un appareil en aval.
  - Les propriétés cloud qui ne sont pas stockées sur l’appareil IoT Edge.
  - Personnalisations qui modifient la façon dont l’interface utilisateur affiche les fonctionnalités de l’appareil.
  - Affichages et formulaires de l’appareil.

  Pour plus d’informations, consultez l’article [Connecter des appareils Azure IoT Edge à une application Azure IoT Central](./concepts-iot-edge.md).

- La capacité à provisionner des appareils IoT Edge à grande échelle avec le service Device Provisioning d’Azure IoT
- Règles et actions.
- Tableaux de bord et analyses personnalisés.
- Exportation continue des données de télémétrie depuis les appareils IoT Edge.

### <a name="iot-edge-device-types"></a>Types d'appareils IoT Edge

IoT Central classifie les types d’appareils IoT Edge comme suit :

- Appareils de nœud terminal. Un appareil IoT Edge peut avoir des appareils de nœud terminal en aval, mais ceux-ci ne sont pas approvisionnés dans IoT Central.
- Appareils de passerelle avec des appareils en aval. L’appareil de passerelle et les appareils en aval sont provisionnés dans IoT Central

![Vue d’ensemble d’IoT Central avec IoT Edge](./media/concepts-architecture/gatewayedge.png)

> [!NOTE]
> IoT Central ne prend actuellement pas en charge la connexion d’un appareil IoT Edge en tant qu’appareil en aval à une passerelle IoT Edge. Cela est dû au fait que tous les appareils qui se connectent à IoT Central sont provisionnés à l’aide du service Device Provisioning Service (DPS), lequel ne prend pas en charge les scénarios IoT Edge imbriqués.

### <a name="iot-edge-patterns"></a>Modèles IoT Edge

IoT Central prend en charge les modèles d’appareils IoT Edge suivants :

#### <a name="iot-edge-as-leaf-device"></a>Appareil de nœud terminal IoT Edge

![Appareil de nœud terminal IoT Edge](./media/concepts-architecture/edgeasleafdevice.png)

L’appareil IoT Edge est approvisionné dans IoT Central et dans tous les appareils en aval, et leur télémétrie est représentée comme provenant de l’appareil IoT Edge. Les appareils en aval connectés à l’appareil IoT Edge ne sont pas approvisionnés dans IoT Central.

#### <a name="iot-edge-gateway-device-connected-to-downstream-devices-with-identity"></a>Appareil de passerelle IoT Edge connecté aux appareils en aval avec une identité

![IoT Edge avec identité d'appareil en aval](./media/concepts-architecture/edgewithdownstreamdeviceidentity.png)

L’appareil IoT Edge est approvisionné dans IoT Central avec les appareils en aval connectés à l’appareil IoT Edge. Le runtime pour l’approvisionnement des appareils en aval via la passerelle n'est pas pris en charge actuellement.

#### <a name="iot-edge-gateway-device-connected-to-downstream-devices-with-identity-provided-by-the-iot-edge-gateway"></a>Appareil de passerelle IoT Edge connecté aux appareils en aval avec une identité fournie par la passerelle IoT Edge

![IoT Edge avec appareil en aval sans identité](./media/concepts-architecture/edgewithoutdownstreamdeviceidentity.png)

L’appareil IoT Edge est approvisionné dans IoT Central avec les appareils en aval connectés à l’appareil IoT Edge. Le runtime de la passerelle fournissant l’identité aux appareils en aval et le provisionnement des appareils en aval ne sont actuellement pas pris en charge. Si vous apportez votre propre module de traduction d’identité, IoT Central peut prendre en charge ce modèle.

## <a name="cloud-gateway"></a>Passerelle cloud

Azure IoT Central utilise Azure IoT Hub comme passerelle cloud pour la connectivité des appareils. IoT Hub permet :

- L’ingestion de données à grande échelle dans le cloud.
- La gestion des appareils.
- La connectivité sécurisée des appareils.

Pour plus d’informations sur IoT Hub, consultez [Azure IoT Hub](../../iot-hub/index.yml).

Pour plus d’informations sur la connectivité des appareils dans Azure IoT Central, consultez [Connectivité des appareils](concepts-get-connected.md).

## <a name="data-stores"></a>Magasins de données

Azure IoT Central stocke les données d’application dans le cloud. Les données d’application stockées incluent :

- Les modèles d’appareil.
- Les identités des appareils.
- Les métadonnées des appareils.
- Les données des utilisateurs et des rôles.

Azure IoT Central utilise un magasin de séries chronologiques pour les données de mesure envoyées depuis vos appareils. Données de séries chronologiques provenant d’appareils utilisés par le service d’analytique.

## <a name="data-export"></a>Exportation de données

Dans une application Azure IoT Central, vous pouvez [exporter en continu vos données](howto-export-data.md) vers votre propre Azure Event Hubs et les instances Azure Service Bus. Vous pouvez également régulièrement exporter vos données vers votre compte de stockage d’objets blob Azure. IoT Central peut exporter des mesures, des appareils et des modèles d’appareil.

## <a name="batch-device-updates"></a>Mises à jour d’appareils par lots

Dans une application Azure IoT Central, vous pouvez [créer et exécuter des travaux](howto-manage-devices-in-bulk.md) pour gérer des appareils connectés. Ces travaux vous permettent d’effectuer des mises à jour en bloc sur les propriétés ou les paramètres des appareils, ou d’exécuter des commandes. Par exemple, vous pouvez créer un travail pour augmenter la vitesse du ventilateur pour plusieurs distributeurs réfrigérés.

## <a name="role-based-access-control-rbac"></a>Contrôle d’accès en fonction du rôle

Chaque application IoT Central possède son propre système RBAC intégré. Un [administrateur peut définir des règles d’accès](howto-manage-users-roles.md) pour une application Azure IoT Central en utilisant un des rôles prédéfinis ou en créant un rôle personnalisé. Les rôles déterminent les zones de l’application auxquelles un utilisateur a accès ainsi que les actions qu’il peut effectuer.

## <a name="security"></a>Sécurité

Les fonctionnalités de sécurité dans Azure IoT Central sont les suivantes :

- Les données sont chiffrées en transit et au repos.
- L’authentification est fournie par Azure Active Directory ou via un compte Microsoft. L’authentification à deux facteurs est prise en charge.
- L’isolation complète des locataires.
- La sécurité au niveau des appareils.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert l’architecture d’Azure IoT Central, l’étape suivante suggérée est de découvrir la [connectivité des appareils](concepts-get-connected.md) dans Azure IoT Central.