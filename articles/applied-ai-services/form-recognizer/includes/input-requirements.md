---
author: laujan
ms.service: applied-ai-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 06/27/2019
ms.author: lajanuar
ms.openlocfilehash: fbf4fc405c92700f993a488d94334338ab2757b4
ms.sourcegitcommit: 5f659d2a9abb92f178103146b38257c864bc8c31
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/17/2021
ms.locfileid: "122326916"
---
Form Recognizer fonctionne sur les documents d’entrée qui répondent aux critères suivants :

* Format JPG, PNG, PDF (texte ou numérisé) ou TIFF. Les PDF englobant du texte sont préférables, car il n’existe aucun risque d’erreur au niveau de l’extraction et de l’emplacement des caractères.
* La taille du fichier doit être inférieure à 50 Mo.
* Les dimensions des images doivent être comprises entre 50 x 50 et 10 000 x 10 000 pixels.
* Les dimensions des PDF ne doivent pas dépasser 17 x 17 pouces, ce qui correspond aux formats de papier Legal ou A3, et plus petits.
* Pour PDF et TIFF, seules les 200 premières pages sont traitées (avec un abonnement gratuit, seules les deux premières pages sont traitées).
* La taille totale du jeu de données d’apprentissage doit être de 500 pages maximum.
* Si vos fichiers PDF sont verrouillés par mot de passe, vous devez supprimer le verrou avant de les soumettre.
* S’ils sont numérisés à partir de documents papier, la qualité de numérisation des formulaires doit être supérieure.
* Pour un apprentissage non supervisé (sans données étiquetées), les données doivent contenir des clés et des valeurs.
* Pour un apprentissage non supervisé (sans données étiquetées), les clés doivent s’afficher au-dessus ou à gauche des valeurs. Elles ne peuvent pas s’afficher ni en dessous ni à droite.
