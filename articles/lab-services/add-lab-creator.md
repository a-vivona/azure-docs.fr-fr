---
title: Ajouter un utilisateur en tant que créateur de laboratoire dans Azure Lab Services
description: Cet article explique comment ajouter un utilisateur au rôle Créateur de laboratoire pour un compte lab dans Azure Lab Services. Les créateurs de laboratoire peuvent créer des labos au sein de ce compte lab.
ms.topic: article
ms.date: 07/26/2021
ms.custom: subject-rbac-steps
ms.openlocfilehash: a2135ab6580d39d6c63f7e948a29f0f0ede4efd6
ms.sourcegitcommit: 9f1a35d4b90d159235015200607917913afe2d1b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/21/2021
ms.locfileid: "122634192"
---
# <a name="add-lab-creators-to-a-lab-account-in-azure-lab-services"></a>Ajouter des créateurs de laboratoire à un compte lab dans Azure Lab Services
Cet article explique comment ajouter des utilisateurs en tant que créateurs de laboratoire à un compte lab dans Azure Lab Services. Ces utilisateurs peuvent alors créer des labos dans le compte de labo. 

## <a name="add-microsoft-user-account-to-lab-creator-role"></a>Ajouter un compte d’utilisateur Microsoft au rôle Créateur de laboratoire
Pour configurer un laboratoire de classe dans un compte de laboratoire, l’utilisateur doit être membre du rôle **Créateur de laboratoire** dans le compte de laboratoire. Le compte utilisé pour créer le compte de laboratoire est automatiquement ajouté à ce rôle. Si vous envisagez d’utiliser le même compte d’utilisateur pour créer un laboratoire de classe, vous pouvez ignorer cette étape. Pour utiliser un autre compte d’utilisateur pour créer un laboratoire de classe, procédez comme suit : 

Pour donner aux formateurs l’autorisation de créer des laboratoires pour leurs classes, ajoutez-les au rôle **Créateur de laboratoire**. Pour en savoir plus sur cette opération, consultez la page [Attribuer des rôles Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md).


1. Sur la page **Compte Lab**, sélectionnez **Contrôle d’accès (IAM)**

1. Sélectionnez **Ajouter** > **Ajouter une attribution de rôle (préversion)** .

    ![Page Contrôle d’accès (IAM) avec le menu Ajouter une attribution de rôle ouvert.](../../includes/role-based-access-control/media/add-role-assignment-menu-generic.png)

1. Dans l’onglet **Rôle**, sélectionnez le rôle **Créateur de laboratoire**.

    ![Page Ajouter une attribution de rôle avec l’onglet Rôle sélectionné.](../../includes/role-based-access-control/media/add-role-assignment-role-generic.png)

1. Sous l’onglet **Membres**, sélectionnez l’utilisateur auquel vous souhaitez ajouter le rôle Créateur de laboratoire.

1. Dans l’onglet **Passer en revue + affecter**, sélectionnez **Passer en revue + affecter** pour affecter le rôle.




    > [!NOTE]
    > Si vous ajoutez un utilisateur de compte non Microsoft en tant que créateur de laboratoire, consultez la section [Ajouter un utilisateur de compte non Microsoft en tant que créateur de laboratoire](#add-a-non-microsoft-account-user-as-a-lab-creator). 

## <a name="add-a-non-microsoft-account-user-as-a-lab-creator"></a>Ajouter un utilisateur de compte non Microsoft en tant que créateur de laboratoire
Pour ajouter un utilisateur en tant que créateur de laboratoire, utilisez ses comptes de messagerie. Les types de comptes de messagerie suivants peuvent être utilisés :

- Un compte de courrier fourni par l'instance Azure Active Directory (AAD) de votre université.
- Un compte de messagerie Microsoft, tel que `@outlook.com`, `@hotmail.com`, `@msn.com` ou `@live.com`.
- Un compte de messagerie non Microsoft, tel qu’un compte fourni par Yahoo ou Google. Cependant, ces types de comptes doivent être liés à un compte Microsoft.
- Un compte GitHub. Ce compte doit être lié à un compte Microsoft.

### <a name="using-a-non-microsoft-email-account"></a>Utiliser un compte de messagerie non Microsoft
Les créateurs de laboratoire et formateurs peuvent utiliser des comptes de messagerie non Microsoft pour s’inscrire et se connecter à un labo de classe.  Toutefois, la connexion au portail Lab Services nécessite que les formateurs créent d’abord un compte Microsoft lié à leur adresse e-mail non Microsoft.

De nombreux formateurs ont peut-être déjà un compte Microsoft lié à leurs adresses e-mail non Microsoft. Par exemple, les formateurs ont déjà un compte Microsoft s’ils ont utilisé leur adresse e-mail avec d’autres produits ou services Microsoft, tels que Office, Skype, OneDrive ou Windows.  

Lorsque les formateurs se connectent au portail Lab Services, ils sont invités à entrer leur adresse e-mail et leur mot de passe. Si le formateur tente de se connecter avec un compte non Microsoft qui n’a pas de compte Microsoft lié, le message d’erreur suivant s’affiche : 

![Message d’erreur](./media/how-to-configure-student-usage/cant-find-account.png)

Pour s’inscrire à un compte Microsoft, les formateurs doivent accéder à la page [http://signup.live.com](http://signup.live.com).  


### <a name="using-a-github-account"></a>Utiliser un compte GitHub
Les formateurs peuvent également utiliser un compte GitHub existant pour s’inscrire et se connecter à un labo de classe. Si le formateur a déjà un compte Microsoft lié à son compte GitHub, il peut se connecter et fournir son mot de passe, comme indiqué dans la section précédente. S’il n’a pas encore lié son compte GitHub à un compte Microsoft, il doit sélectionner **Options de connexion** :

![Lien Options de connexion](./media/how-to-configure-student-usage/signin-options.png)

Dans la page **Options de connexion**, sélectionnez **Se connecter avec GitHub**.

![Lien Se connecter avec GitHub](./media/how-to-configure-student-usage/signin-github.png)

Enfin, il est invité à créer un compte Microsoft qui est lié à son compte GitHub. Cela se produit automatiquement lorsque le formateur sélectionne **Suivant**.  Le formateur est alors immédiatement inscrit et connecté au labo de classe.


## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [En tant que propriétaire de labo, créer et gérer des labos](how-to-manage-classroom-labs.md)
- [En tant que propriétaire de labo, configurer et publier des modèles](how-to-create-manage-template.md)
- [En tant que propriétaire de labo, configurer et contrôler l’utilisation d’un labo](how-to-configure-student-usage.md)
- [En tant qu’utilisateur de labo, accéder aux labos](how-to-use-classroom-lab.md)
