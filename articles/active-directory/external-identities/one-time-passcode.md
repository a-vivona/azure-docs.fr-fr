---
title: Authentification par code secret à usage unique pour les utilisateurs invités B2B - Azure AD
description: Comment utiliser le code secret à usage unique d’e-mail pour authentifier les utilisateurs invités B2B sans avoir besoin d’un compte Microsoft.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 07/26/2021
ms.author: mimart
author: msmimart
manager: CelesteDG
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan, seoapril2019, contperf-fy21q4-portal
ms.collection: M365-identity-device-management
ms.openlocfilehash: d7b12d0611f6488a9ab5475ec6488328a764434f
ms.sourcegitcommit: bb1c13bdec18079aec868c3a5e8b33ef73200592
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/27/2021
ms.locfileid: "114721056"
---
# <a name="email-one-time-passcode-authentication"></a>Authentification par envoi d’un code secret à usage unique par e-mail

La fonctionnalité de code secret à usage unique envoyé par e-mail est un moyen d’authentifier les utilisateurs B2B Collaboration lorsqu’ils ne peuvent pas être authentifiés par d’autres moyens, par exemple Azure AD, un compte Microsoft (MSA) ou des fournisseurs d’identité sociale. Lorsqu’un utilisateur invité B2B tente d’accepter votre invitation ou de se connecter à vos ressources partagées, il peut demander un code secret temporaire, qui est envoyé à son adresse e-mail. Il doit ensuite saisir ce code secret pour se connecter.

Vous pouvez activer cette fonctionnalité à tout moment dans le portail Azure en configurant le fournisseur d’identité Code secret à usage unique envoyé par e-mail sous les paramètres External Identities de votre locataire. Vous pouvez choisir d’activer la fonctionnalité, de la désactiver ou d’attendre une activation automatique en octobre 2021.

![Diagramme de présentation de l’envoi d’un code secret à usage unique par e-mail](media/one-time-passcode/email-otp.png)

> [!IMPORTANT]
> - **À compter d’octobre 2021**, la fonctionnalité d’envoi d’un code secret à usage unique par e-mail sera activée pour tous les locataires existants et sera activée par défaut pour les nouveaux locataires. Si vous ne souhaitez pas autoriser cette fonctionnalité à s’activer automatiquement, vous pouvez la désactiver. Consultez [Désactivation de l’envoi d’un code à usage unique par e-mail](#disable-email-one-time-passcode) ci-dessous.
> - Dans le portail Azure, les paramètres de code à usage unique par e-mail ont été déplacés de **Paramètres de collaboration externe** vers **Tous les fournisseurs d’identité**.

> [!NOTE]
> Les utilisateurs d'un code secret à usage unique doivent se connecter à l'aide d'un lien incluant le contexte du locataire (par exemple, `https://myapps.microsoft.com/?tenantid=<tenant id>` ou `https://portal.azure.com/<tenant id>`, ou dans le cas d'un domaine vérifié, `https://myapps.microsoft.com/<verified domain>.onmicrosoft.com`). Liens directs aux applications et ressources fonctionnent également tant qu’ils incluent le contexte client. Les utilisateurs invités ne peuvent actuellement pas se connecter à l’aide de points de terminaison qui n’ont aucun contexte locataire. Par exemple, `https://myapps.microsoft.com` et `https://portal.azure.com` provoquent une erreur.

## <a name="user-experience-for-one-time-passcode-guest-users"></a>Expérience utilisateur pour les utilisateurs invités avec code secret à usage unique

