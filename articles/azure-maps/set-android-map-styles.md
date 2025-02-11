---
title: Définition d’un style de carte sur les cartes Android | Microsoft Azure Maps
description: Découvrez deux façons de définir le style d’une carte. Découvrez comment utiliser l’Android SDK de Azure Maps dans le fichier layout ou la classe d’activité pour ajuster le style.
author: rbrundritt
ms.author: richbrun
ms.date: 02/26/2021
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
zone_pivot_groups: azure-maps-android
ms.openlocfilehash: 84e87593a67bfda512619c5637ffc13c07fa8111
ms.sourcegitcommit: 8b38eff08c8743a095635a1765c9c44358340aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/30/2021
ms.locfileid: "113091269"
---
# <a name="set-map-style-android-sdk"></a>Définition d’un style de carte (Android SDK)

Cet article vous présente deux façons de définir des styles de carte à l’aide d’Android SDK Azure Maps. Azure Maps offre le choix entre six styles de carte. Pour plus d’informations sur les styles de carte pris en charge, consultez [Styles de carte pris en charge dans Azure Maps](supported-map-styles.md).

## <a name="prerequisites"></a>Conditions préalables requises

Veillez à suivre la procédure du document [Démarrage rapide : Création d’une application Android](quick-android-map.md).

>[!important]
>Un compte Azure Maps du niveau tarifaire Gen1 ou Gen2 est nécessaire pour suivre la procédure de cette section. Pour plus d’informations sur les niveaux tarifaires, consultez [Choix du bon niveau tarifaire dans Azure Maps](choose-pricing-tier.md).


## <a name="set-map-style-in-the-layout"></a>Définir le style de carte dans la disposition

Vous pouvez définir un style de carte dans le fichier de disposition de votre classe d’activité lorsque vous ajoutez le contrôle de carte. Le code suivant définit l’emplacement du centre, le niveau de zoom et le style de carte.

```XML
<com.azure.android.maps.control.MapControl
    android:id="@+id/mapcontrol"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:azure_maps_centerLat="47.602806"
    app:azure_maps_centerLng="-122.329330"
    app:azure_maps_zoom="12"
    app:azure_maps_style="grayscale_dark"
    />
```

La capture d’écran suivante montre le rendu, obtenu avec le code ci-dessus, d’une carte routière avec le style de nuances de gris foncé.

![Carte routière avec un style de nuances de gris foncé](media/set-android-map-styles/android-grayscale-dark.png)

## <a name="set-map-style-in-code"></a>Définition du style de carte dans le code

Le style de carte peut être défini programmatiquement dans le code avec la méthode `setStyle` de la carte. Le code suivant définit l’emplacement du centre et le niveau de zoom à l’aide de la méthode `setCamera` de la carte et spécifie le style de carte `SATELLITE_ROAD_LABELS`.

::: zone pivot="programming-language-java-android"

```java
mapControl.onReady(map -> {

    //Set the camera of the map.
    map.setCamera(center(Point.fromLngLat(-122.33, 47.64)), zoom(14));

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE_ROAD_LABELS));
});
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
mapControl!!.onReady { map: AzureMap ->
    //Set the camera of the map.
    map.setCamera(center(Point.fromLngLat(-122.33, 47.64)), zoom(14))

    //Set the style of the map.
    map.setStyle(style(MapStyle.SATELLITE_ROAD_LABELS))
}
```

::: zone-end

La capture d’écran suivante montre le rendu, obtenu avec le code ci-dessus, d’une carte avec le style des étiquettes de route satellite.

![Carte avec le style des étiquettes de routes satellite](media/set-android-map-styles/android-satellite-road-labels.png)

## <a name="setting-the-map-camera"></a>Définition de la caméra de la carte

La caméra de la carte contrôle la région du monde qui est affichée dans le point de vue de la carte. Elle peut se trouver dans la disposition ou être spécifiée programmatiquement dans le code. Dans le deuxième cas, il existe deux méthodes principales pour définir la position de la carte : avec center et zoom ou en passant un cadre englobant. Le code suivant montre comment définir toutes les options de caméra facultatives avec `center` et `zoom`.

::: zone pivot="programming-language-java-android"

