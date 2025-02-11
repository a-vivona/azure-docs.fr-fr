---
title: Exemples de programmes et procédures pas à pas sur le ML
titleSuffix: Azure Data Science Virtual Machine
description: Grâce à ces exemples et procédures pas à pas, découvrez comment gérer les tâches et les scénarios courants avec la machine Data Science Virtual Machine.
keywords: outils de science des données, machine virtuelle science des données, outils pour la science des données, science des données linux
services: machine-learning
ms.service: data-science-vm
author: timoklimmer
ms.author: tklimmer
ms.topic: conceptual
ms.date: 05/12/2021
ms.openlocfilehash: d907be8262fdc403f1e7b550d57c1aeaf77491fa
ms.sourcegitcommit: 4f185f97599da236cbed0b5daef27ec95a2bb85f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2021
ms.locfileid: "112369053"
---
# <a name="samples-on-azure-data-science-virtual-machines"></a>Exemples sur Data Science Virtual Machine

Les machines Azure Data Science Virtual Machine (DSVM) incluent un ensemble complet d’exemple de code. Ces exemples incluent des blocs-notes Jupyter et des scripts dans des langages tels que Python et R.
> [!NOTE]
> Pour plus d’informations sur l’exécution de blocs-notes Jupyter Notebook sur vos machines DSVM, consultez la section [Accéder à Jupyter](#access-jupyter).

## <a name="prerequisites"></a>Prérequis

Pour exécuter ces exemples, vous devez avoir approvisionné une [Data Science Virtual Machine Ubuntu](./dsvm-ubuntu-intro.md).

## <a name="available-samples"></a>Exemples disponibles
| Catégorie d’exemples | Description | Emplacements |
| ------------- | ------------- | ------------- |
| Langage Python  | Les exemples décrivent des scénarios tels que la connexion aux magasins de données cloud basés sur Azure et l’utilisation d’Azure Machine Learning.  <br/> [Langage Python](#python-language) | <br/>`~notebooks` <br/><br/>|
| Langage Julia  | Fournit une description détaillée du traçage et du Deep Learning dans Julia. Explique également comment appeler les langages C et Python à partir de Julia. <br/> [Langage Julia](#julia-language) |<br/> Windows :<br/> `~notebooks/Julia_notebooks`<br/><br/> Linux :<br/> `~notebooks/julia`<br/><br/> |
| Azure Machine Learning  | Générez des modèles Machine Learning et d’apprentissage profond avec Machine Learning. Déployez-les n’importe où. Utilisez le réglage intelligent des hyperparamètres et le réglage Machine Learning automatisé, ainsi que la gestion des modèles et la formation distribuée. <br/> [Machine Learning](#azure-machine-learning) | <br/>`~notebooks/AzureML`<br/> <br/>|
| Blocs-notes PyTorch  | Exemples Deep Learning qui utilisent des réseaux neuronaux basés sur PyTorch. La gamme des blocs-notes s’étend des scénarios pour débutants aux scénarios pour utilisateurs avancés.  <br/> [Blocs-notes PyTorch](#pytorch) | <br/>`~notebooks/Deep_learning_frameworks/pytorch`<br/> <br/>|
| TensorFlow  |  Variété d’exemples et techniques de réseau neuronal implémentés à l’aide de l’infrastructure TensorFlow. <br/> [TensorFlow](#tensorflow) | <br/>`~notebooks/Deep_learning_frameworks/tensorflow`<br/><br/> |
| H2O   | Exemples basés sur Python qui utilisent H2O pour des scénarios problématiques concrets. <br/> [H2O](#h2o) | <br/>`~notebooks/h2o`<br/><br/> |
| Langage SparkML  | Exemples utilisant les fonctionnalités de la boîte à outils Apache Spark MLLib via pySpark et MMLSpark : Microsoft Machine Learning pour Apache Spark sur Apache Spark 2.x.  <br/> [Langage SparkML](#sparkml) | <br/>`~notebooks/SparkML/pySpark`<br/>`~notebooks/MMLSpark`<br/><br/>  |
| XGBoost | Exemples d’apprentissage automatique standard dans XGBoost pour les scénarios tels que la classification et la régression. <br/> [XGBoost](#xgboost) | <br/>Windows :<br/>`\dsvm\samples\xgboost\demo`<br/><br/> |

<br/>

## <a name="access-jupyter"></a>Accéder à Jupyter 

Pour accéder à Jupyter, sélectionnez l’icône **Jupyter** sur le Bureau ou dans le menu de l’application. Vous pouvez également accéder à Jupyter sur une édition Linux d’une machine DSVM. Pour accéder à distance à partir d’un navigateur web, accédez à `https://<Full Domain Name or IP Address of the DSVM>:8000` sur Ubuntu.

Pour ajouter des exceptions et permettre l’accès à Jupyter via un navigateur, suivez les instructions suivantes :


![Activation d’une exception Jupyter](./media/ubuntu-jupyter-exception.png)


Connectez-vous avec le mot de passe que vous utilisez pour vous connecter à la machine Data Science Virtual Machine.
<br/>

**Page d’accueil de Jupyter**
<br/>![Page d’accueil de Jupyter](./media/jupyter-home.png)<br/>

## <a name="r-language"></a>Langage R 
<br/>![Exemples R](./media/r-language-samples.png)<br/>

## <a name="python-language"></a>Langage Python
<br/>![Exemples Python](./media/python-language-samples.png)<br/>

## <a name="julia-language"></a>Langage Julia 
<br/>![Exemples Julia](./media/julia-samples.png)<br/>

## <a name="azure-machine-learning"></a>Azure Machine Learning 
<br/>![Exemples Azure Machine Learning](./media/azureml-samples.png)<br/>

## <a name="pytorch"></a>PyTorch
<br/>![Exemples PyTorch](./media/pytorch-samples.png)<br/>

## <a name="tensorflow"></a>TensorFlow 
<br/>![Exemples TensorFlow](./media/tensorflow-samples.png)<br/>

## <a name="h2o"></a>H2O 
<br/>![Exemples H2O](./media/h2o-samples.png)<br/>

## <a name="sparkml"></a>SparkML 
<br/>![Exemples SparkML](./media/sparkml-samples.png)<br/>

## <a name="xgboost"></a>XGBoost 
<br/>![Exemples XGBoost](./media/xgboost-samples.png)<br/>