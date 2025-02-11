---
title: Métadonnées et traçabilité à partir de Looker
description: Cet article décrit l’extraction de la traçabilité des données depuis une source Looker.
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 08/12/2021
ms.openlocfilehash: bed8395940643e56d586369fd00341b4c28b27f9
ms.sourcegitcommit: 6c6b8ba688a7cc699b68615c92adb550fbd0610f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122563364"
---
# <a name="how-to-get-lineage-from-looker-into-azure-purview"></a>Comment obtenir la traçabilité depuis Looker dans Azure Purview

Cet article aborde les aspects de la traçabilité des données d’une source Looker dans Azure Purview. Le prérequis pour voir la traçabilité des données dans Purview pour Looker est d’[analyser votre instance Looker.](../purview/register-scan-looker-source.md) 

## <a name="lineage-of-looker-artifacts-in-azure-purview"></a>Traçabilité des artefacts Looker dans Azure Purview

Les utilisateurs peuvent rechercher les artefacts Looker par nom, par description ou par d’autres informations détaillées pour afficher les résultats pertinents. Sous l’onglet Vue d’ensemble et propriétés des ressources, les détails de base comme la description, les propriétés et d’autres informations sont affichés. Sous l’onglet Traçabilité, les relations des ressources entre les dispositions et les vues Looker sont affichées. Par conséquent, les vues Looker d’aujourd’hui comporteront des informations de traçabilité provenant des tables. 

:::image type="content" source="./media/how-to-lineage-looker/lineage.png" alt-text="Capture d’écran montrant le rendu de la traçabilité pour Looker." lightbox="./media/how-to-lineage-looker/lineage.png":::


## <a name="next-steps"></a>Étapes suivantes

- [Découvrir la traçabilité des données dans Azure Purview](catalog-lineage-user-guide.md)
- [Lier Azure Data Factory pour pousser la traçabilité automatisée](how-to-link-azure-data-factory.md)