```java
//Set the camera of the map using center and zoom.
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.64)), 

    //The zoom level. Typically a value between 0 and 22.
    zoom(14),

    //The amount of tilt in degrees the map where 0 is looking straight down.
    pitch(45),

    //Direction the top of the map is pointing in degrees. 0 = North, 90 = East, 180 = South, 270 = West
    bearing(90),

    //The minimum zoom level the map will zoom-out to when animating from one location to another on the map.
    minZoom(10),
    
    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Set the camera of the map using center and zoom.
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.64)), 

    //The zoom level. Typically a value between 0 and 22.
    zoom(14),

    //The amount of tilt in degrees the map where 0 is looking straight down.
    pitch(45),

    //Direction the top of the map is pointing in degrees. 0 = North, 90 = East, 180 = South, 270 = West
    bearing(90),

    //The minimum zoom level the map will zoom-out to when animating from one location to another on the map.
    minZoom(10),
    
    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
)
```

::: zone-end

Il est souvent souhaitable de concentrer la carte sur un jeu de données. Un cadre englobant peut être calculé à partir des caractéristiques à l’aide de la méthode `MapMath.fromData` et transmis dans l’option `bounds` de la caméra de la carte. Lors de la définition d’une vue cartographique basée sur un cadre englobant, il est souvent utile de spécifier une valeur `padding` pour tenir compte de la taille en pixels des points rendus sous forme de bulles ou de symboles. Le code suivant montre comment définir toutes les options de caméra facultatives lorsqu’un cadre englobant est utilisé pour définir la position de la caméra.

::: zone pivot="programming-language-java-android"

```java
//Set the camera of the map using a bounding box.
map.setCamera(
    //The area to focus the map on.
    bounds(BoundingBox.fromLngLats(
        //West
        -122.4594,

        //South
        47.4333,
        
        //East
        -122.21866,
        
        //North
        47.75758
    )),

    //Amount of pixel buffer around the bounding box to provide extra space around the bounding box.
    padding(20),

    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
//Set the camera of the map using a bounding box.
map.setCamera(
    //The area to focus the map on.
    bounds(BoundingBox.fromLngLats(
        //West
        -122.4594,

        //South
        47.4333,
        
        //East
        -122.21866,
        
        //North
        47.75758
    )),

    //Amount of pixel buffer around the bounding box to provide extra space around the bounding box.
    padding(20),

    //The maximum zoom level the map will zoom-in to when animating from one location to another on the map.
    maxZoom(14)
)
```

::: zone-end

Les proportions d’un cadre englobant ne sont pas forcément les mêmes que celles de la carte. De ce fait, la carte affiche souvent le cadre englobant dans sa totalité, mais serré verticalement ou horizontalement seulement.

### <a name="animate-map-view"></a>Animer la vue cartographique

Lorsque vous définissez les options de caméra de la carte, vous pouvez également utiliser des options d’animation pour créer une transition entre la vue de la carte actuelle et la suivante. Ces options spécifient le type d’animation et la durée nécessaire pour déplacer la caméra.

| Option | Description |
|--------|-------------|
| `animationDuration(Integer durationMs)` | Spécifie la durée d’animation de la caméra entre les vues en millisecondes (ms). |
| `animationType(AnimationType animationType)` | Spécifie le type de transition d’animation à effectuer.<br/><br/> - `JUMP` - modification immédiate.<br/> - `EASE` - modification progressive des paramètres de la caméra.<br/> - `FLY` - modification progressive des paramètres de la caméra à la suite d’un vol ressemblant à un arc. |

Le code suivant montre comment animer la vue de la carte en utilisant une animation `FLY` sur une durée de trois secondes.

::: zone pivot="programming-language-java-android"

``` java
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.6)),
    zoom(12),
    animationType(AnimationType.FLY), 
    animationDuration(3000)
);
```

::: zone-end

::: zone pivot="programming-language-kotlin"

```kotlin
map.setCamera(
    center(Point.fromLngLat(-122.33, 47.6)),
    zoom(12.0),
    AnimationOptions.animationType(AnimationType.FLY),
    AnimationOptions.animationDuration(3000)
)
```

::: zone-end

L’exemple suivant montre le code ci-dessus animant la vue cartographique de New York à Seattle.

![Mapper l’animation de la caméra de New York à Seattle](media/set-android-map-styles/android-animate-camera.gif)

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir plus d’exemples de code à ajouter à vos cartes, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Ajouter une couche de symboles](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [Ajouter une couche de bulles](map-add-bubble-layer-android.md)
