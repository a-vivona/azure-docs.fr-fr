---
title: Créer un compte Azure Video Analyzer for Media (anciennement Video Indexer) connecté à Azure
titleSuffix: Azure Video Analyzer for Media
description: Découvrez comment créer un compte Azure Video Analyzer for Media (anciennement Video Indexer) connecté à Azure.
services: azure-video-analyzer
author: Juliako
manager: femila
ms.topic: article
ms.subservice: azure-video-analyzer-media
ms.date: 01/14/2021
ms.author: juliako
ms.openlocfilehash: 7bc5188c5987d393d63049a0a87a84605e4fa372
ms.sourcegitcommit: 0af634af87404d6970d82fcf1e75598c8da7a044
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/15/2021
ms.locfileid: "112121583"
---
# <a name="create-a-video-analyzer-for-media-account"></a>Créer un compte Video Analyzer for Media

Lorsque vous créez un compte Azure Video Analyzer for Media (anciennement Video Indexer), vous pouvez choisir un compte d’essai gratuit (où vous obtenez un certain nombre de minutes d’indexation gratuites) ou une option payante (où vous n’êtes pas limité par le quota). Avec un essai gratuit, Azure Video Analyzer for Media fournit jusqu’à 600 heures d’indexation gratuite aux utilisateurs du site web et jusqu’à 2400 heures d’indexation gratuite aux utilisateurs de l’API. Avec l’option payante, vous créez un compte Video Analyzer for Media connecté à votre abonnement Azure. Vous payez pour les minutes indexées ; pour plus d’informations, consultez [Tarification Media Services](https://azure.microsoft.com/pricing/details/media-services/).

Cet article explique comment créer un compte Video Analyzer for Media connecté à un abonnement Azure et un compte Azure Media Services. Cette rubrique indique les étapes permettant de se connecter à Azure via le flux automatique (par défaut). Elle explique également comment se connecter à Azure manuellement (option avancée).

Si vous passez d’un compte Video Analyzer for Media en *version d’essai* à la *version payante*, vous pouvez choisir de copier toutes les vidéos et la personnalisation du modèle sur le nouveau compte, comme indiqué dans la section [Importer votre contenu à partir du compte d’essai](#import-your-content-from-the-trial-account).

L’article aborde également [Liaison d’un compte Video Analyzer for Media à Azure Government](#video-analyzer-for-media-in-azure-government).

## <a name="prerequisites-for-connecting-to-azure"></a>Prérequis pour la connexion à Azure

* Un abonnement Azure.

    Si vous n’avez pas encore d’abonnement Azure, [inscrivez-vous pour bénéficier d’un essai gratuit Azure](https://azure.microsoft.com/free/).
* Un domaine Azure Active Directory (Azure AD).

    Si vous n’avez pas de domaine Azure AD, créez-en un avec votre abonnement Azure. Pour plus d’informations, consultez la rubrique [Gestion des noms de domaine personnalisés dans Azure AD](../../active-directory/enterprise-users/domains-manage.md)
* Un utilisateur de votre domaine Azure AD avec un rôle d'**administrateur d’application**. Vous allez utiliser ce membre lors de la connexion de votre compte Video Analyzer for Media à Azure.

    Cet utilisateur doit être un utilisateur Azure AD avec un compte professionnel ou scolaire. N’utilisez pas un compte personnel comme outlook.com, live.com ou hotmail.com.

    ![tous les utilisateurs Azure AD](./media/create-account/all-aad-users.png)

### <a name="additional-prerequisites-for-automatic-flow"></a>Prérequis supplémentaires pour le flux automatique

* Un utilisateur et un membre dans votre domaine Azure AD.

    Vous allez utiliser ce membre lors de la connexion de votre compte Video Analyzer for Media à Azure.

    Cet utilisateur doit être membre de votre abonnement Azure avec un rôle de **Propriétaire** ou les rôles de **Contributeur** et **d’Administrateur de l’accès utilisateur**. Un utilisateur peut être ajouté à deux reprises, avec deux rôles. Une fois comme contributeur et une autre fois comme administrateur des accès utilisateur. Pour plus d’informations, consultez [Visualiser l’accès dont dispose un utilisateur aux ressources Azure](../../role-based-access-control/check-access.md).

    ![Contrôle d’accès](./media/create-account/access-control-iam.png)

### <a name="additional-prerequisites-for-manual-flow"></a>Prérequis supplémentaires pour le flux manuel

* Inscrivez le fournisseur de ressources EventGrid à l’aide du portail Azure.

    Dans le [Portail Azure](https://portal.azure.com/), accédez à **Abonnements**-> [abonnement] ->**Fournisseurs de ressources**.

    Recherchez **Microsoft.Media** et **Microsoft.EventGrid**. S’il n’a pas l’état « Inscrit », cliquez sur **Inscrire**. Quelques minutes sont nécessaires pour effectuer l’inscription.

    ![EventGrid](./media/create-account/event-grid.png)

## <a name="create-a-new-account-on-azure"></a>Créer un nouveau compte dans Azure 

> [!NOTE]
> Si votre abonnement Azure utilise l’authentification multifacteur basée sur des certificats, il est essentiel d’effectuer les opérations suivantes sur un appareil sur lequel les certificats requis sont installés.

1. Accédez au site web [Video Analyzer for Media](https://www.videoindexer.ai/) et connectez-vous.
1. Sélectionnez le bouton **Créer un compte illimité** :

    ![Créer un compte Video Analyzer for Media](./media/create-account/create-unlimited-account.png)
1. Lorsque la liste des abonnements s’affiche, sélectionnez l’abonnement que vous souhaitez utiliser.

    ![Connecter Video Analyzer for Media à Azure](./media/create-account/new-account-on-azure-subscription.png)
1. Sélectionnez une région Azure parmi les lieux pris en charge : USA Ouest 2, Europe Nord et Asie-Pacifique.
1. Sous **Compte Azure Media Services**, choisissez une des options suivantes :

    * Pour créer un compte Media Services, sélectionnez **Créer un groupe de ressources**. Indiquez le nom de votre groupe de ressources.

        Azure crée ce compte dans votre abonnement, y compris un compte de stockage Azure.  
    * Pour utiliser un compte Media Services existant, sélectionnez **Utiliser une ressource existante**. Dans la liste des comptes, sélectionnez votre compte.

        Votre compte Media Services doit avoir la même région que votre compte Video Analyzer for Media.

        > [!NOTE]
        > Pour réduire la durée d’indexation et améliorer le débit, il est fortement recommandé d’ajuster le type et le nombre [d’unités réservées](../../media-services/previous/media-services-scale-media-processing-overview.md ) sur **10 unités réservées S3** dans votre compte Media Services. Consultez [Modification du type d’unité réservée](../../media-services/previous/media-services-portal-scale-media-processing.md). Les unités réservées sont facturées pour votre compte, voir [Détails de la tarification](https://azure.microsoft.com/pricing/details/media-services/#analytics).
    * Pour configurer manuellement votre connexion, sélectionnez le lien **Basculer vers la configuration manuelle**.

        Pour plus d’informations, consultez la section [Se connecter à Azure manuellement](#connect-to-azure-manually-advanced-option) (option avancée) ci-dessous.
1. Lorsque vous êtes prêt, choisissez **Créer**. Cette opération peut prendre quelques minutes.

    Une fois que vous êtes connecté à Azure, votre nouveau compte Video Analyzer for Media apparaît dans la liste des comptes :

    ![Nouveau compte](./media/create-account/new-account.png)
1. Assurez-vous que le point de terminaison de streaming du compte Media Services est en cours d’exécution avant de pouvoir lire vos vidéos dans l’application web Video Analyzer for Media (appuyez sur Démarrer si elle est dans l’état arrêté).

> [!TIP]
> Pour fournir un nom d’affichage convivial à votre compte, accédez à **Paramètres**.

## <a name="connect-to-azure-manually-advanced-option"></a>Se connecter à Azure manuellement (option avancée)

En cas d’échec de la connexion à Azure, vous pouvez tenter de résoudre le problème en vous connectant manuellement.

> [!NOTE]
> Il est vivement recommandé de disposer des trois comptes suivants dans la même région : le compte Video Analyzer for Media que vous connectez au compte Media Services, ainsi que le compte de stockage Azure connecté au même compte Media Services.

### <a name="create-and-configure-a-media-services-account"></a>Créer et configurer un compte Media Services

1. Utilisez le portail [Azure](https://portal.azure.com/) pour créer un compte Azure Media Services tel que décrit dans [Création d’un compte](../../media-services/previous/media-services-portal-create-account.md).

     Vérifiez que le compte Media Services a été créé avec les API classiques. 
 
    ![API classiques Media Services](./media/create-account/enable-classic-api.png)


    Lorsque vous créez un compte de stockage pour votre compte Media Services, sélectionnez **StorageV2** en tant que type de compte et **Stockage géo-redondant (GRS)** dans le champ Réplication.

    ![Nouveau compte AMS](./media/create-account/create-new-ams-account.png)

    > [!NOTE]
    > Veillez à noter les noms de ressource et de compte Media Services. Vous en aurez besoin lors des étapes de la prochaine section.
1. Ajustez le type et le nombre [d’unités réservées](../../media-services/previous/media-services-scale-media-processing-overview.md ) sur **10 unités réservées S3** dans le compte Media Services que vous avez créé. Consultez [Modification du type d’unité réservée](../../media-services/previous/media-services-portal-scale-media-processing.md).

    Les unités réservées sont facturées pour votre compte, voir [Détails de la tarification](https://azure.microsoft.com/pricing/details/media-services/#analytics).
1. Avant de pouvoir lire vos vidéos dans l’application web de Video Analyzer for Media, vous devez démarrer le **point de terminaison de streaming** par défaut du nouveau compte Media Services.

    Dans le nouveau compte Media Services, sélectionnez **Points de terminaison de streaming**. Sélectionnez ensuite le point de terminaison de streaming, puis cliquez sur Démarrer.

    ![Points de terminaison de streaming](./media/create-account/create-ams-account-se.png)
4. Pour que Video Analyzer for Media puisse s’authentifier auprès de l’API Media Services, vous devez créer une application AD. Les étapes suivantes vous guident dans le processus d’authentification Azure AD décrit dans [Prise en main de l’authentification Azure AD à l’aide du portail Azure](../../media-services/previous/media-services-portal-get-started-with-aad.md) :

    1. Dans le nouveau compte Media Services, sélectionnez **Accès API**.
    2. Sélectionnez [Service principal authentication method](../../media-services/previous/media-services-portal-get-started-with-aad.md) (Méthode d’authentification de principal de service).
    3. Obtenir l’ID client et la clé secrète client

        Après avoir sélectionné **Paramètres**->**Clés**, ajoutez une **Description** et cliquez sur **Enregistrer**. La valeur de clé est alors remplie automatiquement.

        Si la clé expire, le propriétaire du compte devra contacter le support de Video Analyzer for Media pour la renouveler.

        > [!NOTE]
        > Veillez à noter la valeur de clé et l’ID d’application. Vous en aurez besoin lors des étapes de la prochaine section.

### <a name="connect-manually"></a>Se connecter manuellement

Dans la boîte de dialogue **Créer un nouveau compte sur un abonnement Azure** de la page [Video Analyzer for Media](https://www.videoindexer.ai/), sélectionnez le lien **Basculer vers la configuration manuelle**.

Dans la boîte de dialogue, indiquez les informations suivantes :

|Paramètre|Description|
|---|---|
|Région du compte Video Analyzer for Media|Nom de la région du compte Video Analyzer for Media. Pour améliorer les performances et réduire les coûts, il est fortement recommandé de spécifier le nom de la région où se trouvent la ressource Azure Media Services et le compte de stockage Azure. |
|Client Azure AD|Nom du locataire Azure AD, par exemple contoso.onmicrosoft.com. Les informations relatives au locataire peuvent être récupérées à partir du portail Azure. Placez votre curseur sur le nom de l’utilisateur connecté, en haut à droite. Le nom est indiqué à droite du **domaine**.|
|Identifiant d’abonnement|Abonnement Azure sous lequel cette connexion doit être créée. L’ID d’abonnement peut être récupéré à partir du portail Azure. Sélectionnez **Tous les services** dans le panneau gauche, puis recherchez « abonnements ». Sélectionnez **Abonnements** et choisissez l’ID voulu dans la liste de vos abonnements.|
|Nom du groupe de ressources Azure Media Services|Nom du groupe de ressources dans lequel vous avez créé le compte Media Services.|
|Nom de la ressource de service multimédia|Nom du compte Azure Media Services que vous avez créé dans la section précédente.|
|ID de l'application|ID d’application Azure AD (avec des autorisations pour le compte Media Services spécifié) que vous avez créé dans la section précédente.|
|Clé de l'application|Clé de l’application Azure AD que vous avez créée dans la section précédente. |

### <a name="import-your-content-from-the-trial-account"></a>Importer votre contenu à partir du compte d’*essai*

Quand vous créez un compte, vous avez la possibilité d’importer votre contenu à partir du compte d’*essai* dans le nouveau compte. Si vous activez l’option d’*importation* dans la boîte de dialogue **Créer un compte sur un abonnement Azure**, toutes les personnalisations de modèle de contenu et de fichier multimédia sont copiées à partir du compte d’*essai* dans le nouveau compte.

La possibilité d’importer le contenu est valide pour les approches automatisée et manuelle décrites ci-dessus.

> [!NOTE]
> Le contenu ne peut être importé qu’une seule fois à partir de chaque compte.
>
> Le compte d’*évaluation gratuite* n’est pas disponible sur le cloud Azure Government.

## <a name="azure-media-services-considerations"></a>Considérations relatives à Azure Media Services

Tenez compte des considérations suivantes pour Azure Media Services :

* Si vous envisagez de vous connecter à un compte Media Services existant, assurez-vous que celui-ci a été créé avec les API classiques. 
 
    ![API classiques Media Services](./media/create-account/enable-classic-api.png)
* Si vous vous connectez à un compte Media Services existant, Video Analyzer for Media ne modifie pas la configuration existante pour les **unités réservées** multimédia.

   Vous devrez peut-être ajuster le type et le nombre d’unités réservées multimédia en fonction de la charge prévue. N’oubliez pas que si votre charge est élevée et si vous n’avez pas suffisamment unités ou manquez de vitesse, le traitement de vidéos peut échouer en raison d’une expiration de délai.
* Si vous vous connectez à un nouveau compte Media Services, Video Analyzer for Media démarre automatiquement son **point de terminaison de streaming** par défaut :

    ![Point de terminaison de diffusion en continu Media Services](./media/create-account/ams-streaming-endpoint.png)

    Les points de terminaison de streaming nécessitent un temps de démarrage important. Par conséquent, il peut s’écouler plusieurs minutes entre le moment où vous vous connectez à votre compte Azure et la diffusion et le visionnage de vos vidéos dans l’application web Video Analyzer for Media.
* Si vous vous connectez à un compte Media Services existant, Video Analyzer for Media ne modifie pas la configuration du point de terminaison de streaming par défaut. Si aucun **point de terminaison de streaming** n’est en cours d’exécution, vous ne pouvez pas regarder les vidéos à partir de ce compte Media Services, ni dans Video Analyzer for Media.
* Si vous vous connectez automatiquement, Video Analyzer for Media définit les **unités réservées** multimédia sur 10 unités S3 :

    ![Unités réservées Media Services](./media/create-account/ams-reserved-units.png)
    
## <a name="automate-creation-of-the-video-analyzer-for-media-account"></a>Automatiser la création du compte Video Analyzer for Media

L’automatisation de la création du compte est un processus en deux étapes :
 
1. Utilisez Azure Resource Manager pour créer un compte Azure Media Services et une application Azure AD.

    Consultez un exemple de [modèle de création de compte Media Services](https://github.com/Azure-Samples/media-services-v3-arm-templates).
1. Appelez [Create-Account avec l’application Media Services et Azure AD](https://videoindexer.ai.azure.us/account/login?source=apim).

## <a name="video-analyzer-for-media-in-azure-government"></a>Video Analyzer for Media dans Azure Government

### <a name="prerequisites-for-connecting-to-azure-government"></a>Prérequis pour la connexion à Azure Government

-   Abonnement Azure dans [Azure Government](../../azure-government/index.yml).
- Compte Azure AD dans Azure Government.
- Tous les prérequis des autorisations et des ressources décrits ci-dessus dans [Prérequis pour la connexion à Azure](#prerequisites-for-connecting-to-azure). Veillez à vérifier les [conditions préalables supplémentaires pour le flux automatique](#additional-prerequisites-for-automatic-flow) et les [conditions préalables supplémentaires pour le flux manuel](#additional-prerequisites-for-manual-flow).

### <a name="create-new-account-via-the-azure-government-portal"></a>Créer un nouveau compte via le portail Azure Government

> [!NOTE]
> Le cloud Azure Government n’inclut pas d’expérience d’*évaluation gratuite* de Video Analyzer for Media.

Pour créer un compte payant via le portail Video Analyzer for Media :

1. Accédez à https://videoindexer.ai.azure.us 
1. Connectez-vous avec votre compte Azure AD Azure Government.
1.  Si vous n’avez pas de compte Video Analyzer for Media dans Azure Government dont vous êtes le propriétaire ou le contributeur, vous obtenez une expérience vide à partir de laquelle vous pouvez commencer à créer votre compte. 

    Le reste du processus est décrit ci-dessus. Les régions à sélectionner sont les régions gouvernementales dans lesquelles Video Analyzer for Media est disponible. 

    Si vous êtes déjà le contributeur ou l’administrateur d’un ou plusieurs comptes Video Analyzer for Media dans Azure Government, vous êtes redirigé vers ce compte. À partir de là, vous pouvez commencer à créer un compte supplémentaire si nécessaire, comme décrit ci-dessus.
    
### <a name="create-new-account-via-the-api-on-azure-government"></a>Créer un nouveau compte via l’API sur Azure Government

Pour créer un compte payant dans Azure Government, suivez les instructions de [Création de compte payant](). Ce point de terminaison d’API comprend uniquement les régions du cloud Government.

### <a name="limitations-of-video-analyzer-for-media-on-azure-government"></a>Limitations de Video Analyzer for Media sur Azure Government

*   Aucune modération manuelle du contenu n’est disponible dans le cloud Government. 

    Dans le cloud public, quand un contenu est considéré comme offensant par la modération du contenu, le client peut demander l’examen du contenu par un être humain qui peut éventuellement annuler cette décision.  
*   Pas de compte d’évaluation gratuite. 
* Description Bing – dans le cloud Government, nous ne présentons pas de description de célébrités et d’entités nommées identifiées. Il s’agit d’une capacité d’interface utilisateur uniquement. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé ce tutoriel, supprimez les ressources que vous n’envisagez pas d’utiliser.

### <a name="delete-a-video-analyzer-for-media-account"></a>Supprimer un compte Video Analyzer for Media

Si vous souhaitez supprimer un compte Video Analyzer for Media, vous pouvez supprimer le compte du site web de Video Analyzer for Media. Pour supprimer le compte, vous devez en être le propriétaire.

Sélectionnez le compte -> **Paramètres** -> **Supprimer ce compte**. 

Le compte sera définitivement supprimé après 90 jours.

## <a name="firewall"></a>Pare-feu

Consultez [Compte de stockage qui se trouve derrière un pare-feu](faq.md#can-a-storage-account-connected-to-the-media-services-account-be-behind-a-firewall).

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez interagir par programmation avec votre compte d’évaluation ou avec les comptes Video Analyzer for Media connectés à Azure en suivant les instructions dans : [Utiliser les API](video-indexer-use-apis.md).

Vous devez vous servir du même utilisateur Azure AD que lors de la connexion à Azure.
