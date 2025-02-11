---
title: Accéder à l’API Cassandra Azure Cosmos DB à partir d’Azure Databricks
description: Cet article explique comment utiliser l’API Cassandra Azure Cosmos DB à partir d’Azure Databricks.
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 89986ca27b67f4bd420121c3113324e6e7397219
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122525117"
---
# <a name="access-azure-cosmos-db-cassandra-api-data-from-azure-databricks"></a>Accéder aux données de l’API Cassandra Azure Cosmos DB à partir d’Azure Databricks
[!INCLUDE[appliesto-cassandra-api](../includes/appliesto-cassandra-api.md)]

Cet article explique comment utiliser l’API Cassandra Azure Cosmos DB à partir de Spark sur [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks).

## <a name="prerequisites"></a>Prérequis

* [Provisionner un compte de l’API Cassandra Azure Cosmos DB](manage-data-dotnet.md#create-a-database-account)

* [Passer en revue les principes fondamentaux de la connexion à l’API Cassandra Azure Cosmos DB](connect-spark-configuration.md)

* [Provisionner un cluster Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal)

* [Consulter les exemples de code illustrant l’utilisation de l’API Cassandra](connect-spark-configuration.md#next-steps)

* [Utiliser cqlsh pour la validation si vous le souhaitez](connect-spark-configuration.md#connecting-to-azure-cosmos-db-cassandra-api-from-spark)

* **Configuration de l’instance de l’API Cassandra pour le connecteur Cassandra :**

  Le connecteur pour l’API Cassandra nécessite que les informations de connexion Cassandra soient initialisées dans le cadre du contexte Spark. Quand vous lancez un bloc-notes Databricks, le contexte spark est déjà initialisé et il n’est pas recommandé de l’arrêter et de le réinitialiser. Une solution consiste à ajouter la configuration de l’instance de l’API Cassandra à un niveau de cluster, dans la configuration du cluster spark. Cette opération s’effectue une seule fois par cluster. Ajoutez à la configuration Spark le code suivant, où chaque paire clé-valeur utilise un espace en guise de séparation :
 
  ```scala
  spark.cassandra.connection.host YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com
  spark.cassandra.connection.port 10350
  spark.cassandra.connection.ssl.enabled true
  spark.cassandra.auth.username YOUR_COSMOSDB_ACCOUNT_NAME
  spark.cassandra.auth.password YOUR_COSMOSDB_KEY
  ```

## <a name="add-the-required-dependencies"></a>Ajouter les dépendances nécessaires

* **Connecteur Spark Cassandra :** Pour intégrer l’API Cassandra Azure Cosmos DB à Spark, vous devez attacher le connecteur Cassandra au cluster Azure Databricks. Pour attacher le cluster :

  * Passez en revue la version du runtime de Databricks et la version de Spark. Ensuite, recherchez les [coordonnées Maven](https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector) qui sont compatibles avec le connecteur Spark Cassandra, puis attachez-le au cluster. Consultez l’article [« Upload a Maven package or Spark package »](https://docs.databricks.com/user-guide/libraries.html) (Charger un package Maven ou un package Spark) pour savoir comment attacher la bibliothèque de connecteur au cluster. Nous vous recommandons de sélectionner le runtime Databricks version 7.5, qui prend en charge Spark 3.0. Pour ajouter le connecteur Apache Spark Cassandra à votre cluster, sélectionnez **Bibliothèques** > **Installer Nouveau** > **Maven**, puis ajoutez `com.datastax.spark:spark-cassandra-connector-assembly_2.12:3.0.0` dans les coordonnées Maven. Si vous utilisez Spark 2.x, nous vous recommandons d’utiliser un environnement avec Spark version 2.4.5, avec le connecteur Spark aux coordonnées Maven `com.datastax.spark:spark-cassandra-connector_2.11:2.4.3`.

* **Bibliothèque spécifique à l’API Cassandra Azure Cosmos DB :** si vous utilisez Spark 2.x, une fabrique de connexion personnalisée est nécessaire pour configurer la stratégie de nouvelles tentatives à partir du connecteur Spark sur l’API Cassandra Azure Cosmos DB. Ajoutez les [coordonnées maven](https://search.maven.org/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper/1.2.0/jar)`com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.2.0` pour attacher la bibliothèque au cluster.

> [!NOTE]
> Si vous utilisez Spark 3.0 ou une version ultérieure, vous n’avez pas besoin d’installer la bibliothèque spécifique à l’API Cassandra Cosmos DB mentionnée ci-dessus.

## <a name="sample-notebooks"></a>Exemples de notebooks

Une liste [d’exemples de blocs-notes](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/main/notebooks/scala) Azure Databricks est téléchargeable à partir du dépôt GitHub. Ces exemples montrent comment se connecter à l’API Cassandra Azure Cosmos DB à partir de Spark et effectuer différentes opérations CRUD sur les données. Vous pouvez également [importer tous les blocs-notes](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-databricks/tree/main/dbc) dans votre espace de travail du cluster Databricks, puis exécuter l’espace de travail. 

## <a name="accessing-azure-cosmos-db-cassandra-api-from-spark-scala-programs"></a>Accès à l’API Cassandra Azure Cosmos DB à partir de programmes Scala Spark

Les programmes Spark à exécuter en tant que processus automatisés sur Azure Databricks sont soumis au cluster au moyen du script [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html) et planifiés pour être exécutés par le biais de travaux Azure Databricks.

Les liens suivants vous aident à vous familiariser avec la génération de programmes Scala Spark pour interagir avec l’API Cassandra Azure Cosmos DB.
* [Guide pratique pour se connecter à l’API Cassandra Azure Cosmos DB à partir d’un programme Scala Spark](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-connector-sample/blob/main/src/main/scala/com/microsoft/azure/cosmosdb/cassandra/SampleCosmosDBApp.scala)
* [How to run a Spark Scala program as an automated job on Azure Databricks](/azure/databricks/jobs) (Guide pratique pour exécuter un programme Scala Spark en tant que travail automatisé sur Azure Databricks)
* [Liste complète d’exemples de code illustrant l’utilisation de l’API Cassandra](connect-spark-configuration.md#next-steps)

## <a name="next-steps"></a>Étapes suivantes

Familiarisez-vous avec la [création d’un compte d’API Cassandra, d’une base de données et d’une table](create-account-java.md) à l’aide d’une application Java.
