---
title: Versions des mises en page
titleSuffix: Azure AD B2C
description: Historique des versions de mise en page pour la personnalisation de l’interface utilisateur dans les stratégies personnalisées.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 08/25/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: aa60cf86a8bc59b9eed2adc8ac0ba2cfb89be584
ms.sourcegitcommit: d858083348844b7cf854b1a0f01e3a2583809649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2021
ms.locfileid: "122835609"
---
# <a name="page-layout-versions"></a>Versions des mises en page

Les packages de mise en page sont régulièrement mis à jour afin d’ajouter des correctifs et des améliorations à leurs éléments de page. Le journal des modifications suivant indique les modifications introduites dans chaque version.

> [!IMPORTANT]
> Azure Active Directory B2C publie des améliorations et des correctifs avec chaque nouvelle version de mise en page. Nous vous recommandons vivement de maintenir vos versions de mise en page à jour afin que tous les éléments de la page reflètent les dernières améliorations en matière de sécurité et de normes d’accessibilité ainsi que vos commentaires.
>

## <a name="jquery-and-handlebars-versions"></a>Versions de jQuery et de Handlebars

La mise en page d’Azure AD B2C utilise la version suivante de la [bibliothèque jQuery](https://jquery.com/) et les [modèles Handlebars](https://handlebarsjs.com/) :

|Élément |Plage de versions de mise en page |Version jQuery  |Version de l’exécution Handlebars |Version du compilateur Handlebars |
|---------|---------|------|--------|----------|
|multifacteur |>= 1.2.4 | 3.5.1 | 4.7.6 |4.7.7 |
|            |< 1.2.4 | 3.4.1 |4.0.12 |2.0.1 |
|            |< 1.2.0 | 1.12.4 |
|selfasserted |>= 2.1.4 | 3.5.1 |4.7.6 |4.7.7 |
|            |< 2.1.4 | 3.4.1 |4.0.12 |2.0.1 |
|            |< 1.2.0 | 1.12.4 |
|unifiedssp |>= 2.1.4 | 3.5.1 |4.7.6 |4.7.7 |
|            |< 2.1.4 | 3.4.1 |4.0.12 |2.0.1 |
|            |< 1.2.0 | 1.12.4 |
|globalexception |>= 1.2.1 | 3.5.1 |4.7.6 |4.7.7 |
|            |< 1.2.1 | 3.4.1 |4.0.12 |2.0.1 |
|            |< 1.2.0 | 1.12.4 |
|providerselection |>= 1.2.1 | 3.5.1 |4.7.6 |4.7.7 |
|            |< 1.2.1 | 3.4.1 |4.0.12 |2.0.1 |
|            |< 1.2.0 | 1.12.4 |
|claimsconsent |>= 1.2.1 | 3.5.1 |4.7.6 |4.7.7 |
|            |< 1.2.1 | 3.4.1 |4.0.12 |2.0.1 |
|            |< 1.2.0 | 1.12.4 |
|unifiedssd |>= 1.2.1 | 3.5.1 |4.7.6 |4.7.7 |
|            |< 1.2.1 | 3.4.1 |4.0.12 |2.0.1 |
|            |< 1.2.0 | 1.12.4 |

## <a name="self-asserted-page-selfasserted"></a>Page autodéclarée (selfasserted)

**2.1.7**
- Correction d’un problème d’encodage linguistique qui provoque l’échec de la requête.
- Correction d’un bogue d’accessibilité pour afficher des messages d’erreur inclus uniquement lors de l’envoi d’un formulaire.

**2.1.6**
- Correction d’une erreur de mot de passe effacé lorsque vous tapez trop rapidement dans un autre champ.

**2.1.5**
- Correction du problème de saut de curseur sur iOS lors de la modification au milieu du texte.

**2.1.4**
- JQuery mis à jour vers la version 3.5.1.
- HandlebarJS mis à jour vers la version 4.7.6.

**2.1.3**
- Correctifs de sécurité.

**2.1.2**
- Correction du problème d’encodage de la localisation pour des langues telles que l’espagnol et le français.

**2.1.1**

- Ajout d’un élément `heading` UXString en plus de `intro` à afficher sur la page en tant que titre. Cet élément est masqué par défaut.
- Ajout de la prise en charge de l’enregistrement des mots de passe dans le trousseau iCloud.
- Ajout de la prise en charge de l’utilisation d’une stratégie ou du paramètre QueryString `pageFlavor` pour sélectionner la disposition (classique, bleu océan ou ardoise).
- Ajout d’exclusions de responsabilité sur la page autodéclarée.
- Le focus est maintenant placé sur le premier champ modifiable lors du chargement de la page.
- Le focus est maintenant placé sur le premier champ d’erreur lorsque plusieurs champs comportent des erreurs.
- Le focus est maintenant placé sur le bouton 'change' (modifier) après vérification du code de vérification de l’e-mail.

**2.1.0**

- Correctifs de localisation et d’accessibilité.

**2.0.0**

- Ajout de la prise en charge des [contrôles d’affichage](display-controls.md) dans les stratégies personnalisées.

**1.2.0**

- Les champs de nom d’utilisateur/adresse e-mail et de mot de passe utilisent désormais l’élément HTML `form` pour permettre à Edge et Internet Explorer (IE) d’enregistrer correctement ces informations.
- Ajout d’un délai de validation d’entrée utilisateur configurable pour une expérience utilisateur améliorée.
- Correctifs de l’accessibilité
- Correction d’un problème d’accessibilité afin que les messages d’erreur soient désormais lus par le narrateur. 
- Le focus est désormais placé dans le champ du mot de passe une fois l’e-mail vérifié.
- Suppression de `autofocus` du contrôle de case à cocher. 
- Ajout de la prise en charge d’un contrôle d’affichage pour la vérification du numéro de téléphone.
- Vous pouvez maintenant ajouter l’attribut `data-preload="true"` [dans vos balises HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content).
  - Chargez les fichiers CSS liés en même temps que votre modèle HTML, de sorte qu’il n’y ait pas de « scintillement » entre le chargement des fichiers.
  - Contrôlez l’ordre dans lequel vos balises `script` sont récupérées et exécutées avant le chargement de la page.
- Le champ d’e-mail est désormais `type=email`, et les claviers mobiles fournissent les suggestions appropriées.
- Prise en charge de Chrome Translate.
- Ajout de la prise en charge de la marque de société dans les pages de flux d’utilisateurs.

**1.1.0**

- Suppression de l’alerte d’annulation
- Classe CSS pour les éléments d’erreur
- Amélioration de la fonction Afficher/masquer la logique d’erreur
- Suppression du CSS par défaut

**1.0.0**

- Version initiale

## <a name="unified-sign-in-sign-up-page-with-password-reset-link-unifiedssp"></a>Page d’inscription à la connexion unifiée avec lien de réinitialisation de mot de passe (unifiedssp)

> [!TIP]
> Si vous localisez votre page pour prendre en charge plusieurs paramètres régionaux ou langues dans un flux utilisateur. L’article [ID de localisation](localization-string-ids.md) fournit la liste des ID de localisation que vous pouvez utiliser pour la version de page que vous sélectionnez.

**2.1.5**
- Correction d’un problème dans l’ordre de tabulation lorsque le modèle de sélecteur IDP est utilisé sur la page de connexion.
- Correction d’un problème d’encodage sur le texte du lien de connexion.

**2.1.4**
- JQuery mis à jour vers la version 3.5.1.
- HandlebarJS mis à jour vers la version 4.7.6.

**2.1.3**
- Correctifs de sécurité.
- Correctifs de bogues mineurs.

**2.1.2**
- Correction du problème d’encodage de la localisation pour des langues telles que l’espagnol et le français.
- Possibilité d’utiliser le lien « mot de passe oublié » comme échange de revendications. Pour plus d’informations, consultez [Réinitialisation de mot de passe en libre service](add-password-reset-policy.md#self-service-password-reset-recommended).

**2.1.1**
- Ajout d’un élément `heading` UXString en plus de `intro` à afficher sur la page en tant que titre. Cet élément est masqué par défaut.
- Ajout de la prise en charge de l’utilisation d’une stratégie ou du paramètre QueryString `pageFlavor` pour sélectionner la disposition (classique, bleu océan ou ardoise).
- Ajout de la prise en charge de l’enregistrement des mots de passe dans le trousseau iCloud.
- Le focus est maintenant placé sur le premier champ d’erreur lorsque plusieurs champs comportent des erreurs.
- Le focus est maintenant placé sur le premier champ modifiable lors du chargement de la page.
- Ajout d’un nouvel emplacement pour le lien de sélection du fournisseur de revendications `bottomUnderFormClaimsProviderSelections`.
- Suppression des éléments UXString qui ne sont plus utilisés.

**2.1.0**

- Ajout de la prise en charge de plusieurs liens d’inscription.
- Ajout de la prise en charge de la validation des entrées utilisateur conformément aux règles de prédicat définies dans la stratégie.
- Lorsque l'[option de connexion](sign-in-options.md) est définie sur E-mail, l’en-tête de connexion affiche « Connectez-vous avec votre nom de connexion ». Le champ de nom d’utilisateur affiche « nom de la connexion ». Pour plus d'informations, consultez [localisation](localization-string-ids.md#sign-up-or-sign-in-page-elements).

**1.2.0**

- Les champs de nom d’utilisateur/adresse e-mail et de mot de passe utilisent désormais l’élément HTML `form` pour permettre à Edge et Internet Explorer (IE) d’enregistrer correctement ces informations.
- Correctifs de l’accessibilité
- Vous pouvez maintenant ajouter l’attribut `data-preload="true"` [dans vos balises HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) pour contrôler l’ordre de chargement des fichiers CSS et JavaScript.
  - Chargez les fichiers CSS liés en même temps que votre modèle HTML, de sorte qu’il n’y ait pas de « scintillement » entre le chargement des fichiers.
  - Contrôlez l’ordre dans lequel vos balises `script` sont récupérées et exécutées avant le chargement de la page.
- Le champ d’e-mail est désormais `type=email`, et les claviers mobiles fournissent les suggestions appropriées.
- Prise en charge de Chrome Translate.
- Ajout de la prise en charge de la marque de client dans les pages de flux d’utilisateurs.

**1.1.0**

- Ajout de l’option de contrôle Maintenir la connexion (KMSI)

**1.0.0**

- Version initiale

## <a name="mfa-page-multifactor"></a>Page MFA (multifactor)

**1.2.5**
- Correction d’un problème d’encodage linguistique qui provoque l’échec de la requête.

**1.2.4**
- JQuery mis à jour vers la version 3.5.1.
- HandlebarJS mis à jour vers la version 4.7.6.

**1.2.3**
- Autorisation du remplacement de la chaîne ToolTip via la localisation de la langue.
- Correctifs de sécurité.
- Correctifs de bogues mineurs.

**1.2.2**
- Correction d’un problème de remplissage automatique du code de vérification lors de l’utilisation d’iOS.
- Correction d’un problème de redirection d’un jeton vers la partie de confiance à partir d’Android WebView. 
- Ajout d’un élément `heading` UXString en plus de `intro` à afficher sur la page en tant que titre. Cet élément est masqué par défaut.  
- Ajout de la prise en charge de l’utilisation d’une stratégie ou du paramètre QueryString `pageFlavor` pour sélectionner la disposition (classique, bleu océan ou ardoise).

**1.2.1**

- Correctifs d’accessibilité sur les modèles par défaut

**1.2.0**

- Correctifs de l’accessibilité
- Vous pouvez maintenant ajouter l’attribut `data-preload="true"` [dans vos balises HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) pour contrôler l’ordre de chargement des fichiers CSS et JavaScript.
  - Chargez les fichiers CSS liés en même temps que votre modèle HTML, de sorte qu’il n’y ait pas de « scintillement » entre le chargement des fichiers.
  - Contrôlez l’ordre dans lequel vos balises `script` sont récupérées et exécutées avant le chargement de la page.
- Le champ d’e-mail est désormais `type=email`, et les claviers mobiles fournissent les suggestions appropriées
- Prise en charge de Chrome Translate.
- Ajout de la prise en charge de la marque de client dans les pages de flux d’utilisateurs.

**1.1.0**

- Suppression du bouton « Confirmer le code »
- Il n’est maintenant possible d’entrer que six (6) caractères dans le champ d’entrée du code
- La page essaie automatiquement de vérifier le code entré lorsqu’un code à six chiffres est saisi, sans cliquer sur le moindre bouton
- Si le code est incorrect, le contenu du champ d’entrée est automatiquement effacé
- Après trois (3) tentatives avec un code incorrect, B2C renvoie une erreur à la partie de confiance
- Correctifs de l’accessibilité
- Suppression du CSS par défaut

**1.0.0**

- Version initiale

## <a name="exception-page-globalexception"></a>Page Exception (globalexception)

**1.2.1**
- JQuery mis à jour vers la version 3.5.1.
- HandlebarJS mis à jour vers la version 4.7.6.

**1.2.0**

- Correctifs de l’accessibilité
- Vous pouvez maintenant ajouter l’attribut `data-preload="true"` [dans vos balises HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) pour contrôler l’ordre de chargement des fichiers CSS et JavaScript.
  - Chargez les fichiers CSS liés en même temps que votre modèle HTML, de sorte qu’il n’y ait pas de « scintillement » entre le chargement des fichiers.
  - Contrôlez l’ordre dans lequel vos balises `script` sont récupérées et exécutées avant le chargement de la page.
- Le champ d’e-mail est désormais `type=email`, et les claviers mobiles fournissent les suggestions appropriées
- Prise en charge de Chrome Translate

**1.1.0**

- Correctif de l’accessibilité
- Suppression du message par défaut s’affichant lorsqu’il n’y a aucun contact de la stratégie
- Suppression du CSS par défaut

**1.0.0**

- Version initiale

## <a name="other-pages-providerselection-claimsconsent-unifiedssd"></a>Autres pages (ProviderSelection, ClaimsConsent, UnifiedSSD)

**1.2.1**
- JQuery mis à jour vers la version 3.5.1.
- HandlebarJS mis à jour vers la version 4.7.6.

**1.2.0**

- Correctifs de l’accessibilité
- Vous pouvez maintenant ajouter l’attribut `data-preload="true"` [dans vos balises HTML](customize-ui-with-html.md#guidelines-for-using-custom-page-content) pour contrôler l’ordre de chargement des fichiers CSS et JavaScript.
  - Chargez les fichiers CSS liés en même temps que votre modèle HTML, de sorte qu’il n’y ait pas de « scintillement » entre le chargement des fichiers.
  - Contrôlez l’ordre dans lequel vos balises `script` sont récupérées et exécutées avant le chargement de la page.
- Le champ d’e-mail est désormais `type=email`, et les claviers mobiles fournissent les suggestions appropriées
- Prise en charge de Chrome Translate

**1.0.0**

- Version initiale

## <a name="next-steps"></a>Étapes suivantes

Pour plus de détails sur la personnalisation de l’interface utilisateur de vos applications, voir [Personnaliser l’interface utilisateur de votre application à l’aide d’une stratégie personnalisée](customize-ui-with-html.md).
