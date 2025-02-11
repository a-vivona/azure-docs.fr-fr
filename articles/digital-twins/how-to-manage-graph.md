---
title: Gérer le graphe de jumeaux et les relations
titleSuffix: Azure Digital Twins
description: Consultez la procédure de gestion d’un graphique de jumeaux numériques via la connexion de ceux-ci à des relations.
author: baanders
ms.author: baanders
ms.date: 11/03/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: e43617a2874a7a6817dc8126bca8e1af79436eb2
ms.sourcegitcommit: 63f3fc5791f9393f8f242e2fb4cce9faf78f4f07
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/26/2021
ms.locfileid: "114689965"
---
# <a name="manage-a-graph-of-digital-twins-using-relationships"></a>Gérer un graphique de jumeaux numériques à l’aide de relations

Azure Digital Twins consiste en un [graphique de jumeaux](concepts-twins-graph.md) représentant l’ensemble de votre environnement. Le graphe de jumeaux est constitué de jumeaux numériques individuels connectés via des **relations**. 

Une fois que vous disposez d’une [instance Azure Digital Twins](how-to-set-up-instance-portal.md) opérationnelle et que vous avez configuré un code d’[authentification](how-to-authenticate-client.md) dans votre application cliente, vous pouvez créer, modifier et supprimer des jumeaux numériques et leurs relations dans une instance Azure Digital Twins.

