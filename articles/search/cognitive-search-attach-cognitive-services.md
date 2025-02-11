---
title: Joindre Cognitive Services à un ensemble de compétences
titleSuffix: Azure Cognitive Search
description: Découvrez comment attacher une ressource Cognitive Services multiservice à un pipeline d’enrichissement par IA dans Recherche cognitive Azure.
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/12/2021
ms.openlocfilehash: c5f29cb909dae7b8e362cdf37481111dbf75c8d9
ms.sourcegitcommit: 86ca8301fdd00ff300e87f04126b636bae62ca8a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/16/2021
ms.locfileid: "122525855"
---
# <a name="attach-a-cognitive-services-resource-to-a-skillset-in-azure-cognitive-search"></a>Attacher une ressource Cognitive Services à un ensemble de compétences dans Recherche cognitive Azure

Lorsque vous configurez un [pipeline d’enrichissement par IA](cognitive-search-concept-intro.md) dans la Recherche cognitive Azure, vous pouvez enrichir un nombre limité de documents gratuitement. Pour les charges de travail plus importantes et plus fréquentes, vous devez attacher une [ressource Cognitive Services multiservice](../cognitive-services/cognitive-services-apis-create-account.md) facturable. Une ressource multiservice référence « Cognitive Services » en tant qu’offre (et non en tant que services individuels) dont l’accès est octroyé via une seule clé API.

Une clé de ressource est spécifiée dans un ensemble de compétences et permet à Microsoft de vous facturer l’utilisation des API suivantes :

