---
title: Détecter un problème à l’aide de la commande dsregcmd - Azure Active Directory
description: Utilisation de la sortie de dsregcmd pour comprendre l’état des appareils dans Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: spunukol
ms.collection: M365-identity-device-management
ms.openlocfilehash: ff1c0d1e552ad26832b2c142f5ca1506654a9a0c
ms.sourcegitcommit: 192444210a0bd040008ef01babd140b23a95541b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/15/2021
ms.locfileid: "114219535"
---
# <a name="troubleshooting-devices-using-the-dsregcmd-command"></a>Dépannage des appareils à l’aide de la commande dsregcmd

L’utilitaire dsregcmd/état doit être exécuté en tant que compte d’utilisateur de domaine.

## <a name="device-state"></a>État de l’appareil

Cette section répertorie les paramètres de l’état de jointure de l’appareil. Le tableau ci-dessous répertorie les critères pour que l’appareil se trouve dans différents états de jointure.

| AzureAdJoined | EnterpriseJoined | DomainJoined | État de l’appareil |
| ---   | ---   | ---   | ---   |
| YES | Non | Non | Joint à Azure AD |
| Non | Non | YES | Joint à un domaine |
| YES | Non | YES | Joint à AD hybride |
| Non | YES | YES | Joint à DRS localement |

> [!NOTE]
> L’état de Workplace Join (inscrit à Azure AD) est affiché dans la section « État utilisateur ».

- **AzureAdJoined :** défini sur « YES » si l’appareil est joint à Azure AD. « NO » dans le cas contraire.
- **EnterpriseJoined :** défini sur « YES » si l’appareil est joint à une instance Data Replication Service locale. Un appareil ne peut pas être à la fois EnterpriseJoined et AzureAdJoined.
- **DomainJoined :** défini sur « YES » si l’appareil est joint à un domaine (AD).
- **DomainName :** définit sur le nom du domaine si l’appareil est joint à un domaine.

### <a name="sample-device-state-output"></a>Exemple de sortie d’état de l’appareil

```
+----------------------------------------------------------------------+
| Device State                                                         |
+----------------------------------------------------------------------+
             AzureAdJoined : YES
          EnterpriseJoined : NO
              DomainJoined : YES
                DomainName : HYBRIDADFS
+----------------------------------------------------------------------+
```

## <a name="device-details"></a>Informations sur l’appareil

S’affiche uniquement lorsque l’appareil est joint à Azure AD ou à Azure AD hybride (pas inscrit dans Azure AD). Cette section répertorie les détails d’identification des appareils stockés dans Azure AD.

- **DeviceID :** ID unique de l’appareil dans le locataire Azure AD.
- **Empreinte :** empreinte du certificat de l’appareil.
- **DeviceCertificateValidity :** validité du certificat de l’appareil.
- **KeyContainerId :** -ContainerId de la clé privée de l’appareil associée au certificat de l’appareil
- **KeyProvider :** fournisseur de clé (matériel/logiciel) utilisé pour stocker la clé privée de l’appareil.
- **TpmProtected :** « YES » si la clé privée de l’appareil est stockée dans un module TPM matériel.

> [!NOTE]
> Le champ **DeviceAuthStatus** a été ajouté dans la **mise à jour de mai 2021 de Windows 10 (version 21H1)** .

- **DeviceAuthStatus :** Effectue une vérification pour déterminer l’intégrité de l’appareil dans Azure AD.  
« SUCCESS » si l’appareil est présent et activé dans Azure AD.  
"FAILED. L’appareil est désactivé ou supprimé » si l’appareil est désactivé ou supprimé, [En savoir plus](faq.yml#why-do-my-users-see-an-error-message-saying--your-organization-has-deleted-the-device--or--your-organization-has-disabled-the-device--on-their-windows-10-devices).  
"FAILED. ERREUR » si le test n’a pas pu s’exécuter. Ce test nécessite une connexion réseau pour Azure AD.  

### <a name="sample-device-details-output"></a>Exemple de sortie des détails de l’appareil

