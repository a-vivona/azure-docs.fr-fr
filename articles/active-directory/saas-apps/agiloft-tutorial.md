---
title: 'Didacticiel : Intégration d’Azure Active Directory à Agiloft Contract Management Suite | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Agiloft Contract Management Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/03/2021
ms.author: jeedes
ms.openlocfilehash: 25bea4a80a33d520683ec56ee4b40dd54b9652c9
ms.sourcegitcommit: 70ce9237435df04b03dd0f739f23d34930059fef
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/05/2021
ms.locfileid: "111527007"
---
# <a name="tutorial-azure-active-directory-integration-with-agiloft-contract-management-suite"></a>Didacticiel : Intégration d’Azure Active Directory à Agiloft Contract Management Suite

Dans ce tutoriel, vous allez apprendre à intégrer Agiloft Contract Management Suite dans Azure Active Directory (Azure AD). Quand vous intégrez Agiloft Contract Management Suite à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Agiloft Contract Management Suite.
* Permettre à vos utilisateurs de se connecter automatiquement à Agiloft Contract Management Suite avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Agiloft Contract Management Suite pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Agiloft Contract Management Suite prend en charge l’authentification unique initiée par le **fournisseur de services et le fournisseur d’identité**
* Agiloft Contract Management Suite prend en charge le provisionnement d’utilisateurs **juste-à-temps**.

## <a name="add-agiloft-contract-management-suite-from-the-gallery"></a>Ajouter Agiloft Contract Management Suite à partir de la galerie

Pour configurer l’intégration d’Agiloft Contract Management Suite à Azure AD, vous devez ajouter Agiloft Contract Management Suite, à partir de la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Agiloft Contract Management Suite** dans la zone de recherche.
1. Sélectionnez **Agiloft Contract Management Suite** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-agiloft-contract-management-suite"></a>Configurer et tester l’authentification unique Azure AD pour Agiloft Contract Management Suite

Configurez et testez l’authentification unique Azure AD avec Agiloft Contract Management Suite pour un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Agiloft Contract Management Suite associé.

Pour configurer et tester l’authentification unique Azure AD avec Agiloft Contract Management Suite, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Agiloft Contract Management Suite](#configure-agiloft-contract-management-suite-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer l’utilisateur de test Agiloft Contract Management Suite](#create-agiloft-contract-management-suite-test-user)** pour avoir dans Agiloft Contract Management Suite un équivalent de B.Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le Portail Azure, dans la page d’intégration de l’application **Agiloft Contract Management Suite**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de crayon de **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode initié par **IDP**, suivez les étapes ci-dessous :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<SUBDOMAIN>.agiloft.com/<KB_NAME>`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<SUBDOMAIN>.agiloft.com:443/gui2/spsamlsso?project=<KB_NAME>`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<SUBDOMAIN>.agiloft.com/gui2/samlssologin.jsp?project=<KB_NAME>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique d’Agiloft Contract Management Suite](https://www.agiloft.com/support-login.htm). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer Agiloft Contract Management Suite**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Agiloft Contract Management Suite.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Agiloft Contract Management Suite**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-agiloft-contract-management-suite-sso"></a>Configurer l’authentification unique Agiloft Contract Management Suite

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Agiloft Contract Management Suite en tant qu’administrateur.

2. Cliquez sur **Setup** (Configurer) dans le volet gauche, puis sur **Access** (Accès).

    ![Capture d’écran mettant en évidence la section Access.](./media/agiloft-tutorial/access.png)

3. Cliquez sur le bouton permettant de **configurer l’authentification unique SAML 2.0**.

    ![Capture d’écran mettant en évidence le bouton Configure SAML 2.0 Single Sign-On.](./media/agiloft-tutorial/setup.png)

4. Une boîte de dialogue d’assistant s’affiche. Dans la boîte de dialogue, cliquez sur **Identity Provider Details** (Détails du fournisseur d’identité) et renseignez les champs suivants :  

    ![Configuration d’Agiloft Contract Management Suite](./media/agiloft-tutorial/details.png)

    a. Dans la zone de texte **ID d’entité du fournisseur d’identité/Émetteur**, collez l’**identificateur Azure AD** que vous avez copié depuis le portail Azure.

    b. Dans la zone de texte **URL de connexion du fournisseur d’identité**, collez la valeur **URL de connexion** que vous avez copiée depuis le portail Azure.

    c. Dans la zone de texte **URL de déconnexion du fournisseur d’identité**, collez la valeur de l’**URL de déconnexion** que vous avez copiée à partir du portail Azure.

    d. Ouvrez dans le Bloc-notes votre **certificat codé en base 64** téléchargé à partir du portail Azure, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte du **contenu du certificat X.509 proposé par le fournisseur d’identité**.

    e. Cliquez sur **Terminer**.

### <a name="create-agiloft-contract-management-suite-test-user"></a>Créer un utilisateur de test Agiloft Contract Management Suite

Dans cette section, un utilisateur appelé Britta Simon est créé dans Agiloft Contract Management Suite. Agiloft Contract Management Suite prend en charge le provisionnement d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Agiloft Contract Management Suite, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion à Agiloft Contract Management Suite où vous pouvez lancer le flux de connexion.  

* Accédez directement à l’URL de connexion à Agiloft Contract Management Suite pour lancer le flux de connexion.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Cliquez sur **Tester cette application** dans le Portail Azure : vous devez être connecté automatiquement à l’instance d’Agiloft Contract Management Suite pour laquelle vous avez configuré l’authentification unique. 

Vous pouvez aussi utiliser Mes applications de Microsoft pour tester l’application dans n’importe quel mode. Si, quand vous cliquez sur la vignette Agiloft Contract Management Suite dans Mes applications, le mode Fournisseur de services est configuré, vous êtes redirigé vers la page de connexion de l’application pour lancer le flux de connexion ; s’il s’agit du mode Fournisseur d’identité, vous êtes automatiquement connecté à l’instance d’Agiloft Contract Management Suite pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Agiloft Contract Management Suite, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).