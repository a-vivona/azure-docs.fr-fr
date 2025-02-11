---
title: 'Échec de la requête Azure Digital Twins avec l’état : 403 (Interdit)'
description: 'Causes et résolutions du message « Échec de la demande de service. État : 403 (Interdit) » sur Azure Digital Twins.'
ms.service: digital-twins
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.date: 8/20/2021
ms.openlocfilehash: b3ad9c84e35483cf81bde83703b01ef0ff3d8a9d
ms.sourcegitcommit: 2da83b54b4adce2f9aeeed9f485bb3dbec6b8023
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/24/2021
ms.locfileid: "122772329"
---
# <a name="service-request-failed-status-403-forbidden"></a>Échec de la demande de service. État : 403 (Interdit)

Cet article décrit les causes et les étapes de résolution relatives à la réception d’une erreur 403 des demandes de service à Azure Digital Twins. 

## <a name="symptoms"></a>Symptômes

Cette erreur peut se produire sur de nombreux types de demandes de service qui requièrent une authentification. L’effet est que la demande d’API échoue, renvoyant un état d’erreur `403 (Forbidden)`.

## <a name="causes"></a>Causes

### <a name="cause-1"></a>Cause no 1

La plupart du temps, cette erreur indique que vos autorisations de contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour le service ne sont pas configurées correctement. De nombreuses actions pour une instance Azure Digital Twins nécessitent que vous disposiez du rôle Propriétaire de données Azure Digital Twins **sur l’instance que vous essayez de gérer**. 

### <a name="cause-2"></a>Cause no 2

Si vous utilisez une application cliente pour communiquer avec Azure Digital Twins qui s’authentifie avec une [inscription d’application](./how-to-create-app-registration-portal.md), cette erreur peut se produire si l’inscription de votre application n’a pas d’autorisations configurées pour le service Azure Digital Twins.

L’inscription de l’application doit avoir des autorisations d’accès configurées pour les API d’Azure Digital Twins. Ensuite, lorsque votre application cliente s’authentifie auprès de l’inscription d’application, elle recevra les autorisations configurées par l’inscription d’application.

## <a name="solutions"></a>Solutions

### <a name="solution-1"></a>Solution no 1

La première solution consiste à vérifier que votre utilisateur Azure dispose du rôle Propriétaire de données Azure Digital Twins sur l’instance que vous tentez de gérer. Si vous n’avez pas ce rôle, configurez-le.

Ce rôle est différent de...
* l’ancien nom de ce rôle pendant la préversion, Propriétaire Azure Digital Twins (préversion). Dans ce cas, le rôle est le même, mais le nom a changé.
* du rôle Propriétaire sur l’ensemble de l’abonnement Azure. Propriétaire de données Azure Digital Twins est un rôle au sein d’Azure Digital Twins qui est étendu à cette instance individuelle d’Azure Digital Twins.
* du rôle Propriétaire dans Azure Digital Twins. Il s’agit de deux rôles de gestion distincts pour Azure Digital Twins et Propriétaire de données Azure Digital Twins est le rôle qui doit être utilisé pour la gestion.

#### <a name="check-current-setup"></a>Vérifier la configuration actuelle

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

#### <a name="fix-issues"></a>Corriger les problèmes 

Si vous n’avez pas cette attribution de rôle, une personne disposant d’un rôle Propriétaire dans votre **abonnement Azure** doit exécuter la commande suivante pour donner à votre utilisateur Azure le rôle Propriétaire de données Azure Digital Twins sur l’**instance Azure Digital Twins**. 

Si vous êtes propriétaire de l’abonnement, vous pouvez exécuter cette commande vous-même. Dans le cas contraire, contactez un propriétaire pour qu’il l’exécute à votre place.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<your-Azure-AD-email>" --role "Azure Digital Twins Data Owner"
```

Pour plus d’informations sur cette exigence de rôle et le processus d’attribution, consultez la [section Configurer les autorisations d’accès de votre utilisateur](how-to-set-up-instance-CLI.md#set-up-user-access-permissions) de l’article *Guide pratique : Configurer une instance et une authentification (CLI ou portail)* .

Si vous disposez déjà d’une attribution de rôle **et** que vous utilisez une inscription d’application Azure AD pour authentifier une application cliente, vous pouvez passer à la solution suivante si cette solution n’a pas résolu le problème 403.

### <a name="solution-2"></a>Solution no 2

Si vous utilisez ne inscription d’application Azure AD pour authentifier une application cliente, la deuxième solution possible consiste à vérifier que l’inscription d’application dispose d’autorisations configurées pour le service Azure Digital Twins. Si ce n’est pas le cas, configurez-les.

#### <a name="check-current-setup"></a>Vérifier la configuration actuelle

Pour vérifier si les autorisations ont été correctement configurées, accédez à la [page de vue d’ensemble des inscriptions d’applications Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) dans le portail Azure. Vous pouvez accéder à cette page vous-même en recherchant **Inscriptions d’applications** dans la barre de recherche du portail.

Basculez sur l’onglet **Toutes les applications** pour voir toutes les inscriptions d’application qui ont été créées dans votre abonnement.

Vous devez voir apparaître l’inscription d’application que vous venez de créer dans la liste. Sélectionnez-la pour afficher ses détails.

:::image type="content" source="media/troubleshoot-error-403/app-registrations.png" alt-text="Capture d’écran de la page des inscriptions d’applications dans le portail Azure":::

Tout d’abord, vérifiez que les paramètres des autorisations Azure Digital Twins ont été correctement définis sur l’inscription : sélectionnez **Manifeste** dans la barre de menus pour afficher le code du manifeste de l’inscription de l’application. Faites défiler la fenêtre de code vers le bas et recherchez ces champs sous `requiredResourceAccess`. Les valeurs doivent correspondre à celles de la capture d’écran ci-dessous :

:::image type="content" source="media/troubleshoot-error-403/verify-manifest.png" alt-text="Capture d’écran du manifeste pour l’inscription de l’application Azure AD dans le portail Azure":::

Ensuite, sélectionnez **Autorisations de l’API** dans la barre de menus pour vérifier que cette inscription d’application contient des autorisations de lecture/écriture pour Azure Digital Twins. Vous devriez obtenir une sortie similaire à celle-ci :

:::image type="content" source="media/troubleshoot-error-403/verify-api-permissions.png" alt-text="Capture d’écran des autorisations d’API pour l’inscription d’application Azure AD dans le portail Azure, montrant « Accès en lecture/écriture » pour Azure Digital Twins":::

#### <a name="fix-issues"></a>Corriger les problèmes

Si l’un de ces éléments semble différent de ce qui est décrit, suivez les instructions de configuration d’une inscription d’application dans [Créer une inscription d’application](./how-to-create-app-registration-portal.md).

## <a name="next-steps"></a>Étapes suivantes

Lisez les étapes de configuration relatives à la création et à l’authentification d’une nouvelle instance Azure Digital Twins :
* [Configurer une instance et l’authentification (CLI)](how-to-set-up-instance-cli.md)

En savoir plus sur la sécurité et les autorisations sur Azure Digital Twins :
* [Sécurité pour les solutions Azure Digital Twins](concepts-security.md)