```
+----------------------------------------------------------------------+
| Device Details                                                       |
+----------------------------------------------------------------------+

                  DeviceId : e92325d0-xxxx-xxxx-xxxx-94ae875dxxxx
                Thumbprint : D293213EF327483560EED8410CAE36BB67208179
 DeviceCertificateValidity : [ 2019-01-11 21:02:50.000 UTC -- 2029-01-11 21:32:50.000 UTC ]
            KeyContainerId : 13e68a58-xxxx-xxxx-xxxx-a20a2411xxxx
               KeyProvider : Microsoft Software Key Storage Provider
              TpmProtected : NO
          DeviceAuthStatus : SUCCESS
+----------------------------------------------------------------------+
```

## <a name="tenant-details"></a>Détails des locataires

S’affiche uniquement lorsque l’appareil est joint à Azure AD ou à Azure AD hybride (pas inscrit dans Azure AD). Cette section répertorie les détails courants des locataires lorsqu’un appareil est joint à Azure AD.

> [!NOTE]
> Si les URL de la Gestion des données de référence de cette section sont vides, cela indique que la GPM est soit non configurée soit que l’utilisateur actuel n’est pas dans l’étendue de l’accord de mise en œuvre de la GPM. Vérifiez les paramètres de mobilité dans Azure AD pour vérifier votre configuration GPM.

> [!NOTE]
> Même si vous voyez des URL MDM, cela ne signifie pas que l’appareil est géré par un MDM. Les informations s’affichent si le locataire a une configuration MDM pour l’inscription automatique, même si l’appareil proprement dit n’est pas géré.

### <a name="sample-tenant-details-output"></a>Exemple de sortie de détails du locataire

```
+----------------------------------------------------------------------+
| Tenant Details                                                       |
+----------------------------------------------------------------------+

                TenantName : HybridADFS
                  TenantId : 96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx
                       Idp : login.windows.net
               AuthCodeUrl : https://login.microsoftonline.com/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/oauth2/authorize
            AccessTokenUrl : https://login.microsoftonline.com/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/oauth2/token
                    MdmUrl : https://enrollment.manage-beta.microsoft.com/EnrollmentServer/Discovery.svc
                 MdmTouUrl : https://portal.manage-beta.microsoft.com/TermsOfUse.aspx
          MdmComplianceUrl : https://portal.manage-beta.microsoft.com/?portalAction=Compliance
               SettingsUrl : eyJVxxxxIjpbImh0dHBzOi8va2FpbGFuaS5vbmUubWljcm9zb2Z0LmNvbS8iLCJodHRwczovL2thaWxhbmkxLm9uZS5taWNyb3NvZnQuY29tLyxxxx==
            JoinSrvVersion : 1.0
                JoinSrvUrl : https://enterpriseregistration.windows.net/EnrollmentServer/device/
                 JoinSrvId : urn:ms-drs:enterpriseregistration.windows.net
             KeySrvVersion : 1.0
                 KeySrvUrl : https://enterpriseregistration.windows.net/EnrollmentServer/key/
                  KeySrvId : urn:ms-drs:enterpriseregistration.windows.net
        WebAuthNSrvVersion : 1.0
            WebAuthNSrvUrl : https://enterpriseregistration.windows.net/webauthn/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/
             WebAuthNSrvId : urn:ms-drs:enterpriseregistration.windows.net
    DeviceManagementSrvVer : 1.0
    DeviceManagementSrvUrl : https://enterpriseregistration.windows.net/manage/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/
     DeviceManagementSrvId : urn:ms-drs:enterpriseregistration.windows.net
+----------------------------------------------------------------------+
```

## <a name="user-state"></a>État utilisateur

Cette section répertorie l’état de différents attributs pour l’utilisateur actuellement connecté à l’appareil.

> [!NOTE]
> La commande doit s’exécuter dans un contexte utilisateur pour récupérer un état valide.

- **NgcSet :** défini sur « YES » si une clé Windows Hello est définie pour l’utilisateur actuellement connecté.
- **NgcSet :** ID de la clé Windows Hello si une telle clé est définie pour l’utilisateur actuellement connecté.
- **CanReset :** indique si la clé Windows Hello peut être réinitialisée par l’utilisateur.
- **Valeurs possibles :** DestructiveOnly, NonDestructiveOnly, DestructiveAndNonDestructive ou Unknown en cas d’erreur.
- **WorkplaceJoined :** défini sur « YES » si des comptes inscrits à Azure AD ont été ajoutés à l’appareil dans le contexte NTUSER actuel.
- **WamDefaultSet :** défini sur « YES » si un compte WAM par défaut est créé pour l’utilisateur connecté. Ce champ peut afficher une erreur si dsregcmd /status est exécuté dans une invite de commandes avec élévation de privilèges.
- **WamDefaultAuthority :** défini sur « organisations » pour Azure AD.
- **WamDefaultId :** toujours « https://login.microsoft.com  » pour Azure AD.
- **WamDefaultGUID :** GUID du fournisseur WAM (compte Azure AD/Microsoft) pour le compte WAM par défaut.