Lorsque la fonctionnalité d’envoi d’un code secret à usage unique par e-mail est activée, les utilisateurs invités [qui remplissent certaines conditions](#when-does-a-guest-user-get-a-one-time-passcode) utilisent l’authentification par code secret à usage unique. Les utilisateurs invités qui ont accepté une invitation avant que cette fonctionnalité ne soit activée continueront de passer par la même méthode d’authentification qu’avant.

Avec l’authentification par code secret à usage unique, l’utilisateur invité peut accepter votre invitation en cliquant sur un lien direct ou à l’aide de l’e-mail d’invitation. Dans les deux cas, un message dans le navigateur indique qu’un code sera envoyé à l’adresse e-mail de l’utilisateur invité. L’utilisateur invité sélectionne **Envoyer le code** :

   ![Capture d’écran montrant le bouton Envoyer le code](media/one-time-passcode/otp-send-code.png)

Un code secret est envoyé à l’adresse e-mail de l’utilisateur. L’utilisateur récupère le code secret à partir de l’e-mail et le saisit dans la fenêtre du navigateur :

   ![Capture d’écran montrant le bouton Entrer le code](media/one-time-passcode/otp-enter-code.png)

L’utilisateur invité est maintenant authentifié, et il peut voir la ressource partagée ou continuer à se connecter.

> [!NOTE]
> Les codes secrets à usage unique sont valides pendant 30 minutes. Après 30 minutes, ce code secret à usage unique spécifique n’est plus valide, et l’utilisateur doit en demander un nouveau. Les sessions utilisateur expirent après 24 heures. Passée cette durée, l’utilisateur invité reçoit un nouveau code secret quand ils accèdent à la ressource. L’expiration de la session garantit plus de sécurité, particulièrement quand un utilisateur invité quitte l’entreprise ou n’a plus besoin d’accéder à la ressource.

## <a name="when-does-a-guest-user-get-a-one-time-passcode"></a>Quand un utilisateur invité obtient-il un code secret à usage unique ?

Quand un utilisateur invité accepte une invitation ou utilise un lien vers une ressource qui a été partagée avec lui, il va recevoir un code secret à usage unique dans les cas suivants :

- Il n’a pas de compte Azure AD
- Il n’a pas de compte Microsoft
- Le locataire qui invite n’a pas configuré de fédération avec des fournisseurs d’identité sociale (comme [Google](google-federation.md)) ou d’autres fournisseurs d’identité.

Au moment de l’invitation, rien n’indique que l’utilisateur que vous invitez devra utiliser l’authentification par code secret à usage unique. Mais lorsque l’utilisateur invité se connecte, l’authentification par code secret à usage unique est la méthode de secours si aucune autre méthode d’authentification ne peut être utilisée.


> [!NOTE]
> Lorsqu’un utilisateur accepte un code secret à usage unique, puis obtient un compte Azure AD, MSA, ou un autre compte fédéré, il continue d’être authentifié à l’aide d’un code secret à usage unique. Si vous souhaitez mettre à jour la méthode d’authentification de l’utilisateur, vous pouvez [réinitialiser son état d’acceptation](reset-redemption-status.md).

### <a name="example"></a>Exemple

L’utilisateur invité teri@gmail.com est invité sur Fabrikam, qui n’a pas de fédération Google configurée. Teri ne possède pas de compte Microsoft. Il va recevoir un code secret à usage unique pour s’authentifier.

## <a name="enable-email-one-time-passcode"></a>Désactiver le code secret à usage unique envoyé par e-mail

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur général Azure AD.

2. Dans le volet de navigation, sélectionnez **Azure Active Directory**.

3. Sélectionnez **Identités externes** > **Tous les fournisseurs d’identité**.

4. Sélectionnez **Code secret à usage unique envoyé par e-mail** pour ouvrir le volet de configuration.

5. Sous **Code secret à usage unique envoyé par e-mail**, sélectionnez l’une des options suivantes :

   - **Activer automatiquement le code secret à usage unique envoyé par e-mail pour les invités à partir d’octobre 2021** si vous ne souhaitez pas activer la fonctionnalité immédiatement et souhaitez attendre la date d’activation automatique en octobre 2021.
   - **Activer dès maintenant le code secret à usage unique envoyé par e-mail pour les invités** pour activer la fonctionnalité maintenant.
   - **Oui** pour activer la fonctionnalité maintenant si vous voyez un bouton bascule Oui/Non (ce bouton apparaît si la fonctionnalité a été précédemment désactivée).

   ![Bouton bascule du code secret à usage unique envoyé par e-mail désactivé](media/one-time-passcode/enable-email-otp-options.png)

5. Sélectionnez **Enregistrer**.

## <a name="disable-email-one-time-passcode"></a>Désactivation de l’envoi d’un code à usage unique par e-mail

À compter d’octobre 2021, la fonctionnalité d’envoi d’un code secret à usage unique par e-mail sera activée pour tous les locataires existants et sera activée par défaut pour les nouveaux locataires. Microsoft ne prendra alors plus en charge l’utilisation d’invitations effectuée en créant des locataires et des comptes Azure AD non gérés (« viraux » ou « juste-à-temps ») pour les scénarios B2B Collaboration. Il s’agit en effet d’une méthode d’authentification de secours transparente pour les utilisateurs invités. Vous aurez toutefois la possibilité de désactiver cette fonctionnalité si vous choisissez de ne pas l’utiliser.

> [!NOTE]
>
> Si la fonctionnalité d’envoi d’un code secret à usage unique par e-mail a été activée dans votre locataire et que vous la désactivez, les utilisateurs invités qui ont demandé un code secret à usage unique ne seront pas en mesure de se connecter. Vous pouvez [réinitialiser leur état d’acceptation](reset-redemption-status.md) pour leur permettre de se reconnecter à l’aide d’une autre méthode d’authentification.

### <a name="to-disable-the-email-one-time-passcode-feature"></a>Procédure de désactivation de la fonctionnalité d’envoi d’un code à usage unique par e-mail

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur général Azure AD.

2. Dans le volet de navigation, sélectionnez **Azure Active Directory**.

3. Sélectionnez **Identités externes** > **Tous les fournisseurs d’identité**.

4. Sélectionnez **Code secret à usage unique envoyé par e-mail**, puis, sous **Code secret à usage unique envoyé par e-mail pour les invités**, sélectionnez **Désactiver le code secret à usage unique envoyé par e-mail pour les invités** (ou **Non** si la fonctionnalité a été précédemment activée, désactivée ou choisie pendant la préversion).

   ![Bouton bascule du code à usage unique par e-mail désactivé](media/one-time-passcode/disable-email-otp-options.png)

   > [!NOTE]
   > Dans le portail Azure, les paramètres de code à usage unique par e-mail ont été déplacés de **Paramètres de collaboration externe** vers **Tous les fournisseurs d’identité**.
   > Si vous voyez un bouton bascule au lieu des options d’envoi d’un code à usage unique par e-mail, cela signifie que vous avez précédemment activé, désactivé ou choisi la préversion de la fonctionnalité. Sélectionnez **Non** pour désactiver la fonctionnalité.

5. Sélectionnez **Enregistrer**.

## <a name="note-for-public-preview-customers"></a>Remarque pour les clients de la préversion publique

Si vous avez déjà choisi la préversion publique de la fonctionnalité d’envoi d’un code secret à usage unique par e-mail, la date d’activation automatique de la fonctionnalité (octobre 2021) ne s’applique pas à vous. De ce fait, vos processus d’entreprise associés ne seront pas affectés. Vous ne verrez pas non plus l’option **Activer automatiquement l’envoi d’un code à usage unique par e-mail pour les invités à partir d’octobre 2021** sous les propriétés **Code à usage unique par e-mail pour les invités** sur le portail Azure. Seul le bouton bascule **Oui** ou **Non** apparaîtra :

![Activation de la fonctionnalité d’envoi d’un code à usage unique par e-mail](media/one-time-passcode/enable-email-otp-opted-in.png)

Toutefois, si vous préférez désactiver la fonctionnalité et autoriser son activation automatique en octobre 2021, vous pouvez revenir aux paramètres par défaut à l’aide du [type de ressource de configuration de la méthode d’authentification par e-mail](/graph/api/resources/emailauthenticationmethodconfiguration) de l’API Microsoft Graph. Les options suivantes seront alors disponibles sous **Envoi d’un code secret à usage unique par e-mail pour les invités** :

![Choix de la fonctionnalité Envoi d’un code à usage unique par e-mail](media/one-time-passcode/email-otp-options.png)

- **Activer automatiquement le code à usage unique par e-mail pour les invités à partir d’octobre 2021**. (Par défaut) Si la fonctionnalité d’envoi d’un code à usage unique par e-mail n’est pas encore activée pour votre locataire, elle le sera automatiquement à partir d’octobre 2021. Aucune action supplémentaire n’est nécessaire si vous souhaitez que la fonctionnalité soit activée à ce moment-là. Si vous avez déjà activé ou désactivé la fonctionnalité, cette option n’est pas disponible.

- **Activer dès maintenant l’envoi d’un code à usage unique par e-mail pour les invités** : cette option permet d’activer la fonctionnalité d’envoi d’un code secret à usage unique par e-mail pour votre locataire.

- **Désactiver l’envoi d’un code à usage unique par e-mail pour les invités** : cette option permet de désactiver la fonctionnalité d’envoi d’un code secret à usage unique par e-mail pour votre locataire et d’empêcher qu’elle ne s’active en octobre 2021.

## <a name="note-for-azure-us-government-customers"></a>Remarque pour les clients Azure US Government

La fonctionnalité de code à usage unique par e-mail est désactivée par défaut dans le cloud Azure US Government. Vos partenaires ne pourront pas se connecter si cette fonctionnalité n’est pas activée. Contrairement au cloud public Azure, le cloud US Government Azure ne prend pas en charge l’acceptation d'invitation avec les comptes Azure Active Directory en libre-service.

 ![Code à usage unique par e-mail désactivé](media/one-time-passcode/enable-email-otp-disabled.png)

Pour activer la fonctionnalité de code à usage unique par e-mail dans le cloud Azure US Government :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général Azure AD.
2. Dans le volet de navigation, sélectionnez **Azure Active Directory**.
3. Sélectionnez **Relations organisationnelles** > **Tous les fournisseurs d’identité**.

   > [!NOTE]
   > - Si vous ne voyez pas l’option **Relations organisationnelles**, recherchez « identités externes » dans la barre de recherche en haut.

4. Sélectionnez **Code à usage unique par e-mail**, puis cliquez sur **Oui**.
5. Sélectionnez **Enregistrer**.

Pour plus d’informations sur les limitations actuelles, consultez la page sur les [clouds Azure US Government](current-limitations.md#azure-us-government-clouds).
