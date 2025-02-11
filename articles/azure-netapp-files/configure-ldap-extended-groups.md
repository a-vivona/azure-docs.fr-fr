---
title: Configurer le protocole LDAP avec groupes étendus ADDS pour l’accès à un volume NFS Azure NetApp Files | Microsoft Docs
description: Décrit les considérations et les étapes à suivre pour activer LDAP avec groupes étendus lorsque vous créez un volume NFS à l’aide d’Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/19/2021
ms.author: b-juche
ms.openlocfilehash: 01d8f23331525443a9f83245b8eec2b402e92d6e
ms.sourcegitcommit: 7d63ce88bfe8188b1ae70c3d006a29068d066287
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/22/2021
ms.locfileid: "114443019"
---
# <a name="configure-adds-ldap-with-extended-groups-for-nfs-volume-access"></a>Configurer ADDS LDAP avec des groupes étendus pour l’accès au volume NFS

Lorsque vous [créez un volume NFS](azure-netapp-files-create-volumes.md), vous avez la possibilité d’activer la fonctionnalité LDAP avec groupes étendus (option **LDAP**) pour le volume. Cette fonctionnalité permet aux utilisateurs du protocole LDAP Active Directory et aux groupes étendus (jusqu’à 1 024 groupes) d’accéder au volume. Vous pouvez utiliser la fonctionnalité LDAP avec les groupes étendus pour les volumes NFSv 4.1 et NFSv3. 

Cet article explique les considérations et les étapes à suivre pour activer le protocole LDAP avec groupes étendus lorsque vous créez un volume NFS.  

## <a name="considerations"></a>Considérations

* Vous pouvez activer la fonctionnalité LDAP avec les groupes étendus uniquement lors de la création du volume. Cette fonctionnalité ne peut pas être activée rétroactivement sur des volumes existants.  

* LDAP avec groupes étendus est pris en charge uniquement avec Active Directory Domain Services (ADDS) ou Azure Active Directory Domain Services (AADDS). OpenLDAP et les autres services d’annuaire LDAP tiers ne sont pas pris en charge. 

* LDAP sur TLS *ne doit pas* être activé si vous utilisez Azure Active Directory Domain Services (AADDS).  

* Vous ne pouvez pas modifier le paramètre de l’option LDAP (activé ou désactivé) après avoir créé le volume.  

* Le tableau suivant décrit les paramètres de durée de vie (TTL) du cache LDAP. Vous devez attendre que le cache soit actualisé avant d’essayer d’accéder à un fichier ou à un répertoire par le biais d’un client. Dans le cas contraire, un message d’accès refusé s’affiche sur le client. 

    |     État d’erreur    |     Résolution    |
    |-|-|
    | Cache |  Délai d’expiration par défaut |
    | Liste d’appartenance au groupe  | 24-hour TTL  |
    | Groupes Unix  | TTL de 24 heures, TTL négative de 1 minute  |
    | Utilisateurs Unix  | TTL de 24 heures, TTL négative de 1 minute  |

    Les caches ont un délai d’expiration spécifique appelé *durée de vie*. Après le délai d’expiration, les entrées vieillissent, de sorte que les entrées obsolètes ne persistent pas. La valeur de *TTL négative* correspond au moment où une recherche ayant échoué réside pour éviter des problèmes de performances dus à des requêtes LDAP portant sur des objets qui n’existent peut-être pas.        

## <a name="steps"></a>Étapes

1. La fonctionnalité LDAP avec groupes étendus est actuellement en préversion. Avant d’utiliser cette fonctionnalité pour la première fois, vous devez commencer par l’inscrire :  

    1. Inscrivez la fonctionnalité :   

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapExtendedGroups
        ```

    2. Vérifiez l’état d’inscription de la fonctionnalité : 

        > [!NOTE]
        > **RegistrationState** peut être à l’état `Registering` pendant plusieurs minutes, et jusqu’à 60 minutes, avant de passer à l’état `Registered`. Avant de continuer, attendez que l’état soit `Registered`.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapExtendedGroups
        ```
        
    Vous pouvez également utiliser les [commandes Azure CLI](/cli/azure/feature) `az feature register` et `az feature show` pour inscrire la fonctionnalité et afficher l’état de l’inscription. 