### <a name="sample-user-state-output"></a>Exemple de sortie d’état utilisateur

```
+----------------------------------------------------------------------+
| User State                                                           |
+----------------------------------------------------------------------+

                    NgcSet : YES
                  NgcKeyId : {FA0DB076-A5D7-4844-82D8-50A2FB42EC7B}
                  CanReset : DestructiveAndNonDestructive
           WorkplaceJoined : NO
             WamDefaultSet : YES
       WamDefaultAuthority : organizations
              WamDefaultId : https://login.microsoft.com
            WamDefaultGUID : { B16898C6-A148-4967-9171-64D755DA8520 } (AzureAd)

+----------------------------------------------------------------------+
```

## <a name="sso-state"></a>État de la SSO

Cette section peut être ignorée pour les appareils inscrits à Azure AD.

> [!NOTE]
> La commande doit s’exécuter dans un contexte utilisateur afin de récupérer un état valide pour cet utilisateur.

- **AzureAdPrt :** défini sur « YES » si un PRT est présent sur l’appareil pour l’utilisateur connecté.
- **AzureAdPrtUpdateTime :** défini sur l’heure UTC de la dernière mise à jour du PRT.
- **AzureAdPrtExpiryTime :** défini sur l’heure UTC de l’expiration du PRT s’il n’est pas renouvelé.
- **AzureAdPrtAuthority :** URL d’autorité Azure AD.
- **EnterprisePrt :** défini sur « YES » si l’appareil dispose d’un PRT à partir d’ADFS local. Pour les appareils hybrides Azure AD joints, l’appareil peut avoir un PRT provenant à la fois d’Azure AD et d’AD local. Les appareils joints locaux ne disposent que d’un PRT d’entreprise.
- **EnterprisePrtUpdateTime :** défini sur l’heure UTC de la dernière mise à jour du PRT d’entreprise.
- **EnterprisePrtExpiryTime :** défini sur l’heure UTC de l’expiration du PRT s’il n’est pas renouvelé.
- **EnterprisePrtAuthority :** URL de l’autorité ADFS.

>[!NOTE]
> Les champs de diagnostic PRT suivants ont été ajoutés dans **la mise à jour de mai 2021 de Windows 10 (version 21H1)** .

>[!NOTE]
> Les informations de diagnostic affichées sous **AzureAdPrt** sont pour les informations d’acquisition/actualisation et de diagnostic AzureAD PRT affichées sous **EnterprisePrt**  et pour l’acquisition/actualisation d’Enterprise PRT respectivement.

>[!NOTE]
>Les informations de diagnostic s’affichent uniquement si l’échec de l’acquisition/actualisation s’est produit après l’heure de la dernière mise à jour PRT réussie (AzureAdPrtUpdateTime/EnterprisePrtUpdateTime).  
>Sur un appareil partagé, ces informations de diagnostic peuvent être une tentative d’ouverture de session d’un autre utilisateur.

- **AcquirePrtDiagnostics :** Défini sur « PRESENT » si les informations de diagnostic PRT sont présentes dans les journaux.  
Ce champ est ignoré si aucune information de diagnostic n’est disponible.
- **Tentative de PRT précédente :** Heure locale UTC à laquelle la tentative de PRT a échoué.  
- **État de la tentative :** code d’erreur client retourné (HRESULT).
- **Identité utilisateur :** Le nom d’utilisateur principal de l’utilisateur pour lequel la tentative de PRT a eu lieu.
- **Type d’informations d’identification :** Informations d’identification utilisées pour acquérir/actualiser PRT. Les types d’informations d’identification courants sont Mot de passe et NGC (Windows Hello).
- **Identification de corrélation :** L’ID de corrélation envoyé par le serveur pour l’échec de la tentative de PRT.
- **URI du point de terminaison :** Dernier accès de point de terminaison avant l’échec.
- **Méthode HTTP :** La méthode HTTP utilisée pour accéder au point de terminaison.
- **Erreur HTTP :** Code d’erreur de transport WinHttp. Les erreurs WinHttp peuvent être consultées [ici](/windows/win32/winhttp/error-messages).
- **État HTTP :** l’état HTTP retourné par le point de terminaison.
- **Code d’erreur du serveur :** Le code d’erreur du serveur.  
- **Description de l’erreur du serveur :** Le message d’erreur du serveur.
- **RefreshPrtDiagnostics :** Défini sur « PRESENT » si les informations de diagnostic PRT sont présentes dans les journaux.  
Ce champ est ignoré si aucune information de diagnostic n’est disponible.
Les champs d’informations de diagnostic sont identiques à ceux de **AcquirePrtDiagnostics**


