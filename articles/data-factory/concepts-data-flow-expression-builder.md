---
title: Générateur d’expressions dans un flux de données de mappage
titleSuffix: Azure Data Factory & Azure Synapse
description: Générer des expressions à l’aide du Générateur d’expressions dans des flux de données de mappage dans Azure Data Factory et Azure Synapse Analytics
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.subservice: data-flows
ms.custom: synapse
ms.topic: conceptual
ms.date: 08/24/2021
ms.openlocfilehash: 7dd40b52cbc74e62a6dbb8ed83d19c968e48d9c4
ms.sourcegitcommit: d11ff5114d1ff43cc3e763b8f8e189eb0bb411f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/25/2021
ms.locfileid: "122822813"
---
# <a name="build-expressions-in-mapping-data-flow"></a>Générer des expressions dans un flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Dans un flux de données de mappage, de nombreuses propriétés de transformation sont entrées en tant qu’expressions. Ces expressions sont composées de valeurs de colonne, de paramètres, de fonctions, d’opérateurs et de littéraux qui correspondent à un type de données Spark au moment de l’exécution. Le mappage des flux de données a une expérience dédiée visant à vous aider à créer ces expressions, appelée **Générateur d’expressions**. Utilisant [IntelliSense](/visualstudio/ide/using-intellisense) pour la mise en surbrillance, la vérification de la syntaxe et la saisie semi-automatique, le générateur d’expressions est conçu pour faciliter la création de flux de données. Cet article explique comment utiliser le générateur d’expressions pour créer efficacement votre logique métier.

:::image type="content" source="media/data-flow/expresion-builder.png" alt-text="Générateur d’expressions":::

## <a name="open-expression-builder"></a>Générateur d’expressions ouvert

Il existe plusieurs points d’entrée pour ouvrir le générateur d’expressions. Ils dépendent tous du contexte spécifique de la transformation du flux de données. Le cas d’utilisation le plus courant est celui des transformations telles que [colonne dérivée](data-flow-derived-column.md) et [agrégation](data-flow-aggregate.md), où les utilisateurs créent ou mettent à jour des colonnes à l’aide du langage d’expression de flux de données. Vous pouvez ouvrir le générateur d’expressions en sélectionnant **Ouvrir le générateur d’expressions** au-dessus de la liste de colonnes. Vous pouvez également cliquer sur un contexte de colonne et ouvrir le générateur d’expressions directement avec cette expression.

![Dérivation du générateur d’expressions ouvert](media/data-flow/open-expression-builder-derive.png "Dérivation du générateur d’expressions ouvert")

Dans certaines transformations comme les [filtres](data-flow-filter.md), un clic sur une zone de texte d’expression bleue ouvre le générateur d’expressions. 

![Zone d’expression bleue](media/data-flow/expressionbox.png "Zone d’expression bleue")

Lorsque vous référencez des colonnes dans une correspondance ou un groupe par condition, une expression peut extraire des valeurs des colonnes. Pour créer une expression, sélectionnez **Colonne calculée**.

![Option de colonne calculée](media/data-flow/computedcolumn.png "Option de colonne calculée")

Pour les cas où une expression ou une valeur littérale sont des entrées valides, sélectionnez **Ajout de contenu dynamique** pour générer une expression qui renvoie une valeur littérale.

![Option d’ajout de contenu dynamique](media/data-flow/add-dynamic-content.png "Option d’ajout de contenu dynamique")

## <a name="expression-elements"></a>Éléments d’expression

Dans le mappage des flux de données, les expressions peuvent être composées de valeurs de colonne, de paramètres, de fonctions, de variables locales, d’opérateurs et de littéraux. Ces expressions doivent correspondre à un type de données Spark, par exemple une chaîne, une valeur booléenne ou un entier.

![Éléments d’expression](media/data-flow/expression-elements.png "Éléments d’expression")

### <a name="functions"></a>Fonctions

Les flux de données de mappage comportent des fonctions et opérateurs intégrés qui peuvent être utilisés dans des expressions. Pour obtenir la liste des fonctions disponibles, consultez la [référence sur le langage de flux de données de mappage](data-flow-expression-functions.md).

#### <a name="address-array-indexes"></a>Adresser des index de tableau

Lorsque vous traitez des colonnes ou des fonctions qui retournent des types tableau, utilisez des crochets ([]) pour accéder à un élément spécifique. Si l’index n’existe pas, l’expression prend la valeur NULL.

![Générateur d’expressions - tableau](media/data-flow/expression-array.png "Aperçu des données d’expression")

> [!IMPORTANT]
> Dans le mappage des flux de données, les tableaux sont basés sur un, ce qui signifie que le premier élément est référencé par l’index 1. Par exemple, myArray[1] accède au premier élément d’un tableau appelé « myArray ».

