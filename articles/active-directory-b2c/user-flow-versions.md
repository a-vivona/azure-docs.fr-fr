---
title: Versions de flux utilisateur dans Azure Active Directory B2C | Microsoft Docs
description: Découvrez-en plus sur les différentes versions de flux utilisateur disponibles dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 08/17/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: ff22ed8c1ef96d1839767ac56a7215737ae18c14
ms.sourcegitcommit: 5f659d2a9abb92f178103146b38257c864bc8c31
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122527713"
---
# <a name="user-flow-versions-in-azure-active-directory-b2c"></a>Versions de flux utilisateur dans Azure Active Directory B2C

Les flux utilisateur dans Azure Active Directory B2C (Azure AD B2C) vous aident à configurer des [stratégies](user-flow-overview.md) courantes qui décrivent entièrement les expériences liées à l’identité des clients. Ces expériences incluent l’inscription, la connexion, la réinitialisation du mot de passe et la modification du profil. Les tableaux ci-dessous décrivent les flux d’utilisateurs disponibles dans Azure AD B2C.

> [!IMPORTANT]
> Nous avons modifié la manière dont nous référençons les versions de flux utilisateur. Auparavant, nous proposions des versions V1 (prêtes pour la production) et des versions V1.1 et V2 (préversions). Maintenant, nous avons regroupé les flux d’utilisateur en deux versions :
>
>- Les flux d’utilisateurs **recommandés** sont les flux d’utilisateurs de nouvelle génération mis à la disposition générale et dotés des dernières fonctionnalités. Ils combinent toutes les fonctionnalités des versions **v1**, **v1.1** et **v2** héritées. À l’avenir, les flux d’utilisateurs **recommandés** seront maintenus et mis à jour. Quand vous passez à ces nouveaux flux d’utilisateur recommandés, vous avez accès aux nouvelles fonctionnalités à mesure qu’elles sont publiées.
>- Les flux d’utilisateurs **standard (hérités)** , précédemment appelés **v1**, sont des flux d’utilisateurs hérités. À moins que vous n’ayez un besoin professionnel spécifique, nous vous déconseillons d’utiliser ces versions des flux d’utilisateurs, car elles ne seront pas maintenues ou mises à jour.
>
>Dans le cloud public, tous les flux d’utilisateurs en préversion hérités (v1.1 et v2) sont déconseillés. *Ces modifications s’appliquent uniquement au cloud public Azure. Les autres environnements continueront à utiliser le [contrôle de version de flux utilisateur hérité](user-flow-versions-legacy.md).*

## <a name="recommended-user-flows"></a>Flux utilisateur recommandés

Les flux d’utilisateurs recommandés sont les flux d’utilisateurs de nouvelle génération mis à la disposition générale et dotés des dernières fonctionnalités. À l’avenir, les flux utilisateur recommandés seront maintenus et mis à jour.

| Flux utilisateur | Description |
| --------- | ----------- |
| Réinitialisation du mot de passe | Permet à un utilisateur de choisir un nouveau mot de passe après la vérification de son adresse e-mail. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>Paramètres de conformité de jeton</li><li>[Vérification de l’âge](age-gating.md)</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Modification de profil | Permet à un utilisateur de configurer ses attributs utilisateur. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li></ul> |
| Se connecter | Permet à un utilisateur de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Vérification de l’âge](age-gating.md)</li><li>Personnalisation de la page de connexion</li></ul> |
| Inscription | Permet à un utilisateur de créer un compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Vérification de l’âge](age-gating.md)</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Inscription et connexion | Permet à un utilisateur de créer un compte ou de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Vérification de l’âge](age-gating.md)</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |

## <a name="standard-user-flows"></a>Flux d’utilisateur standard

Les flux d’utilisateurs standard, précédemment appelés **v1**, sont des flux d’utilisateurs hérités. À moins que vous n’ayez un besoin professionnel spécifique, nous vous déconseillons d’utiliser ces versions des flux d’utilisateurs, car elles ne seront pas mises à jour à l’avenir.