Cet article se concentre sur la gestion globale des relations et du graphe. Pour utiliser des jumeaux numériques individuels, consultez [Gérer des jumeaux numériques](how-to-manage-twin.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

[!INCLUDE [digital-twins-developer-interfaces.md](../../includes/digital-twins-developer-interfaces.md)]

[!INCLUDE [visualizing with Azure Digital Twins explorer](../../includes/digital-twins-visualization.md)]

:::image type="content" source="media/concepts-azure-digital-twins-explorer/azure-digital-twins-explorer-demo.png" alt-text="Capture d’écran d’Azure Digital Twins Explorer montrant des exemples de modèles et de jumeaux" lightbox="media/concepts-azure-digital-twins-explorer/azure-digital-twins-explorer-demo.png":::


## <a name="create-relationships"></a>Créer des relations

Les relations décrivent les interconnexions entre différents jumeaux numériques, ce qui constitue la base du graphique de jumeaux.

Les relations sont créées à l’aide de l’appel `CreateOrReplaceRelationshipAsync()`. 

Pour créer une relation, vous devez spécifier :
* L’ID de jumeau source (`srcId` dans l’exemple de code ci-dessous) : l’ID du jumeau à l’origine de la relation.
* L’ID de jumeau cible (`targetId` dans l’exemple de code ci-dessous) : l’ID du jumeau destinataire de la relation.
* Un nom de relation (`relName` dans l’exemple de code ci-dessous) : le type générique de la relation, tel que _contient_.
* Un ID de relation (`relId` dans l’exemple de code ci-dessous) : le nom spécifique de cette relation, tel que _Relation1_.

L’ID de relation doit être unique au sein du jumeau source donné. Il ne doit pas être globalement unique.
Par exemple, pour le jumeau Foo, chaque ID de relation spécifique doit être unique. Toutefois, un autre jumeau Bar peut avoir une relation sortante qui correspond au même ID que celui d’une relation Foo.

L’exemple de code suivant illustre la procédure de création d’une relation dans votre instance Azure Digital Twins. Il utilise l’appel du kit SDK (en surbrillance) dans une méthode personnalisée pouvant apparaître au sein d’un programme plus volumineux.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="CreateRelationshipMethod" highlight="13":::

Cette fonction personnalisée peut maintenant être appelée pour créer une relation _contains_ de cette manière : 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseCreateRelationship":::

Si vous souhaitez créer plusieurs relations, vous pouvez répéter des appels à la même méthode, en passant différents types de relations dans l’argument. 

Pour plus d’informations sur la classe d’assistance `BasicRelationship`, consultez [API et SDK Azure Digital Twins](concepts-apis-sdks.md#serialization-helpers).

### <a name="create-multiple-relationships-between-twins"></a>Créer plusieurs relations entre jumeaux

Les relations peuvent être classées comme suit : 

* Relations sortantes : relations appartenant à ce jumeau qui pointent vers l’extérieur pour le connecter à d’autres jumeaux. La méthode `GetRelationshipsAsync()` est utilisée pour obtenir les relations sortantes d’un jumeau.
* Relations entrantes : relations appartenant à d’autres jumeaux qui pointent vers ce jumeau pour créer un lien « entrant ». La méthode `GetIncomingRelationshipsAsync()` est utilisée pour obtenir les relations entrantes d’un jumeau.

Il n’existe aucune restriction du nombre de relations entre deux jumeaux : vous pouvez avoir autant de relations entre jumeaux que vous le souhaitez. 

Cela signifie que vous pouvez exprimer plusieurs types de relations entre deux jumeaux à la fois. Par exemple, le Jumeau A peut avoir une relation *stockée* et une relation *fabriquée* avec le Jumeau B.

Vous pouvez même créer plusieurs instances du même type de relation entre les deux mêmes jumeaux si vous le voulez. Dans cet exemple, le Jumeau A peut avoir deux relations *stockées* différentes avec le Jumeau B, à condition que les relations aient différents ID de relation.

## <a name="list-relationships"></a>Lister les relations

### <a name="list-properties-of-a-single-relationship"></a>Lister les propriétés d’une relation unique

Vous pouvez toujours désérialiser les données de relation vers le type de votre choix. Pour un accès de base à une relation, utilisez le type `BasicRelationship`. La classe d’assistance `BasicRelationship` vous donne également accès aux propriétés définies dans la relation, par le biais d’un `IDictionary<string, object>`. Pour lister les propriétés, vous pouvez utiliser ceci :

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_other.cs" id="ListRelationshipProperties":::

### <a name="list-outgoing-relationships-from-a-digital-twin"></a>Lister les relations sortantes d’un jumeau numérique

Pour accéder à la liste des relations **sortantes** pour un jumeau donné dans le graphe, vous pouvez utiliser la méthode `GetRelationships()` comme suit : 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="GetRelationshipsCall":::

Cette méthode retourne `Azure.Pageable<T>` ou `Azure.AsyncPageable<T>`, selon que vous utilisez la version synchrone ou asynchrone de l’appel.

Voici un exemple qui récupère une liste de relations. Il utilise l’appel du kit SDK (en surbrillance) dans une méthode personnalisée pouvant apparaître au sein d’un programme plus volumineux.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FindOutgoingRelationshipsMethod" highlight="8":::

Vous pouvez à présent appeler cette méthode personnalisée pour voir les relations sortantes des jumeaux comme ceci :

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFindOutgoingRelationships":::

Vous pouvez utiliser les relations récupérées pour accéder à d’autres jumeaux dans votre graphe en lisant le champ `target` de la relation retournée et en l’utilisant comme ID de votre prochain appel de `GetDigitalTwin()`.

### <a name="list-incoming-relationships-to-a-digital-twin"></a>Lister les relations entrantes vers un jumeau numérique

Azure Digital Twins fournit également un appel d’API qui permet de rechercher toutes les relations **entrantes** vers un jumeau donné. Ce SDK s’avère souvent utile pour la navigation inverse ou lors de la suppression d’un jumeau.

>[!NOTE]
> Les appels à `IncomingRelationship` ne retournent pas le corps complet de la relation. Pour plus d’informations sur la classe `IncomingRelationship`, consultez sa [documentation de référence](/dotnet/api/azure.digitaltwins.core.incomingrelationship?view=azure-dotnet&preserve-view=true).

L’exemple de code de la section précédente s’est concentré sur la recherche des relations sortantes d’un jumeau. L’exemple suivant est structuré de la même façon, mais il recherche des relations *entrantes*. Cet exemple utilise également l’appel du kit SDK (en surbrillance) dans une méthode personnalisée pouvant apparaître au sein d’un programme plus volumineux.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FindIncomingRelationshipsMethod" highlight="8":::

Vous pouvez à présent appeler cette méthode personnalisée pour voir les relations entrantes des jumeaux comme ceci :

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFindIncomingRelationships":::

### <a name="list-all-twin-properties-and-relationships"></a>Répertorier toutes les propriétés de jumeau et les relations

En utilisant les méthodes ci-dessus pour répertorier les relations sortantes et entrantes en lien avec un jumeau, vous pouvez créer une méthode qui imprime des informations complètes sur le jumeau, notamment les propriétés et les deux types de ses relations. Voici un exemple de méthode personnalisée qui montre comment combiner les méthodes personnalisées ci-dessus à cette fin.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="FetchAndPrintMethod":::

Vous pouvez à présent appeler cette fonction personnalisée comme ceci : 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseFetchAndPrint":::

## <a name="update-relationships"></a>Mettre à jour les relations

Les relations sont mises à jour à l’aide de la méthode `UpdateRelationship`. 

>[!NOTE]
>Cette méthode permet de mettre à jour les **propriétés** d’une relation. Si vous êtes amené à changer le jumeau source ou le jumeau cible de la relation, vous devez [supprimer la relation](#delete-relationships) et [en recréer une](#create-relationships) à l’aide des nouveaux jumeaux.

Les paramètres obligatoires pour l’appel client sont les suivants :
- L’ID du jumeau source (le jumeau à l’origine de la relation).
- L’ID de la relation à mettre à jour.
- Un document [JSON Patch](http://jsonpatch.com/) contenant les propriétés et les nouvelles valeurs à mettre à jour.

Voici un exemple de code qui montre comment utiliser cette méthode. Cet exemple utilise l’appel du kit SDK (en surbrillance) dans une méthode personnalisée pouvant apparaître au sein d’un programme plus volumineux.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UpdateRelationshipMethod" highlight="6":::

Voici un exemple d’appel de cette méthode personnalisée, qui passe un document JSON Patch avec les informations nécessaires à la mise à jour d’une propriété.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseUpdateRelationship":::

## <a name="delete-relationships"></a>Supprimer des relations

Le premier paramètre spécifie le jumeau source (le jumeau à l’origine de la relation). L’autre paramètre est l’ID de relation. Vous avez besoin de l’ID de jumeau et de l’ID de relation, car les ID de relation ne sont uniques que dans l’étendue d’un jumeau.

Voici un exemple de code qui montre comment utiliser cette méthode. Cet exemple utilise l’appel du kit SDK (en surbrillance) dans une méthode personnalisée pouvant apparaître au sein d’un programme plus volumineux.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="DeleteRelationshipMethod" highlight="5":::

Vous pouvez à présent appeler cette méthode personnalisée pour supprimer une relation comme ceci :

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs" id="UseDeleteRelationship":::

## <a name="create-graph-from-a-csv-file"></a>Créer un graphe à partir d’un fichier CSV

Dans des cas d’usage pratiques, les hiérarchies de jumeaux sont souvent créées à partir des données stockées dans une base de données différente ou peut-être dans une feuille de calcul ou un fichier CSV. Cette section explique comment lire des données à partir d’un fichier CSV et créer un graphe de jumeaux en dehors de celui-ci.

Examinez le tableau de données suivant, qui décrit un ensemble de jumeaux numériques et de relations.

|  ID de modèle    | ID de jumeau (doit être unique) | Nom de la relation  | ID de jumeau cible  | Données d’initialisation du jumeau |
| --- | --- | --- | --- | --- |
| dtmi:example:Floor;1    | Floor1 | contains | Room1 | |
| dtmi:example:Floor;1    | Floor0 | contains | Room0 | |
| dtmi:example:Room;1    | Room1 | | | {"Temperature": 80} |
| dtmi:example:Room;1    | Room0 | | | {"Temperature": 70} |

L’une des méthodes permettant d’obtenir ces données dans Azure Digital Twins consiste à convertir le tableau en fichier CSV. Une fois ce tableau converti, le code peut être écrit pour interpréter le fichier en commandes permettant de créer des jumeaux et des relations. L’exemple de code suivant illustre la lecture des données du fichier CSV et la création d’un graphe de jumeaux dans Azure Digital Twins.

Dans le code ci-dessous, le fichier CSV est appelé *data.csv* ; il contient un espace réservé représentant le **nom d’hôte** de votre instance Azure Digital Twins. L’exemple utilise aussi plusieurs packages que vous pouvez ajouter à votre projet pour faciliter ce processus.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graphFromCSV.cs":::

## <a name="runnable-twin-graph-sample"></a>Exemple de graphe de jumeaux exécutable

L’extrait de code exécutable suivant utilise les opérations de relation de cet article pour créer un graphe de jumeaux en dehors des relations et des jumeaux.

### <a name="set-up-sample-project-files"></a>Configurer des exemples de fichiers de projet

L’extrait de code utilise deux exemples de définitions de modèle, [Room.json](https://raw.githubusercontent.com/Azure-Samples/digital-twins-samples/master/AdtSampleApp/SampleClientApp/Models/Room.json) et [Floor.json](https://raw.githubusercontent.com/Azure-Samples/digital-twins-samples/master/AdtSampleApp/SampleClientApp/Models/Floor.json). Pour **télécharger les fichiers de modèle** et les utiliser dans votre code, utilisez les liens suivants afin d’accéder directement aux fichiers dans GitHub. Cliquez avec le bouton droit n’importe où sur l’écran, sélectionnez **Enregistrer sous** dans le menu contextuel du navigateur, puis utilisez la fenêtre Enregistrer sous pour enregistrer les fichiers sous les noms **Room.json** et **Floor.json**.

Créez ensuite un **projet d’application console** dans Visual Studio ou l’éditeur de votre choix.

Puis, **copiez le code suivant** de l’exemple exécutable dans votre projet :

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/graph_operations_sample.cs":::

### <a name="configure-project"></a>Configurer un projet

Ensuite, effectuez les étapes ci-après pour configurer votre code de projet :
1. Ajoutez les fichiers **Room.json** et **Floor.json** précédemment téléchargés à votre projet et remplacez les espaces réservés `<path-to>` dans le code pour indiquer à votre programme où les trouver.
1. Remplacez l’espace réservé `<your-instance-hostname>` par le nom d’hôte de votre instance Azure Digital Twins.
1. Ajoutez deux dépendances à votre projet. Elles seront nécessaires pour travailler avec Azure Digital Twins. La première correspond au package pour le [SDK Azure Digital Twins pour .NET](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true), et la seconde fournit des outils facilitant l’authentification auprès d’Azure.

      ```cmd/sh
      dotnet add package Azure.DigitalTwins.Core
      dotnet add package Azure.Identity
      ```

Vous devez également configurer des informations d’identification locales si vous souhaitez exécuter l’exemple directement. La section suivante décrit ce processus pas à pas.
[!INCLUDE [Azure Digital Twins: local credentials prereq (outer)](../../includes/digital-twins-local-credentials-outer.md)]

### <a name="run-the-sample"></a>Exécution de l'exemple

Vous avez terminé la configuration et pouvez à présent exécuter l’exemple de projet de code.

Voici la sortie de la console du programme : 

:::image type="content" source="./media/how-to-manage-graph/console-output-twin-graph.png" alt-text="Capture d’écran de la sortie de la console montrant les détails sur les jumeaux, avec les relations entrantes et sortantes des jumeaux" lightbox="./media/how-to-manage-graph/console-output-twin-graph.png":::

> [!TIP]
> Le graphe de jumeaux est un concept en lien avec la création de relations entre les jumeaux. Si vous souhaitez afficher la représentation visuelle du graphe de jumeaux, consultez la section [Visualisation](how-to-manage-graph.md#visualization) de cet article. 

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’interrogation d’un graphique de jumeaux Azure Digital Twins :
* [Langage de requête](concepts-query-language.md)
* [Interroger le graphe de jumeaux](how-to-query-graph.md)