---
title: Transformation de fenêtre dans le flux de données de mappage
titleSuffix: Azure Data Factory & Azure Synapse
description: Azure Data Factory correspondant à la transformation de fenêtres de flux de données
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.subservice: data-flows
ms.topic: conceptual
ms.custom: synapse
ms.date: 11/16/2020
ms.openlocfilehash: 8331d3908e484f1b82a4aa73cd6d498dac2a0ad3
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122641360"
---
# <a name="window-transformation-in-mapping-data-flow"></a>Transformation de fenêtre dans le flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

C’est lors de la transformation de fenêtres que vous allez définir des agrégations de colonnes utilisant une fenêtre dans vos flux de données. Dans le Générateur d’expressions, vous pouvez définir différents types d’agrégations basées sur les données ou sur des fenêtres de temps (clause SQL OVER), telles que LEAD, LAG, NTILE, CUMEDIST, RANK, etc.). Un nouveau champ qui inclut ces agrégations sera créé dans votre sortie. Vous pouvez également inclure des champs de regroupement facultatifs.

![Capture d’écran montrant l’option de fenêtrage sélectionnée dans le menu.](media/data-flow/windows1.png "fenêtres 1")

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4IAVu]

## <a name="over"></a>Over
Définir le partitionnement des données de colonne pour la transformation de votre fenêtre. L’équivalent SQL est le ```Partition By``` dans la clause Over de SQL. Si vous souhaitez créer un calcul ou une expression à utiliser pour le partitionnement, vous pouvez le faire en plaçant le curseur sur le nom de colonne et en sélectionnant « colonne calculée ».

![Capture d’écran affichant la section des paramètres de fenêtrage avec l’onglet Over sélectionné.](media/data-flow/windows4.png "fenêtres 4")

## <a name="sort"></a>Trier
Une autre partie de la clause Over consiste à définir le ```Order By```. Ceci définit l’ordre de tri de données. Vous pouvez également créer une expression pour une valeur de calcul dans ce champ de colonne pour le tri.

![Capture d’écran affichant la section des paramètres de fenêtrage avec l’onglet Tri sélectionné.](media/data-flow/windows5.png "fenêtres 5")

## <a name="range-by"></a>Plage par
Ensuite, définir le cadre de fenêtre comme Non limité ou Limité. Pour définir un cadre de fenêtre non limité, positionnez le curseur sur Non limité aux deux extrémités. Si vous choisissez un paramètre entre Non limité et Ligne actuelle, vous devez définir les valeurs de décalage de début et de fin. Les deux valeurs seront des entiers positifs. Vous pouvez utiliser des nombres ou des valeurs relatifs à partir de vos données.

Le curseur de la fenêtre a deux valeurs à définir : les valeurs avant la ligne actuelle et les valeurs après la ligne actuelle. Le décalage de début et de fin correspond aux deux sélecteurs sur le curseur.

![Capture d’écran affichant la section des paramètres de fenêtrage avec l’onglet Plage par sélectionné.](media/data-flow/windows6.png "fenêtres 6")

## <a name="window-columns"></a>Colonnes de fenêtres
Enfin, utilisez le Générateur d’expressions pour définir les agrégations que vous souhaitez utiliser avec les fenêtres de données, telles que RANK, COUNT, MIN, MAX, DENSE RANK, LEAD, LAG, etc.

![Capture d’écran présentant le résultat de l’action de fenêtrage.](media/data-flow/windows7.png "fenêtres 7")

La liste complète d’agrégation et de fonctions analytiques disponibles que vous pouvez utiliser dans le langage d’expression de flux de données ADF via le Générateur d’expressions est répertoriée ici : https://aka.ms/dataflowexpressions.

## <a name="next-steps"></a>Étapes suivantes

Si vous recherchez une agrégation simple groupe par groupe, utilisez la [transformation Agrégat](data-flow-aggregate.md)