2. Les volumes LDAP requièrent une configuration Active Directory pour les paramètres de serveur LDAP. Pour configurer des connexions Active Directory sur le portail Azure, suivez les instructions des rubriques [Configuration requise pour les connexions Active Directory](create-active-directory-connections.md#requirements-for-active-directory-connections) et [Créer une connexion Active Directory](create-active-directory-connections.md#create-an-active-directory-connection).  

    > [!NOTE]
    > Vérifiez que vous avez configuré les paramètres de connexion Active Directory. Un compte d’ordinateur est créé dans l’unité d’organisation (UO) spécifiée dans les paramètres de connexion d’Active Directory. Les paramètres sont utilisés par le client LDAP pour s’authentifier auprès de votre instance Active Directory.

3. Assurez-vous que le serveur LDAP Active Directory est opérationnel sur Active Directory. 

4. Les utilisateurs NFS de LDAP doivent avoir certains attributs POSIX sur le serveur LDAP. Définissez les attributs pour les utilisateurs LDAP et les groupes LDAP comme suit : 

    * Attributs requis pour les utilisateurs LDAP :   
        `uid: Alice`, `uidNumber: 139`, `gidNumber: 555`, `objectClass: user`
    * Attributs requis pour les groupes LDAP :   
        `objectClass: group`, `gidNumber: 555`

    Vous pouvez gérer les attributs POSIX à l’aide du composant logiciel enfichable MMC Utilisateurs et ordinateurs Active Directory. L'exemple suivant illustre l’Éditeur d’attributs Active Directory :  

    ![Éditeur d’attributs Active Directory](../media/azure-netapp-files/active-directory-attribute-editor.png) 

    Pour plus d’informations, consultez [Accéder à l’éditeur d’attribut Active Directory](create-volumes-dual-protocol.md#access-active-directory-attribute-editor) .  

5. Si vous souhaitez configurer un client Linux NFSv4.1 intégré à LDAP, consultez [Configurer un client NFS pour Azure NetApp Files](configure-nfs-clients.md).

6.  Pour créer un volume NFS, suivez les étapes décrites dans [Créer un volume NFS pour Azure NetApp Files](azure-netapp-files-create-volumes.md). Pendant le processus de création du volume, sous l’onglet **Protocole**, activez l’option **LDAP**.   

    ![Capture d’écran montrant la page Créer un volume avec l’option LDAP.](../media/azure-netapp-files/create-nfs-ldap.png)  

7. Facultatif : vous pouvez autoriser les utilisateurs du client NFS local qui ne sont pas présents sur le serveur LDAP Windows à accéder à un volume NFS sur lequel la fonctionnalité LDPA avec groupes étendus est activée. Pour ce faire, activez l’option **Autoriser les utilisateurs NFS locaux avec LDAP**, comme suit :
    1. Cliquez sur **Connexions Active Directory**.  Sur une connexion Active Directory existante, cliquez sur le menu contextuel (les trois points `…`), puis sélectionnez **Modifier**.  
    2. Dans la fenêtre **Modifier les paramètres Active Directory** qui s’affiche, sélectionnez l’option **Autoriser les utilisateurs NFS locaux avec LDAP**.  

    ![Capture d’écran montrant l’option Autoriser les utilisateurs NFS locaux avec LDAP](../media/azure-netapp-files/allow-local-nfs-users-with-ldap.png)  

## <a name="next-steps"></a>Étapes suivantes  

* [Créer un volume NFS pour Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Créer et gérer des connexions Active Directory](create-active-directory-connections.md)
* [Résoudre les problèmes liés au volume LDAP](troubleshoot-ldap-volumes.md)
