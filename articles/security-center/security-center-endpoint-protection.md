---
title: Suggestions de protection de point de terminaison dans Azure Security Centers
description: Découvrez comment les solutions de protection de point de terminaison sont détectées et identifiées comme saines.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 07/21/2021
ms.author: memildin
ms.openlocfilehash: 841d52b53fd492f1b9f6760deba438f68a47d004
ms.sourcegitcommit: 7d63ce88bfe8188b1ae70c3d006a29068d066287
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/22/2021
ms.locfileid: "114470709"
---
# <a name="endpoint-protection-assessment-and-recommendations-in-azure-security-center"></a>Évaluation de la protection de point de terminaison et recommandations dans Azure Security Center

Azure Security Center fournit des évaluations d’intégrité des versions [prises en charge](security-center-services.md#endpoint-supported) des solutions de protection de point de terminaison. Cet article explique les scénarios qui conduisent Security Center à générer les deux recommandations suivantes :

- [Endpoint Protection doit être installé sur vos machines](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/4fb67663-9ab9-475d-b026-8c544cced439)
- [Les problèmes d’intégrité de la protection du point de terminaison doivent être résolus sur vos machines](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/37a3689a-818e-4a0e-82ac-b1392b9bb000)

## <a name="windows-defender"></a>Windows Defender

- Security Center formule la recommandation **Endpoint Protection doit être installé sur vos machines** quand la commande [Get-MpComputerStatus](/powershell/module/defender/get-mpcomputerstatus) est exécutée et que le résultat est **AMServiceEnabled : False**

- Security Center formule la recommandation **Les problèmes d’intégrité Endpoint Protection doivent être résolus sur vos machines** quand la commande [Get-MpComputerStatus](/powershell/module/defender/get-mpcomputerstatus) est exécutée et que l’une des situations suivantes se présente :

  * L’une des propriétés suivantes est false :

    - **AMServiceEnabled**
    - **AntispywareEnabled**
    - **RealTimeProtectionEnabled**
    - **BehaviorMonitorEnabled**
    - **IoavProtectionEnabled**
    - **OnAccessProtectionEnabled**

  * Si au moins une des deux propriétés suivantes a une valeur égale ou supérieure à 7 :

    - **AntispywareSignatureAge**
    - **AntivirusSignatureAge**

## <a name="microsoft-system-center-endpoint-protection"></a>Protection du point de terminaison Microsoft System Center

* Security Center formule la recommandation **Endpoint Protection doit être installé sur vos machines** lors de l’importation de **SCEPMpModule ("$env:ProgramFiles\Microsoft Security Client\MpProvider\MpProvider.psd1")** quand l’exécution de la commande **Get-MProtComputerStatus** retourne **AMServiceEnabled = false**.

* Security Center formule la recommandation **Les problèmes d’intégrité Endpoint Protection doivent être résolus sur vos machines** quand la commande **Get-MprotComputerStatus** est exécutée et que l’une des situations suivantes se présente :

  * Au moins une des propriétés suivantes est False :

    - **AMServiceEnabled**
    - **AntispywareEnabled**
    - **RealTimeProtectionEnabled**
    - **BehaviorMonitorEnabled**
    - **IoavProtectionEnabled**
    - **OnAccessProtectionEnabled**

  * Si une voire les deux mises à jour de signature sont supérieures ou égales à 7 :

    * **AntispywareSignatureAge**
    * **AntivirusSignatureAge**

## <a name="trend-micro"></a>Trend Micro

* Security Center formule la recommandation **Endpoint Protection doit être installé sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :
    - **HKLM:\SOFTWARE\TrendMicro\Deep Security Agent** existe
    - **HKLM:\SOFTWARE\TrendMicro\Deep Security Agent\InstallationFolder** existe
    - Le fichier **dsa_query.cmd** se trouve dans le dossier d’installation
    - L'exécution de **dsa_query.cmd** indique **Component.AM.mode : on - Trend Micro Deep Security Agent détecté**

## <a name="symantec-endpoint-protection"></a>Protection de point de terminaison Symantec
Security Center formule la recommandation **Endpoint Protection doit être installé sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :

- **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\PRODUCTNAME = "Symantec Endpoint Protection"**
- **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\ASRunningStatus = 1**

ou

- **HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection\CurrentVersion\PRODUCTNAME = "Symantec Endpoint Protection"**
- **HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\ASRunningStatus = 1**

Security Center formule la recommandation **Les problèmes d’intégrité Endpoint Protection doivent être résolus sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :

- Vérifier la version de Symantec > = 12 : Emplacement du registre : **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion" -Value "PRODUCTVERSION"**
- Vérifier l'état de la protection en temps réel : **HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection\AV\Storages\Filesystem\RealTimeScan\OnOff == 1**
- Vérifier l’état de la mise à jour de la signature : **HKLM\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\LatestVirusDefsDate <= 7 days**
- Vérifier l’état de l’analyse complète : **HKLM:\Software\Symantec\Symantec Endpoint Protection\CurrentVersion\public-opstate\LastSuccessfulScanDateTime <= 7 days**
- Rechercher le numéro de version de signature pour Symantec 12 : **Registry Paths+ "CurrentVersion\SharedDefs" -Value "SRTSP"** 
- Chemin d’accès à la version de signature pour Symantec 14 : **Registry Paths+ "CurrentVersion\SharedDefs\SDSDefs" -Value "SRTSP"**

Chemins d’accès au Registre :
- **"HKLM:\Software\Symantec\Symantec Endpoint Protection" + $Path;**
- **"HKLM:\Software\Wow6432Node\Symantec\Symantec Endpoint Protection" + $Path**

## <a name="mcafee-endpoint-protection-for-windows"></a>Protection de point de terminaison McAfee pour Windows

Security Center formule la recommandation **Endpoint Protection doit être installé sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :

- **HKLM:\SOFTWARE\McAfee\Endpoint\AV\ProductVersion** exists
- **HKLM:\SOFTWARE\McAfee\AVSolution\MCSHIELDGLOBAL\GLOBAL\enableoas = 1**

Security Center formule la recommandation **Les problèmes d’intégrité Endpoint Protection doivent être résolus sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :

- Version McAfee : **HKLM:\SOFTWARE\McAfee\Endpoint\AV\ProductVersion >= 10**
- Rechercher la version de signature : **HKLM:\Software\McAfee\AVSolution\DS\DS -Value "dwContentMajorVersion"**
- Rechercher la date signature : **HKLM:\Software\McAfee\AVSolution\DS\DS -Value "szContentCreationDate" >= 7 days**
- Rechercher la date d'analyser : **HKLM:\Software\McAfee\Endpoint\AV\ODS -Value "LastFullScanOdsRunTime" >= 7 days**

## <a name="mcafee-endpoint-security-for-linux-threat-prevention"></a>McAfee Endpoint Security for Linux Threat Prevention 

Security Center formule la recommandation **Endpoint Protection doit être installé sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :

- Le fichier **/opt/isec/ens/threatprevention/bin/isecav** existe
- La sortie **"/opt/isec/ens/threatprevention/bin/isecav --version"** est : **McAfee name = McAfee Endpoint Security for Linux Threat Prevention and McAfee version >= 10**

Security Center formule la recommandation **Les problèmes d’intégrité Endpoint Protection doivent être résolus sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :

- **"/opt/isec/ens/threatprevention/bin/isecav --listtask"** retourne **Quick scan, Full scan** et les deux analyses <= 7 jours
- **"/opt/isec/ens/threatprevention/bin/isecav --listtask"** retourne **DAT and engine Update time** et les deux <= 7 jours
- **"/opt/isec/ens/threatprevention/bin/isecav --getoasconfig --summary"** retourne l’état **On Access Scan**

## <a name="sophos-antivirus-for-linux"></a>Sophos Anti-Virus pour Linux 

Security Center formule la recommandation **Endpoint Protection doit être installé sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :
- Le fichier **/opt/sophos-av/bin/savdstatus** se ferme ou recherche l'emplacement personnalisé **"readlink $(which savscan)"**
- **"/opt/sophos-av/bin/savdstatus --version"** retourne **Sophos name = Sophos Anti-Virus and Sophos version >= 9**

Security Center formule la recommandation **Les problèmes d’intégrité Endpoint Protection doivent être résolus sur vos machines** quand l’une des vérifications suivantes n’est pas satisfaite :
- **"/opt/sophos-av/bin/savlog --maxage=7 | grep -i "Scheduled scan .\* completed" | tail -1"** , retourne une valeur
- **"/opt/sophos-av/bin/savlog --maxage=7 | grep "scan finished"** | tail -1", retourne une valeur
- **"/opt/sophos-av/bin/savdstatus --lastupdate"** retourne lastUpdate qui doit être <= 7 jours 
- **"/opt/sophos-av/bin/savdstatus -v"** est égal à **"On-access scanning is running"** 
- **"/opt/sophos-av/bin/savconfig get LiveProtection"** retourne activé

## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Les journaux d’activité de l’extension Microsoft Antimalware sont disponibles à l’emplacement suivant : **%Systemdrive%\WindowsAzure\Logs\Plugins\Microsoft.Azure.Security.IaaSAntimalware(Ou PaaSAntimalware)\1.5.5.x(version#)\CommandExecution.log**

### <a name="support"></a>Support

Pour obtenir une aide supplémentaire, contactez les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).