### <a name="sample-sso-state-output"></a>Exemple de sortie d’état SSO

```
+----------------------------------------------------------------------+
| SSO State                                                            |
+----------------------------------------------------------------------+

                AzureAdPrt : NO
       AzureAdPrtAuthority : https://login.microsoftonline.com/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx
     AcquirePrtDiagnostics : PRESENT
      Previous Prt Attempt : 2020-07-18 20:10:33.789 UTC
            Attempt Status : 0xc000006d
             User Identity : john@contoso.com
           Credential Type : Password
            Correlation ID : 63648321-fc5c-46eb-996e-ed1f3ba7740f
              Endpoint URI : https://login.microsoftonline.com/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/oauth2/token/
               HTTP Method : POST
                HTTP Error : 0x0
               HTTP status : 400
         Server Error Code : invalid_grant
  Server Error Description : AADSTS50126: Error validating credentials due to invalid username or password.
             EnterprisePrt : YES
   EnterprisePrtUpdateTime : 2019-01-24 19:15:33.000 UTC
   EnterprisePrtExpiryTime : 2019-02-07 19:15:33.000 UTC
    EnterprisePrtAuthority : https://fs.hybridadfs.nttest.microsoft.com:443/adfs

+----------------------------------------------------------------------+
```

## <a name="diagnostic-data"></a>Données de diagnostic

### <a name="pre-join-diagnostics"></a>Diagnostics de préjointure

Cette section s’affiche uniquement si l’appareil est joint à un domaine et ne peut pas joindre Azure AD hybride.

Dans cette section, différents tests sont effectués pour faciliter le diagnostic des échecs de jointure. Cette section contient également les détails de (?) précédent(e). Ces informations incluent la phase d’erreur, le code d’erreur, l’ID de la requête serveur, l’état http de la réponse du serveur, le message d’erreur de réponse du serveur.

- **User Context :** contexte dans lequel les diagnostics sont exécutés. Valeurs possibles : SYSTEM (système), utilisateur UN-ELEVATED (sans élévation de privilèges), utilisateur ELEVATED (avec élévation de privilèges).

   > [!NOTE]
   > Étant donné que la jointure réelle est effectuée dans le contexte SYSTEM, l’exécution des diagnostics dans le contexte SYSTEM est plus proche du scénario de jointure réel. Pour exécuter des diagnostics dans le contexte SYSTEM, la commande dsregcmd/état doit être exécutée à partir d’une invite de commandes avec élévation de privilèges.

- **Client Time :** heure UTC du système.
- **AD Connectivity Test :** test de connectivité effectué sur le contrôleur de domaine. Une erreur dans ce test entraînera probablement des erreurs de jointure lors de la phase de vérification préalable.
- **AD Configuration Test** : test permettant de lire et de vérifier si l’objet SCP est correctement configuré dans la forêt AD locale. Des erreurs dans ce test entraîneraient probablement des erreurs de jointure dans la phase de découverte avec le code d’erreur 0x801c001d.
- **DRS Discovery Test :** test obtenant les points de terminaison Data Replication Service à partir du point de terminaison des métadonnées de découverte, et exécutant une requête de domaine d’utilisateur. Des erreurs dans ce test entraîneraient probablement des erreurs de jointure dans la phase de découverte.
- **DRS Connectivity Test :** test de connectivité de base effectué sur le point de terminaison Data Replication Service.
- **Token acquisition Test :**  test tentant d’obtenir un jeton d’authentification Azure AD si le locataire de l’utilisateur est fédéré. Des erreurs dans ce test entraîneraient probablement des erreurs de jointure dans la phase d’authentification. En cas d’échec de l’authentification, une jointure de synchronisation est tentée en tant qu’option de secours, sauf si celle-ci est explicitement désactivée avec les paramètres de clé de registre ci-dessous.

