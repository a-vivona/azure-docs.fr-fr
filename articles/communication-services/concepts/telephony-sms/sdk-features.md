---
title: Vue d’ensemble du kit SDK SMS pour Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Fournit une vue d’ensemble du kit SDK SMS et de ses offres.
author: probableprime
manager: chpalm
services: azure-communication-services
ms.author: prakulka
ms.date: 06/30/2021
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: 5e30f7741352945fb634b2f8066a47f2c47c879b
ms.sourcegitcommit: 2eac9bd319fb8b3a1080518c73ee337123286fa2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2021
ms.locfileid: "123258135"
---
# <a name="sms-sdk-overview"></a>Vue d’ensemble du kit SDK SMS

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Les kits SDK SMS Azure Communication Services peuvent être utilisés pour ajouter la messagerie SMS à vos applications.

## <a name="sms-sdk-capabilities"></a>Fonctionnalités du kit SDK SMS

La liste suivante présente l’ensemble des fonctionnalités actuellement disponibles dans nos kits SDK.

| Groupe de fonctionnalités | Fonctionnalité                                                                            | JS  | Java | .NET | Python |
| ----------------- | ------------------------------------------------------------------------------------- | --- | ---- | ---- | ------ |
| Fonctionnalités principales | Envoyer et recevoir des messages SMS                                                         | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Activer des rapports de remise pour les messages envoyés                                             | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Tous les jeux de caractères (prise en charge de la langue/Unicode)                                         | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Prise en charge des messages longs (jusqu’à 2 048 octets)                                          | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Concaténation automatique des messages longs                                                   | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Envoi des messages à plusieurs destinataires à la fois                                        | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Prise en charge de l’idempotence                                                               | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Étiquettes personnalisées pour les messages.                                                             | ✔️   | ✔️    | ✔️    | ✔️      |
| Événements            | Utiliser Event Grid pour configurer des webhooks afin de recevoir des messages entrants et des rapports de remise | ✔️   | ✔️    | ✔️    | ✔️      |
| Numéro de téléphone      | Numéros gratuits                                                                     | ✔️   | ✔️    | ✔️    | ✔️      |
| Appels RTPC      | Ajouter des fonctionnalités d’appel RTPC à votre numéro vert SMS                    | ✔️   | ✔️    | ✔️    | ✔️      |

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [ l’envoi de SMS](../../quickstarts/telephony-sms/send.md)

Les documents suivants peuvent vous intéresser :

- Se familiariser avec les [concepts SMS](../telephony-sms/concepts.md) généraux
- Obtenir un [numéro de téléphone](../../quickstarts/telephony-sms/get-phone-number.md) compatible SMS
- [Types de numéros de téléphone dans Azure Communication Services](../telephony-sms/plan-solution.md)