+ [Vision par ordinateur](https://azure.microsoft.com/services/cognitive-services/computer-vision/) pour l’analyse d’images et la reconnaissance optique de caractères (OCR)
+ [Analyse de texte](https://azure.microsoft.com/services/cognitive-services/text-analytics/) pour la détection de la langue, la reconnaissance des entités, l’analyse des sentiments et l’extraction des expressions clés
+ [Traduction de texte](https://azure.microsoft.com/services/cognitive-services/translator-text-api/)

La clé est utilisée pour la facturation, mais pas pour les connexions. De manière interne, un service de recherche se connecte à une ressource Cognitive Services colocalisée dans la [même région physique](https://azure.microsoft.com/global-infrastructure/services/?products=search).

## <a name="key-requirements"></a>Conditions clés

Une clé de ressource est nécessaire pour les [compétences intégrées](cognitive-search-predefined-skills.md) facturables utilisées plus de 20 fois par jour sur le même indexeur. Les compétences qui effectuent des appels back-end à Cognitive Services incluent la liaison d’entités, la reconnaissance d’entités, l’analyse d’images, l’extraction de phrases clés, la détection de langue, l’OCR, la détection d’informations d’identification personnelle, l’analyse des sentiments ou la traduction de texte.

La fonctionnalité [Recherche d’entité personnalisée](cognitive-search-skill-custom-entity-lookup.md) est mesurée par la Recherche cognitive Azure, et non par Cognitive Services. Toutefois, elle nécessite une clé de ressource Cognitive Services pour déverrouiller les transactions au-delà de 20 transactions par indexeur et par jour. Pour cette compétence uniquement, même si la clé de ressource permet de débloquer le nombre de transactions, elle n’est pas liée à la facturation.

Vous pouvez omettre la clé et la section Cognitive Services pour les ensembles de compétences constitués uniquement de compétences personnalisées ou de compétences utilitaires (Logique conditionnelle, Extraction de documents, Modélisateur, Fusion de texte, Fractionnement de texte). Vous pouvez également omettre la section si votre utilisation des compétences facturables est inférieure à 20 transactions par indexeur et par jour.

## <a name="how-billing-works"></a>Comment la facturation fonctionne

+ La Recherche cognitive Azure utilise la clé de ressource Cognitive Services que vous fournissez sur un ensemble de compétences pour facturer l’enrichissement des images et du texte. L’exécution de qualifications facturables est facturée au [tarif de paiement à l’utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).

+ L’extraction d’images est une opération de la Recherche cognitive Azure qui se produit lorsque les documents sont décodés avant l’enrichissement. L’extraction d’images est facturable pour tous les niveaux, à l’exception des 20 extractions quotidiennes gratuites du niveau Gratuit. Les coûts d’extraction d’images s’appliquent aux fichiers image contenus dans les objets blob, aux images incorporées dans d’autres fichiers (fichiers PDF et autres fichiers d’application) ainsi qu’aux images extraites à l’aide de l’[extraction de documents](cognitive-search-skill-document-extraction.md). Pour connaître les prix appliqués à l’extraction d’images, voir la [page de tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).

+ L’extraction de texte a lieu également durant la phase de [craquage de document](search-indexer-overview.md#document-cracking). Elle n’est pas facturable.

+ Les qualifications qui n’appellent pas Cognitive Services, y compris les compétences conditionnelles, le modélisateur, la fusion de texte et le fractionnement du texte, ne sont pas facturables. 

  Comme indiqué, la [recherche d’entité personnalisée](cognitive-search-skill-custom-entity-lookup.md) est un cas particulier dans la mesure où elle nécessite une clé, mais qu’elle est [mesurée par le service Recherche cognitive](https://azure.microsoft.com/pricing/details/search/#pricing).

> [!TIP]
> Pour réduire le coût du traitement des ensembles de compétences, activez l’[enrichissement incrémentiel (préversion)](cognitive-search-incremental-indexing-conceptual.md) afin de mettre en cache et de réutiliser les enrichissements non affectés par les changements apportés à un ensemble de compétences. La mise en cache nécessite Stockage Azure (consultez les [tarifs](/pricing/details/storage/blobs/)). Toutefois, le coût cumulé de l’exécution des ensembles de compétences est moindre si les enrichissements existants peuvent être réutilisés, en particulier pour les ensembles de compétences qui utilisent l’analyse et l’extraction d’images.

## <a name="same-region-requirement"></a>Exigence de même région

Recherche cognitive et Cognitive Services doivent être présents dans la même région physique, comme indiqué sur la page [Disponibilité des produits](https://azure.microsoft.com/global-infrastructure/services/?products=search). La plupart des régions qui offrent Recherche cognitive proposent également Cognitive Services.

Si vous essayez l’enrichissement par IA dans une région qui ne présente pas les deux services, le message suivant s’affiche : « La clé fournie n’est pas une clé de type CognitiveServices valide pour la région de votre service de recherche ».

> [!NOTE]
> Certaines compétences intégrées sont basées sur des services cognitifs non régionaux (par exemple la [compétence de traduction de texte](cognitive-search-skill-text-translation.md)). L’utilisation d’une compétence non régionale signifie que votre requête peut être desservie dans une région autre que la région de la Recherche cognitive Azure. Pour plus d’informations sur les services non régionaux, consultez la page [Produit Cognitive Services par région](https://aka.ms/allinoneregioninfo).

## <a name="use-free-resources"></a>Utiliser des ressources gratuites

Vous pouvez utiliser une option de traitement gratuite, limitée aux exercices des guides de démarrage rapide et des tutoriels d’enrichissement par IA.

Les ressources du niveau tarifaire Gratuit (enrichissements limités) sont limitées à 20 documents par jour, par indexeur. Vous pouvez [réinitialiser l’indexeur](search-howto-run-reset-indexers.md) pour réinitialiser le compteur.

Si vous utilisez l’Assistant **Importation de données** pour l’enrichissement par IA, vous trouverez les options « Attacher Cognitive Services » sur la page **Ajouter l’enrichissement par IA (facultatif)** .

![Section Attacher Cognitive Services développée](./media/cognitive-search-attach-cognitive-services/attach1.png "Section Attacher Cognitive Services développée")

## <a name="use-billable-resources"></a>Utiliser des ressources facturables

Pour les charges de travail qui créent plus de 20 enrichissements facturables par jour, veillez à attacher une ressource Cognitive Services. 

Si vous utilisez l’Assistant **Importation de données**, vous pouvez configurer une ressource facturable à partir de la page **Ajouter l’enrichissement par IA (facultatif)** .

1. Développez **Attacher Cognitive Services**, puis sélectionnez **Créer une ressource Cognitive Services**. Un nouvel onglet s’ouvre pour vous permettre de créer la ressource :

   ![Créer une ressource Cognitive Services](./media/cognitive-search-attach-cognitive-services/cog-services-create.png "Créer une ressource Cognitive Services")

1. Dans la liste **Emplacement**, sélectionnez la région qui contient votre service de recherche.

1. Dans la liste **Niveau tarifaire**, sélectionnez **S0** pour obtenir la collection complète de fonctionnalités Cognitive Services, y compris les fonctionnalités Vision et Langue qui sous-tendent les compétences prédéfinies fournies par Recherche cognitive Azure.

   Pour le niveau S0, vous pouvez trouver les tarifs des charges de travail spécifiques dans la [page de tarification de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).
  
   + Dans la liste **Sélectionner une offre**, assurez-vous que **Cognitive Services** est sélectionné.
   + Sous les fonctionnalités **Langue**, les tarifs d’**Analyse de texte** standard s’appliquent à l’indexation basée sur l’intelligence artificielle.
   + Sous les fonctionnalités **Vision**, les tarifs de **Vision par ordinateur S1** s’appliquent.

1. Sélectionnez **Créer** pour approvisionner la nouvelle ressource Cognitive Services.

1. Revenez à l’onglet précédent. Sélectionnez **Actualiser** pour afficher la ressource Cognitive Services, puis sélectionnez-la :

   ![Sélectionner la ressource Cognitive Services](./media/cognitive-search-attach-cognitive-services/attach2.png "Sélectionner la ressource Cognitive Services")

1. Développez la section **Ajouter des compétences cognitives** pour sélectionner les qualifications cognitives spécifiques à exécuter sur vos données. Suivez les recommandations restantes de l’Assistant.

## <a name="attach-an-existing-skillset-to-a-cognitive-services-resource"></a>Attacher un ensemble de qualifications à une ressource Cognitive Services existante

Si vous avez un ensemble de qualifications existant, vous pouvez l’attacher à une ressource Cognitive Services nouvelle ou différente.

1. Dans la page de la vue d’ensemble du service de recherche, sélectionnez **Ensembles de compétences** :

   ![Onglet Ensemble de compétences](./media/cognitive-search-attach-cognitive-services/attach-existing1.png "Onglet Ensemble de compétences")

1. Sélectionnez le nom de l’ensemble de qualifications, puis choisissez une ressource existante ou créez-en une. Cliquez sur **OK** pour confirmer vos modifications.

   ![Liste de ressources de l’ensemble de compétences](./media/cognitive-search-attach-cognitive-services/attach-existing2.png "Liste de ressources de l’ensemble de compétences")

   N’oubliez pas que l’option **Gratuit (enrichissements limités)** vous limite à 20 documents par jour et que vous pouvez utiliser l’option **Créer une ressource Cognitive Services** pour approvisionner une nouvelle ressource facturable. Si vous avez créé une ressource, sélectionnez **Actualiser** pour rafraîchir la liste des ressources Cognitive Services, puis sélectionnez la ressource.

## <a name="attach-cognitive-services-programmatically"></a>Attacher Cognitive Services par programme

Lorsque vous définissez l’ensemble de qualifications par programme, ajoutez une section `cognitiveServices` à l’ensemble de qualifications. Dans cette section, incluez la clé de la ressource Cognitive Services que vous souhaitez associer à l’ensemble de qualifications. Rappelez-vous que la ressource doit se trouver dans la même région que votre ressource Recherche cognitive Azure. Incluez également `@odata.type`, que vous définissez sur `#Microsoft.Azure.Search.CognitiveServicesByKey`.

L’exemple ci-après illustre ce modèle. Notez la section `cognitiveServices` à la fin de la définition.

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2020-06-30
api-key: [admin key]
Content-Type: application/json
{
    "name": "skillset name",
    "skills": 
    [
      {
        "@odata.type": "#Microsoft.Skills.Text.V3.EntityRecognitionSkill",
        "categories": [ "Organization" ],
        "defaultLanguageCode": "en",
        "inputs": [
          {
            "name": "text", "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "organizations", "targetName": "organizations"
          }
        ]
      }
    ],
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "mycogsvcs",
        "key": "<your key goes here>"
    }
}
```

## <a name="example-estimate-costs"></a>Exemple : Estimer les coûts

Pour estimer les coûts associés à l’indexation de Recherche cognitive, commencez par vous représenter à quoi ressemble un document moyen afin de pouvoir donner une approximation. Par exemple, vous pourriez faire une approximation :

+ 1 000 fichiers PDF.
+ Six pages par fichier.
+ Une image par page (6 000 images).
+ 3 000 caractères par page.

Supposons un pipeline consistant en un craquage de document PDF, une extraction d’image et de texte, une reconnaissance optique de caractères (OCR) d’images, et une reconnaissance d’entités d’organisations.

Les prix indiqués dans cet article sont hypothétiques. Ils sont utilisés pour illustrer le processus d’estimation. Vos coûts peuvent être inférieurs. Pour obtenir les prix réels des transactions, voir [Tarification Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services).

1. Pour le décodage de documents avec contenu de texte et d’image, l’extraction de texte est actuellement gratuite. Pour 6 000 images, supposez un prix de 1 USD pour chaque tranche de 1 000 images extraites. Cela revient à un coût de 6,00 USD pour cette étape.

2. Pour la reconnaissance optique de caractères (OCR) de 6 000 images en anglais, la qualification cognitive OCR utilise le meilleur algorithme (DescribeText). À raison de 2,50 $ par lot de 1 000 images à analyser, cette étape vous coûterait 15 $.

3. Pour l’extraction d’entité, vous auriez un total de trois enregistrements texte par page. Chaque enregistrement contient 1 000 caractères. Trois enregistrements texte par page multipliés par 6 000 pages équivalent à 18 000 enregistrements texte. À raison de 2 $ par lot de 1 000 enregistrements texte, cette étape vous coûterait 36 $.

En additionnant tout cela, l’ingestion de 1 000 documents PDF de ce type avec l’ensemble de qualifications décrit vous reviendrait à environ 57 USD.

## <a name="next-steps"></a>Étapes suivantes

+ [Page de tarification de Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Créer un jeu de compétences (REST)](/rest/api/searchservice/create-skillset)
+ [Guide pratique pour mapper des champs enrichis](cognitive-search-output-field-mapping.md)