```
Keyname: Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\CDJ
Value: FallbackToSyncJoin
Type:  REG_DWORD
Value: 0x0 -> Disabled
Value: 0x1 -> Enabled
Default (No Key): Enabled
```

- **Fallback to Sync-Join :** défini sur « Enabled » (Activé) si la clé de Registre ci-dessus n’est PAS présente afin d’empêcher l’option de secours de jointure de synchronisation en cas d’échec de l’authentification. Cette option est disponible à partir de Windows 10 1803 et versions ultérieures.
- **Previous Registration :** heure à laquelle la tentative de jointure précédente a eu lieu. Seules les tentatives de jointure ayant échoué sont journalisées.
- **Error Phase :** étape de la jointure où celle-ci a été abandonnée. Les valeurs possibles sont prévérification, détection, authentification et jointure.
- **Client ErrorCode :** code d’erreur client retourné (HRESULT).
- **Server ErrorCode :** code d’erreur serveur si une requête a été envoyée au serveur et que le serveur a retourné un code d’erreur.
- **Server Message** : message de serveur retourné avec le code d’erreur.
- **Https Status :** état HTTP retourné par le serveur.
- **Request ID :** identifiant de requête du client envoyé au serveur. Utile pour la corrélation avec les journaux côté serveur.

### <a name="sample-pre-join-diagnostics-output"></a>Exemple de sortie de diagnostics de préjointure

L’exemple suivant montre l’échec du test de diagnostic avec une erreur de détection.

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

     Diagnostics Reference : www.microsoft.com/aadjerrors
              User Context : SYSTEM
               Client Time : 2019-01-31 09:25:31.000 UTC
      AD Connectivity Test : PASS
     AD Configuration Test : PASS
        DRS Discovery Test : FAIL [0x801c0021/0x801c000c]
     DRS Connectivity Test : SKIPPED
    Token acquisition Test : SKIPPED
     Fallback to Sync-Join : ENABLED

     Previous Registration : 2019-01-31 09:23:30.000 UTC
               Error Phase : discover
          Client ErrorCode : 0x801c0021

+----------------------------------------------------------------------+
```

L’exemple suivant montre que les tests de diagnostic sont réussis, mais que la tentative d’inscription a échoué avec une erreur d’annuaire, ce qui est attendu pour la jointure de synchronisation. Une fois le travail de synchronisation Azure AD Connect terminé, l’appareil peut être joint.

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

     Diagnostics Reference : www.microsoft.com/aadjerrors
              User Context : SYSTEM
               Client Time : 2019-01-31 09:16:50.000 UTC
      AD Connectivity Test : PASS
     AD Configuration Test : PASS
        DRS Discovery Test : PASS
     DRS Connectivity Test : PASS
    Token acquisition Test : PASS
     Fallback to Sync-Join : ENABLED

     Previous Registration : 2019-01-31 09:16:43.000 UTC
         Registration Type : sync
               Error Phase : join
          Client ErrorCode : 0x801c03f2
          Server ErrorCode : DirectoryError
            Server Message : The device object by the given id (e92325d0-7ac4-4714-88a1-94ae875d5245) is not found.
              Https Status : 400
                Request Id : 6bff0bd9-820b-484b-ab20-2a4f7b76c58e

+----------------------------------------------------------------------+
```

### <a name="post-join-diagnostics"></a>Diagnostics postérieurs à la jointure

Cette section affiche la sortie des vérifications d’intégrité effectuées sur un appareil joint au cloud.

- **AadRecoveryEnabled :** si « YES », les clés stockées dans l’appareil ne sont pas utilisables et l’appareil est marqué pour la récupération. La connexion suivante déclenche le flux de récupération et réinscrit l’appareil.
- **KeySignTest :** « PASSED » (RÉUSSITE) indique que les clés d’appareil sont saines. Si KeySignTest échoue, l’appareil est généralement marqué pour la récupération. La connexion suivante déclenche le flux de récupération et réinscrit l’appareil. Pour les appareils hybrides Azure AD joints, la récupération est silencieuse. Pendant la jonction ou l’inscription d’Azure AD, les appareils demandent l’authentification utilisateur pour récupérer et réinscrire l’appareil si nécessaire. **Le test KeySignTest requiert des privilèges élevés.**