| Flux utilisateur | Description |
| --------- | ----------- |
| Réinitialisation du mot de passe | Permet à un utilisateur de choisir un nouveau mot de passe après la vérification de son adresse e-mail. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>Paramètres de conformité de jeton</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Modification de profil | Permet à un utilisateur de configurer ses attributs utilisateur. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li></ul> |
| Se connecter | Permet à un utilisateur de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>Bloquer la connexion</li><li>Forcer la réinitialisation du mot de passe</li><li>Maintenir la connexion</ul><br>Vous ne pouvez pas personnaliser l’interface utilisateur avec ce flux utilisateur. |
| Inscription | Permet à un utilisateur de créer un compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul> |
| Inscription et connexion | Permet à un utilisateur de créer un compte ou de se connecter à son compte. À l’aide de ce flux utilisateur, vous pouvez configurer les éléments suivants : <ul><li>[Authentification multifacteur](multi-factor-authentication.md)</li><li>[Durée de vie du jeton](tokens-overview.md)</li><li>Paramètres de conformité de jeton</li><li>Comportement de la session</li><li>[Exigences en termes de « complexité » du mot de passe](password-complexity.md)</li></ul>|


## <a name="how-to-switch-to-a-recommended-user-flow"></a>Comment basculer vers un flux d’utilisateur recommandé

Pour passer d’une version héritée d’un flux d’utilisateur à la version **recommandée**, procédez comme suit :

1. Créez une nouvelle stratégie de flux utilisateur en suivant les étapes décrites dans [Tutoriel : Créer des flux utilisateur dans Azure Active Directory](tutorial-create-user-flows.md). Lors de la création du flux utilisateur, sélectionnez la version **recommandée**.

3. Configurez votre nouveau flux utilisateur avec les mêmes paramètres que ceux configurés dans la stratégie héritée.

4. Mettez à jour l’URL de connexion de votre application vers la stratégie nouvellement créée.

5. Une fois que vous avez testé le flux utilisateur et confirmé qu’il fonctionne, supprimez le flux utilisateur hérité en procédant comme suit :
   1. Dans le menu de la page de vue d’ensemble du locataire Azure AD B2C, sélectionnez **Flux utilisateur**.
   2. Recherchez le flux utilisateur que vous souhaitez supprimer.
   3. Dans la dernière colonne, sélectionnez le menu contextuel ( **...** ), puis choisissez **Supprimer**.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="can-i-still-create-legacy-v2-and-v11-user-flows"></a>Puis-je toujours créer des flux utilisateur V2 et V1.1 hérités ?

Vous ne pouvez pas créer de flux utilisateur basés sur les versions V2 et V1.1 héritées, mais vous pouvez continuer à lire, mettre à jour et supprimer tout flux utilisateur V2 et V1.1 hérité que vous utilisez actuellement.

### <a name="is-there-any-reason-to-continue-using-legacy-v2-and-v11-user-flows"></a>Y a-t-il une raison de continuer à utiliser les flux d’utilisateur V2 et V1.1 hérités ?

Pas vraiment. Les versions **recommandées** contiennent les mêmes fonctionnalités que les versions v2 et v1.1 héritées. Rien n’a été supprimé et, en fait, elles incluent maintenant des fonctionnalités supplémentaires.

### <a name="if-i-dont-switch-from-legacy-v2-and-v11-policies-how-will-it-impact-my-application"></a>Si je ne passe pas des stratégies V2 et V1.1 héritées aux nouvelles, comment cela affectera-t-il mon application ?

Si vous utilisez un flux utilisateur V2 ou V1.1 hérité, votre application n’est pas affectée par ce changement de version. Mais pour accéder aux nouvelles fonctionnalités ou modifications de stratégie à l’avenir, vous devrez basculer vers les versions **recommandées**.

### <a name="will-microsoft-still-support-my-legacy-v2-or-v11-user-flow-policy"></a>Microsoft continuera-t-il de prendre en charge ma stratégie de flux utilisateur V2 ou V1.1 héritée ?

Dans le cloud public, tous les flux d’utilisateurs en préversion hérités (v1.1 et v2) sont déconseillés. *Ces modifications s’appliquent uniquement au cloud public Azure. Les autres environnements continueront à utiliser le [contrôle de version de flux utilisateur hérité](user-flow-versions-legacy.md).*