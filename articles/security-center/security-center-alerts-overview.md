---
title: Alertes et incidents de sécurité dans Azure Security Center
description: Découvrez comment Azure Security Center génère des alertes de sécurité et les met en corrélation avec des incidents.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 04/07/2021
ms.author: memildin
ms.openlocfilehash: f205f66adc2c875e271a85efc2e48763c260dfa0
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/16/2021
ms.locfileid: "112237686"
---
# <a name="security-alerts-and-incidents-in-azure-security-center"></a>Alertes et incidents de sécurité dans Azure Security Center

Security Center génère des alertes pour les ressources déployées sur vos environnements Azure, locaux et cloud hybrides.

Les alertes de sécurité sont déclenchées par des détections avancées et sont disponibles uniquement avec Azure Defender. Vous pouvez procéder à une mise à niveau à partir de la page **Tarification et paramètres**, comme décrit dans [Démarrage rapide : Activer Azure Defender](enable-azure-defender.md). Une version d’évaluation gratuite de 30 jours est disponible. Pour plus d’informations sur les prix dans la devise de votre choix et en fonction de votre région, consultez la page [Tarification de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="what-are-security-alerts-and-security-incidents"></a>Définition des alertes de sécurité et des incidents de sécurité 

Les **alertes** sont les notifications générées par Security Center lorsqu’il détecte des menaces pesant sur des ressources. Security Center hiérarchise et répertorie les alertes ainsi que les informations vous permettant d’analyser rapidement le problème. Security Center fournit également des instructions détaillées pour vous aider à parer les attaques. Les données d’alerte sont conservées pendant 90 jours.

Un **incident de sécurité** est une collection d’alertes apparentées, par opposition à une présentation individuelle des alertes. Security Center utilise la [corrélation des alertes intelligentes cloud](#cloud-smart-alert-correlation-in-azure-security-center-incidents) corréler divers signaux de faible fidélité et alertes dans des incidents de sécurité.

À l’aide d’incidents, Security Center vous offre une vue unique d’une campagne d’attaque et de toutes les alertes associées. Cette vue vous permet de comprendre rapidement quelles actions l’attaquant a effectuées et quelles ressources ont été impactées.

## <a name="respond-to-todays-threats"></a>Répondre aux menaces actuelles 

Les menaces ont fortement évolué au cours des 20 dernières années. Auparavant, les entreprises avaient uniquement à se soucier de la dégradation des sites web par des attaquants isolés qui cherchaient principalement à découvrir « jusqu’où ils pouvaient aller ». Les pirates d’aujourd’hui sont beaucoup plus évolués et organisés. Ils ont souvent des objectifs financiers et stratégiques spécifiques. Ils ont également davantage de ressources disponibles, car ils peuvent être financés par des États-nations ou encore par le crime organisé.

Ces évolutions ont conduit à un niveau inédit de professionnalisme dans les rangs des attaquants. Ces derniers ne s’intéressent plus à la dégradation de site web, mais plutôt au vol d’informations, aux comptes financiers et aux données privées qu’ils peuvent utiliser pour générer des liquidités sur le marché ouvert ou pour exploiter une activité commerciale particulière, ou encore une position politique ou militaire. Les pirates qui violent les réseaux afin de nuire à l’infrastructure et aux personnes constituent une menace bien plus importante que ceux avec des objectifs financiers.

Face au problème, les organisations déploient souvent plusieurs solutions, axées sur la défense du périmètre de l’entreprise ou de points de terminaison, en recherchant des signatures d’attaques connues. Ces solutions ont tendance à générer un volume élevé d’alertes basse fidélité, qui nécessitent qu’un analyste les trie et les examine. La plupart des organisations ne disposent pas du temps et de l’expertise nécessaires pour répondre à ces alertes, qui restent alors bien souvent sans réponse.  

En outre, les attaquants ont fait évoluer leurs méthodes pour compromettre les nombreuses défenses basées sur la signature et pour [s’adapter aux environnements de cloud](https://azure.microsoft.com/blog/detecting-threats-with-azure-security-center/). De nouvelles approches sont nécessaires pour identifier plus rapidement les nouvelles menaces, et accélérer la détection et la réaction.


## <a name="continuous-monitoring-and-assessments"></a>Surveillance et évaluations continues

Azure Security Center s’appuie sur des équipes de recherche de sécurité et de sciences des données au sein de Microsoft qui surveillent en continu les évolutions en matière de menaces. Cela inclut les initiatives suivantes :

* **Analyse des informations sur les menaces** : les informations sur les menaces incluent des mécanismes, des indicateurs, des implications et des conseils pratiques sur les menaces, nouvelles ou existantes. Ces informations sont partagées avec la communauté dédiée à la sécurité, et Microsoft surveille en permanence le flux des informations sur les menaces provenant de sources internes et externes.
* **Partage de signal** : les informations fournies par les équipes de sécurité sur le portefeuille complet de services cloud et locaux, de serveurs et d’appareils de point de terminaison client de Microsoft sont partagées et analysées.
* **Spécialistes de la sécurité Microsoft** : engagement continu avec les équipes Microsoft travaillant dans des domaines de la sécurité spécialisés, tels que la forensique et la détection d’attaques web.
* **Réglage de la détection** : des algorithmes sont exécutés sur les jeux de données client réels, et les chercheurs en sécurité collaborent avec les clients pour valider les résultats. Les vrais et les faux positifs sont utilisés pour affiner les algorithmes d’apprentissage automatique.

Ces efforts combinés aboutissent à de nouvelles détections et à des améliorations, dont vous pouvez bénéficier instantanément sans aucune action de votre part.

## <a name="how-does-security-center-detect-threats"></a>Comment Security Center détecte-t-il les menaces ? <a name="detect-threats"> </a>

Les chercheurs en sécurité de Microsoft sont constamment à l’affût des nouvelles menaces. Grâce à notre présence mondiale sur le cloud et localement, nous avons accès à un vaste jeu de télémétrie. Cette collection diverse et étendue de jeux de données nous permet de détecter de nouveaux modèles et de nouvelles tendances d’attaques dans nos produits locaux destinés au consommateur et aux entreprises, ainsi que dans nos services en ligne. Par conséquent, Azure Security Center peut rapidement mettre à jour ses algorithmes de détection, puisque les pirates sont à l’origine d’attaques innovantes de plus en plus sophistiquées. Cette approche permet de faire face à des menaces en pleine mutation.

Pour détecter les menaces réelles et réduire le nombre de faux positifs, Security Center collecte, analyse et intègre automatiquement les données de journaux provenant de vos ressources Azure et du réseau. Cela fonctionne également avec les solutions de partenaires connectées, comme les solutions de pare-feu et de protection de point de terminaison. Security Center analyse ces informations, souvent issues de plusieurs sources, pour identifier les menaces.

![Collecte et présentation des données dans Azure Security Center.](./media/security-center-alerts-overview/security-center-detection-capabilities.png)

Azure Security Center emploie des analyses de sécurité avancées allant bien au-delà des approches simplement basées sur la signature. Les innovations en matière de Big Data et de technologies [Machine Learning](https://azure.microsoft.com/blog/machine-learning-in-azure-security-center/) sont mises à profit pour évaluer des événements dans toute la structure du cloud, et permettent ainsi de détecter des menaces qui seraient impossibles à identifier à l’aide de méthodes manuelles et de prédire l’évolution des attaques. Ces analyses de sécurité comprennent les éléments suivants :

- **Threat Intelligence intégrée** : Microsoft dispose d’une multitude d’informations en matière de menaces à l’échelle mondiale. La télémétrie provient de plusieurs sources, telles qu’Azure, Microsoft 365, Microsoft CRM Online, Microsoft Dynamics AX, outlook.com, MSN.com, Microsoft Digital Crimes Unit (DCU) et Microsoft Security Response Center (MSRC). Les chercheurs reçoivent aussi les informations sur les menaces partagées par les principaux fournisseurs de services cloud et les flux en provenance d’autres tiers. Azure Security Center peut utiliser ces informations pour vous alerter en cas de menaces provenant d’éléments malveillants connus.

- **Analytique comportementale** : L’analyse comportementale est une technique qui analyse et compare les données à une collection de modèles connus. Toutefois, ces modèles ne sont pas de simples signatures. Ils sont déterminés par le biais d’algorithmes d’apprentissage automatique appliqués aux ensembles de données massifs. Ils sont également déterminés à travers une analyse minutieuse des comportements malveillants par des experts. Azure Security Center peut utiliser l’analyse comportementale pour identifier les ressources compromises en se basant sur l’analyse des journaux d’activité de la machine virtuelle, des journaux d’activité du périphérique réseau virtuel, des journaux d’activité Service Fabric et d’autres sources.

- **Détection des anomalies** : Azure Security Center utilise également la détection des anomalies pour identifier les menaces. Contrairement à l’analyse comportementale (qui dépend de modèles connus dérivés de grands jeux de données), la détection d’anomalie est « personnalisée » et se concentre sur les lignes de base propres à vos déploiements. L’apprentissage automatique est appliqué pour déterminer l’activité normale de vos déploiements et les règles sont générées pour définir les conditions de valeurs aberrantes pouvant signaler un événement de sécurité.

## <a name="how-are-alerts-classified"></a>Comment les alertes sont-elles classifiées ?

Security Center affecte un degré de gravité aux alertes pour vous aider à hiérarchiser l’ordre dans lequel vous remédiez à chaque alerte. Ainsi, quand une ressource est compromise, vous pouvez vous en occuper immédiatement. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

> [!NOTE]
> La gravité des alertes s’affiche différemment dans le portail et les versions de l’API REST antérieures au 01-01-2019. Si vous utilisez une ancienne version de l’API, effectuez une mise à niveau pour bénéficier de l’expérience cohérente décrite ci-dessous.

| severity          | Réponse recommandée                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Importante**          | il est fort probable que votre ressource ait été compromise. Vous devez étudier le problème immédiatement. Security Center est très confiant quant à l’intention malveillante et aux constats à l’origine de l’alerte. Par exemple, une alerte qui détecte l’exécution d’un outil malveillant connu, tel que Mimikatz, un outil couramment utilisé pour dérober des informations d’identification.                                                                                                           |
| **Moyenne**        | il s’agit probablement d’une activité suspecte susceptible d’indiquer qu’une ressource est compromise. Le niveau de confiance de Security Center dans l’analyse et la recherche est moyen, tandis le niveau de confiance quant à l’intention malveillante est moyen ou élevé. Il s’agit généralement de détections basées sur des anomalies ou l’apprentissage automatique. Par exemple, une tentative de connexion depuis un emplacement anormal.                                                                                                            |
| **Faible**           | cela peut être un positif sans gravité ou une attaque bloquée. Security Center n’est pas suffisamment confiant sur le fait que l’intention soit malveillante. L’activité est peut-être innocente. Par exemple, l’effacement des journaux est une action qui peut se produire lorsqu’un pirate tente de masquer ses traces, mais, dans de nombreux cas, il s’agit d’une opération de routine effectuée par les administrateurs. Security Center n’indique généralement pas les attaques bloquées, sauf s’il s’agit d’un cas qu’il peut être intéressant d’examiner. |
| **Informational** | Un incident est généralement constitué de plusieurs alertes, dont certaines peuvent apparaître individuellement à titre d’information seulement, mais mériter un examen plus approfondi dans le contexte des autres alertes.                                                                                                                                                                                                                                                               |
|                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

## <a name="export-alerts"></a>Exporter les alertes

Vous disposez d’une gamme d’options pour l’affichage de vos alertes en dehors de Security Center, notamment :

- **Télécharger le rapport CSV** dans le tableau de bord des alertes fournit une exportation ponctuelle au format CSV.
- L’**exportation continue** dans Tarification et paramètres vous permet de configurer des flux d’alertes et de recommandations de sécurité vers les espaces de travail Log Analytics et Event Hubs. [En savoir plus sur l’exportation continue](continuous-export.md).
- Le **connecteur Azure Sentinel** diffuse des alertes de sécurité d’Azure Security Center vers Azure Sentinel. [En savoir plus sur la connexion d’Azure Security Center à Azure Sentinel](../sentinel/connect-azure-security-center.md).

En savoir plus sur toutes les options d’exportation dans [Diffuser des alertes vers un système SIEM, SOAR ou une solution de gestion des services informatiques](export-to-siem.md) et [Exporter en continu des données Security Center](continuous-export.md).

## <a name="cloud-smart-alert-correlation-in-azure-security-center-incidents"></a>Corrélation des alertes intelligentes cloud dans Azure Security Center (incidents)

Azure Security Center analyse en continu les charges de travail de cloud hybride en utilisant une analytique avancée et des informations de Threat Intelligence pour vous alerter sur les activités malveillantes.

La couverture contre les menaces s’étend progressivement. Face à la nécessité de détecter jusqu’à la moindre compromission, il peut s’avérer compliqué pour les analystes de sécurité de trier les différentes alertes et d’identifier une attaque réelle. Security Center aide les analystes à faire face à la charge de travail liée aux alertes. Il facilite le diagnostic des attaques en temps réel, en mettant en corrélation les différentes alertes et les signaux de faible fidélité en incidents de sécurité.

L’analytique de fusion constitue le back-end technologique et analytique sur lequel reposent les incidents Security Center, permettant la mise en corrélation des différentes alertes et signaux contextuels. La fusion examine les différents signaux relevés sur un abonnement parmi les ressources. La fusion recherche des modèles qui révèlent la progression des attaques ou les signaux avec des informations contextuelles partagées, indiquant que vous devez utiliser une procédure de réponse unifiée pour eux.

L’analytique de fusion combine les connaissances dans le domaine de la sécurité avec l’intelligence artificielle pour analyser les alertes, ce qui permet de découvrir les nouveaux modèles d’attaque à mesure qu’ils apparaissent. 

Security Center s’appuie sur MITRE Attack Matrix pour associer les alertes à leur intention perçue, facilitant la formalisation des connaissances dans le domaine de la sécurité. De plus, en utilisant les informations collectées à chaque étape d’une attaque, Security Center peut écarter les activités qui semblent être des étapes d’une attaque, mais qui ne le sont pas en réalité.

Étant donné que les attaques se produisent souvent sur différents locataires, Security Center peut combiner des algorithmes d’intelligence artificielle pour analyser les séquences d’attaque signalées dans chaque abonnement. Cette technique identifie les séquences d’alerte comme des modèles d’alerte prédominants au lieu de simplement les associer les unes aux autres incidemment.

Dans le cadre d’une investigation d’un incident, les analystes ont souvent besoin de plus de contexte pour aboutir à un verdict sur la nature de la menace et sur les moyens de l’atténuer. Par exemple, même si une anomalie de réseau a été détectée, sans connaître les autres événements qui se produisent sur le réseau ou en lien avec la ressource ciblée, il est difficile de déterminer les actions à prendre. Pour vous aider, un incident de sécurité peut inclure des artefacts, des événements connexes et des informations. Les informations supplémentaires disponibles pour les incidents de sécurité varient en fonction du type de menace détecté et de la configuration de votre environnement. 

> [!TIP]
> Pour obtenir la liste des alertes d’incident de sécurité qui peuvent être générées par l’analytique de fusion, consultez la [table de référence des alertes](alerts-reference.md#alerts-fusion).

:::image type="content" source="./media/security-center-alerts-cloud-smart/security-incident.png" alt-text="Capture d’écran du rapport d’incident de sécurité détecté.":::

Pour gérer vos incidents de sécurité, consultez le [Comment gérer les incidents de sécurité dans Azure Security Center](security-center-incident.md).

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a présenté les différents types d’alertes disponibles dans Azure Security Center. Pour plus d'informations, consultez les pages suivantes :

- [Alertes de sécurité dans le journal d’activité Azure](https://go.microsoft.com/fwlink/?linkid=2114113) : en plus d’être disponibles sur le portail Azure ou par programmation, les alertes et incidents de sécurité sont audités en tant qu’événements dans le Journal d’activité Azure.
- [Table de référence des alertes Azure Defender](alerts-reference.md)
- [Répondre à des alertes de sécurité](security-center-managing-and-responding-alerts.md#respond-to-security-alerts)