#### <a name="sample-post-join-diagnostics-output"></a>Exemple de sortie de diagnostics après la jointure

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

         AadRecoveryEnabled: NO
               KeySignTest : PASSED
+----------------------------------------------------------------------+
```

## <a name="ngc-prerequisite-check"></a>Vérification du prérequis NGC

Cette section effectue les vérifications des conditions préalables de l’approvisionnement de Windows Hello Entreprise (WHFB).

> [!NOTE]
> Il est possible que les détails des vérifications des conditions préalables ne soient pas visibles dans dsregcmd/status si l’utilisateur a déjà correctement configuré WHFB.

- **IsDeviceJoined :**  défini sur « YES » si l’appareil est joint à Azure AD.
- **IsUserAzureAD:**  défini sur « YES » si l’utilisateur connecté est présent dans Azure AD.
- **PolicyEnabled:**  défini sur « YES » si la stratégie WHFB est activée sur l’appareil.
- **PostLogonEnabled:**  défini sur « YES » si l’inscription WHFB est déclenchée en mode natif par la plateforme. Si la valeur définie est « NO », cela indique que l’inscription Windows Hello Entreprise est déclenchée par un mécanisme personnalisé
- **DeviceEligible:**  défini sur « YES » si l’appareil respecte la configuration matérielle exigée pour l’inscription auprès de WHFB.
- **SessionIsNotRemote:**  défini sur « YES » si l’utilisateur actuel est connecté à l’appareil directement, et non à distance.
- **CertEnrollment:**  spécifique au déploiement d’approbation avec certificat WHFB, indiquant l’autorité d’inscription de certificats pour WHFB. Défini sur « enrollment authority » (autorité d’inscription) si la source de la stratégie WHFB est la Stratégie de groupe, ou sur « mobile device management » (gestion des appareils mobiles) si la source est MDM. Sinon, « none » (aucun).
- **AdfsRefreshToken:**  spécifique au déploiement d’approbation avec certificat WHFB. Présent uniquement si CertEnrollment a la valeur « enrollment authority » (autorité d’inscription). Indique si l’appareil possède un PRT d’entreprise pour l’utilisateur.
- **AdfsRaIsReady:** spécifique au déploiement d’approbation avec certificat WHFB.  Présent uniquement si CertEnrollment a la valeur « enrollment authority » (autorité d’inscription). Défini sur « YES » si ADFS indiquait dans les métadonnées de découverte qu’il prend en charge WHFB *et* si un modèle de certificat d’ouverture de session est disponible.
- **LogonCertTemplateReady :** spécifique au déploiement d’approbation avec certificat WHFB. Présent uniquement si CertEnrollment a la valeur « enrollment authority » (autorité d’inscription). Défini sur « YES » si l’état du modèle de certificat d’ouverture de session est valide et permet de résoudre les problèmes liés à l’autorité d’inscription ADFS.
- **PreReqResult :**  fournit le résultat de l’évaluation de toutes les conditions préalables de WHFB. Défini sur « Will Provision » (Provisionnement ultérieur) si l’inscription de WHFB est lancée en tant que tâche après ouverture de session lors de la prochaine connexion de l’utilisateur.

### <a name="sample-ngc-prerequisite-check-output"></a>Exemple de sortie de vérification du prérequis NGC

```
+----------------------------------------------------------------------+
| Ngc Prerequisite Check                                               |
+----------------------------------------------------------------------+

            IsDeviceJoined : YES
             IsUserAzureAD : YES
             PolicyEnabled : YES
          PostLogonEnabled : YES
            DeviceEligible : YES
        SessionIsNotRemote : YES
            CertEnrollment : enrollment authority
          AdfsRefreshToken : YES
             AdfsRaIsReady : YES
    LogonCertTemplateReady : YES ( StateReady )
              PreReqResult : WillProvision
+----------------------------------------------------------------------+
```

## <a name="next-steps"></a>Étapes suivantes

- [Outil de recherche d’erreurs Microsoft](/windows/win32/debug/system-error-code-lookup-tool)
