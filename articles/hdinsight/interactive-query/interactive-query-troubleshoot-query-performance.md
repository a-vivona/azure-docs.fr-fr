---
title: Problèmes de performances dans les requêtes LLAP Apache Hive dans Azure HDInsight
description: Les requêtes d’un LLAP Apache Hive s’exécutent plus lentement que prévu dans Azure HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/30/2019
ms.openlocfilehash: e21b49d26e8b7570b7f70e30f04b44fd23341f03
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2021
ms.locfileid: "112290608"
---
# <a name="scenario-poor-performance-in-apache-hive-llap-queries-in-azure-hdinsight"></a>Scénario : Problèmes de performances dans les requêtes LLAP Apache Hive dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Interactive Query dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Les configurations de cluster par défaut ne sont pas suffisamment adaptées à votre charge de travail. Dans Hive LLAP, les requêtes s’exécutent plus lentement que prévu.

## <a name="cause"></a>Cause

Plusieurs raisons peuvent être à l’origine de ce problème.

## <a name="resolution"></a>Résolution

LLAP est optimisé pour les requêtes impliquant des jointures et des agrégats. Des requêtes telles que les suivantes ne fonctionnent pas correctement dans un cluster Hive interactif :

```
select * from table where column = "columnvalue"
```

Pour améliorer les performances des requêtes de pointage dans Hive LLAP, définissez les configurations suivantes :

```
hive.llap.io.enabled=false; (disable LLAP IO)
hive.optimize.index.filter=false; (disable ORC row index)
hive.exec.orc.split.strategy=BI; (to avoid recombining splits)
```

Vous pouvez également augmenter l’utilisation du cache LLAP pour améliorer les performances moyennant le changement de configuration suivant :

```
hive.fetch.task.conversion=none
```

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../includes/hdinsight-troubleshooting-next-steps.md)]