### <a name="input-schema"></a>Schéma d’entrée

Si votre flux de données utilise un schéma défini dans l’une de ses sources, vous pouvez référencer une colonne par nom dans de nombreuses expressions. Si vous utilisez la dérive de schéma, vous pouvez faire référence à des colonnes de manière explicite à l’aide des fonctions `byName()` ou `byNames()` ou à l’aide de modèles de colonne.

#### <a name="column-names-with-special-characters"></a>Noms de colonne avec des caractères spéciaux

Quand vous avez des noms de colonnes qui comportent des espaces ou des caractères spéciaux, placez-les entre accolades pour les référencer dans une expression.

```{[dbo].this_is my complex name$$$}```

### <a name="parameters"></a>Paramètres

Les paramètres sont des valeurs qui sont passées dans un flux de données au moment de l’exécution à partir d’un pipeline. Pour faire référence à un paramètre, cliquez sur le paramètre à partir de la vue **Éléments d’expression** ou référencez-le avec un signe dollar devant son nom. Par exemple, un paramètre appelé parameter1 est référencé par `$parameter1`. Pour plus d’informations, consultez [Paramétrage de flux de données de mappage](parameters-data-flow.md). 

### <a name="cached-lookup"></a>Recherche mise en cache

Une recherche mise en cache vous permet d’effectuer une recherche instantanée de la sortie d’un récepteur mis en cache. Deux fonctions peuvent être utilisées sur chaque récepteur, `lookup()` et `outputs()`. La syntaxe permettant de référencer ces fonctions est `cacheSinkName#functionName()`. Pour plus d’informations, consultez les [récepteurs de cache](data-flow-sink.md#cache-sink).

`lookup()` prend les colonnes correspondantes dans la transformation actuelle en tant que paramètres, et retourne une colonne complexe égale à la ligne qui correspond aux colonnes clés du récepteur de cache. La colonne complexe retournée contient une sous-colonne pour chaque colonne mappée dans le récepteur de cache. Par exemple, si vous aviez un récepteur de cache de code d’erreur `errorCodeCache` qui avait une colonne clé correspondant au code, et une colonne appelée `Message`. L’appel à `errorCodeCache#lookup(errorCode).Message` retournerait le message correspondant au code passé. 

`outputs()` n’accepte aucun paramètre et retourne l’intégralité du récepteur de cache sous la forme d’un tableau de colonnes complexes. Cet appel ne peut pas être effectué si des colonnes clés sont spécifiées dans le récepteur et que leur utilisation n’est possible qu’en présence d’un petit nombre de lignes dans le récepteur de cache. Un cas d’usage courant consiste à ajouter la valeur maximale d’une clé d’incrémentation. Si une ligne unique agrégée en cache `CacheMaxKey` contient une colonne `MaxKey`, vous pouvez référencer la première valeur en appelant `CacheMaxKey#outputs()[1].MaxKey`.

![Recherche mise en cache](media/data-flow/cached-lookup-example.png "Recherche mise en cache")

### <a name="locals"></a>Locals

Si vous partagez la logique entre plusieurs colonnes ou si vous souhaitez compartimenter votre logique, vous pouvez créer une locale dans une colonne dérivée\. Pour faire référence à une locale, cliquez sur le local à partir de la vue **Éléments d’expression** ou référencez-le avec un signe deux-points devant son nom. Par exemple, une variable locale appelée local1 est référencée par `:local1`. Pour plus d’informations sur les variables locales, consultez la [documentation sur les colonnes dérivées](data-flow-derived-column.md#locals).

## <a name="preview-expression-results"></a>Aperçu des résultats d’expressions

Si le [mode débogage](concepts-data-flow-debug-mode.md) est activé, vous pouvez utiliser le cluster de débogage de manière interactive pour afficher un aperçu de ce sur quoi votre expression est évaluée. Sélectionnez **Actualiser** en regard de l’aperçu des données pour mettre à jour les résultats de l’aperçu des données. Vous pouvez voir la sortie de chaque ligne en fonction des colonnes d’entrée.

![Aperçu en cours](media/data-flow/preview-expression.png "Aperçu des données d’expression")

## <a name="string-interpolation"></a>Interpolation de chaîne

Lorsque vous créez des chaînes longues qui utilisent des éléments d’expression, utilisez l’interpolation de chaîne pour créer facilement une logique de chaîne complexe. L’interpolation de chaîne évite une utilisation intensive de la concaténation de chaînes lorsque des paramètres sont inclus dans les chaînes de requête. Utilisez des guillemets doubles pour entourer le texte de chaîne littérale avec les expressions. Vous pouvez inclure des fonctions, des colonnes et des paramètres d’expression. Pour utiliser la syntaxe d’expression, mettez-la entre accolades.

Voici quelques exemples d’interpolation de chaîne :

* ```"My favorite movie is {iif(instr(title,', The')>0,"The {split(title,', The')[1]}",title)}"```

* ```"select * from {$tablename} where orderyear > {$year}"```

* ```"Total cost with sales tax is {round(totalcost * 1.08,2)}"```

* ```"{:playerName} is a {:playerRating} player"```

> [!NOTE]
> Lors de l’utilisation de la syntaxe d’interpolation de chaîne dans les requêtes sources SQL, la chaîne de requête doit être sur une seule ligne, sans « /n ».

## <a name="commenting-expressions"></a>Commenter les expressions

Ajoutez des commentaires à vos expressions en utilisant une seule ligne et une syntaxe de commentaire multiligne.

Les exemples suivants sont des commentaires valides :

* ```/* This is my comment */```

* ```/* This is a```
* ```multi-line comment */```

Si vous placez un commentaire en haut de votre expression, il apparaît dans la zone de texte de transformation afin de documenter vos expressions de transformation.

![Commentaire dans la zone de texte de transformation](media/data-flow/comment-expression.png "Commentaires")

## <a name="regular-expressions"></a>Expressions régulières

De nombreuses fonctions du langage d’expression utilisent la syntaxe des expressions régulières. Quand vous utilisez des fonctions d’expression régulière, le Générateur d’expressions tente d’interpréter une barre oblique inverse (\\) comme une séquence de caractères d’échappement. Quand vous utilisez des barres obliques inverses dans une expression régulière, placez l’expression régulière entière entre des barres obliques (\`) ou utilisez une barre oblique inverse double.

Voici un exemple utilisant des barres obliques :

```
regex_replace('100 and 200', `(\d+)`, 'digits')
```

Voici un exemple utilisant des barres obliques doubles :

```
regex_replace('100 and 200', '(\\d+)', 'digits')
```

## <a name="keyboard-shortcuts"></a>Raccourcis clavier

Vous trouverez ci-dessous une liste des raccourcis disponibles dans le générateur d’expressions. La plupart des raccourcis IntelliSense sont disponibles lors de la création d’expressions.

* Ctrl+K, Ctrl+C : marquer un commentaire sur la ligne entière.
* Ctrl+K, Ctrl+U : supprimer les marques de commentaire.
* F1 : fournir à l’éditeur des commandes d’aide.
* Alt + touche flèche bas : descendre d’une ligne par rapport à la ligne actuelle.
* Alt + touche flèche haut : monter d’une ligne par rapport à la ligne actuelle.
* Ctrl + Espace : afficher l’aide contextuelle.

## <a name="commonly-used-expressions"></a>Expressions couramment utilisées

### <a name="convert-to-dates-or-timestamps"></a>Convertir en dates ou en timestamps

Pour inclure des littéraux de chaîne dans votre sortie d’horodatage, wrappez votre conversion dans ```toString()```.

```toString(toTimestamp('12/31/2016T00:12:00', 'MM/dd/yyyy\'T\'HH:mm:ss'), 'MM/dd /yyyy\'T\'HH:mm:ss')```

Pour convertir des millisecondes issues de l’époque en date ou en horodatage, utilisez `toTimestamp(<number of milliseconds>)`. Si le temps s’écoule en secondes, multipliez par 1 000.

```toTimestamp(1574127407*1000l)```

Le « l » situé à la fin de l’expression précédente signifie la conversion en type long comme la syntaxe en ligne.

### <a name="find-time-from-epoch-or-unix-time"></a>Connaître l’heure à l’aide d’Epoch ou de l’heure Unix

toLong( currentTimestamp() - toTimestamp('1970-01-01 00:00:00.000', 'yyyy-MM-dd HH:mm:ss.SSS') ) * 1000l

### <a name="data-flow-time-evaluation"></a>Évaluation de la durée du flux de données

Le flux de données est traité jusqu’aux millisecondes. Pour *2018-07-31T20:00:00.2170000*, la sortie est *2018-07-31T20:00:00.217*.
Sur le portail du service, l’horodateur est affiché dans le **paramètre de navigateur actuel**. La partie des millisecondes (217) n’y figure pas toujours, mais elle est traitée lorsque le workflow est exécuté de bout en bout. Vous pouvez utiliser toString(myDateTimeColumn) comme expression pour prévisualiser les données avec une précision complète. Traitez DateHeure comme DateHeure plutôt que comme chaîne à toutes fins pratiques.
 

## <a name="next-steps"></a>Étapes suivantes

[Commencer à créer des expressions de transformation de données](data-flow-expression-functions.md)
