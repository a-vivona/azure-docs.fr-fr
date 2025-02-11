---
title: Demander des données météorologiques en temps réel et prévues à l’aide des services Météo Azure Maps
description: Découvrez comment demander des données météorologiques en temps réel (actuelles) et prévues (par minute, heure, jour) à l’aide des services Météo Microsoft Azure Maps
author: anastasia-ms
ms.author: v-stharr
ms.date: 04/26/2021
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
ms.custom: mvc
ms.openlocfilehash: ed0985778f27f17292428dddadaf4d0dedc9cd46
ms.sourcegitcommit: 0046757af1da267fc2f0e88617c633524883795f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/13/2021
ms.locfileid: "122562780"
---
# <a name="request-real-time-and-forecasted-weather-data-using-azure-maps-weather-services"></a>Demander des données météorologiques en temps réel et prévues à l’aide des services Météo Azure Maps

Le [service Météo](/rest/api/maps/weather) Azure Maps est un ensemble d’API RESTful qui permet aux développeurs d’intégrer des données météorologiques dynamiques, historiques, en temps réel et prévisionnelles, ainsi que des visualisations dans leurs solutions. Dans cet article, nous allons vous montrer comment demander des données météorologiques en temps réel et prévues.

Dans cet article, vous allez apprendre à :

* Demander des données météorologiques en temps réel (actuelles) à l’aide de [l’API Get Current Conditions](/rest/api/maps/weather/getcurrentconditions).
* Demander des alertes météorologiques graves à l’aide de [l’API Get Severe Weather Alerts](/rest/api/maps/weather/getsevereweatheralerts).
* Demander des prévisions quotidiennes à l’aide de [l’API Get Daily Forecast](/rest/api/maps/weather/getdailyforecast).
* Demander des prévisions horaires à l’aide de [l’API Get Hourly Forecast](/rest/api/maps/weather/gethourlyforecast).
* Demander des prévisions minute par minute à l’aide de [l’API Get Minute Forecast](/rest/api/maps/weather/getminuteforecast).

Cette vidéo fournit des exemples montrant comment effectuer des appels REST vers le service Météo Azure Maps.

</br>

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-Maps-Weather-services-for-developers/player?format=ny]

## <a name="prerequisites"></a>Prérequis

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement. Pour plus d’informations sur l’authentification dans Azure Maps, voir [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md).

    >[!IMPORTANT]
    >Un niveau tarifaire Gen1 (S1) or Gen2 est nécessaire pour utiliser [l’API Get Minute Forecast](/rest/api/maps/weather/getminuteforecast).  Toutes les autres API requièrent une clé de niveau tarifaire S0.

Ce tutoriel utilise l’application [Postman](https://www.postman.com/), mais vous pouvez choisir un autre environnement de développement d’API.

## <a name="request-real-time-weather-data"></a>Demander des données météorologiques en temps réel

[L’API Get Current Conditions](/rest/api/maps/weather/getcurrentconditions) retourne des conditions météorologiques détaillées telles que les précipitations, la température et le vent pour des coordonnées données. En outre, il est possible de récupérer les observations des dernières 6 heures ou 24 heures pour un emplacement spécifique. La réponse contient des détails tels que la date et l’heure de l’observation, une brève description des conditions météorologiques, une icône météo, des indicateurs de précipitation et la température. La température RealFeel™ et l’indice UV sont également retournés.

Dans cet exemple, vous allez utiliser [l’API Get Current Conditions](/rest/api/maps/weather/getcurrentconditions) pour récupérer les conditions météorologiques actuelles pour des coordonnées situées à Seattle, WA.

1. Ouvrez l’application Postman. Pour créer la demande, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer), sélectionnez **HTTP Request** (Requête HTTP). Entrez un **Request name** (Nom de demande) pour la demande. 

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/weather/currentConditions/json?api-version=1.0&query=47.60357,-122.32945&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Cliquez sur le bouton bleu **Envoyer**. Le corps de la réponse contient les informations météorologiques actuelles.

    ```json
    {
    "results": [
        {
            "dateTime": "2020-10-19T20:39:00+00:00",
            "phrase": "Cloudy",
            "iconCode": 7,
            "hasPrecipitation": false,
            "isDayTime": true,
            "temperature": {
                "value": 12.4,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperature": {
                "value": 13.7,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperatureShade": {
                "value": 13.7,
                "unit": "C",
                "unitType": 17
            },
            "relativeHumidity": 87,
            "dewPoint": {
                "value": 10.3,
                "unit": "C",
                "unitType": 17
            },
            "wind": {
                "direction": {
                    "degrees": 23.0,
                    "localizedDescription": "NNE"
                },
                "speed": {
                    "value": 4.5,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "windGust": {
                "speed": {
                    "value": 9.0,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "uvIndex": 1,
            "uvIndexPhrase": "Low",
            "visibility": {
                "value": 9.7,
                "unit": "km",
                "unitType": 6
            },
            "obstructionsToVisibility": "",
            "cloudCover": 100,
            "ceiling": {
                "value": 1494.0,
                "unit": "m",
                "unitType": 5
            },
            "pressure": {
                "value": 1021.2,
                "unit": "mb",
                "unitType": 14
            },
            "pressureTendency": {
                "localizedDescription": "Steady",
                "code": "S"
            },
            "past24HourTemperatureDeparture": {
                "value": -2.1,
                "unit": "C",
                "unitType": 17
            },
            "apparentTemperature": {
                "value": 15.0,
                "unit": "C",
                "unitType": 17
            },
            "windChillTemperature": {
                "value": 12.2,
                "unit": "C",
                "unitType": 17
            },
            "wetBulbTemperature": {
                "value": 11.3,
                "unit": "C",
                "unitType": 17
            },
            "precipitationSummary": {
                "pastHour": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past3Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past6Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past9Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past12Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past18Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past24Hours": {
                    "value": 0.4,
                    "unit": "mm",
                    "unitType": 3
                }
            },
            "temperatureSummary": {
                "past6Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 14.0,
                        "unit": "C",
                        "unitType": 17
                    }
                },
                "past12Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 14.0,
                        "unit": "C",
                        "unitType": 17
                    }
                },
                "past24Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 15.6,
                        "unit": "C",
                        "unitType": 17
                    }
                }
            }
        }
    ]
    }
    ```

## <a name="request-severe-weather-alerts"></a>Demander des alertes météorologiques graves

[L’API Azure Maps Get Severe Weather Alerts](/rest/api/maps/weather/getsevereweatheralerts) retourne les alertes météorologiques graves disponibles dans le monde entier à partir des agences météorologiques gouvernementales officielles et des fournisseurs d’alertes météorologiques globaux ou régionaux. Le service peut retourner des détails tels que le type d’alerte, la catégorie, le niveau et des descriptions détaillées sur les alertes graves actives pour l’emplacement demandé (ouragans, orages, foudre, canicule ou incendies de forêt). Par exemple, les responsables de la logistique peuvent visualiser des conditions météorologiques graves sur une carte, ainsi que les sites d’activité et les itinéraires planifiés, et mieux renseigner les chauffeurs et les travailleurs locaux.

Dans cet exemple, vous allez utiliser [l’API Get Severe Weather Alerts](/rest/api/maps/weather/getsevereweatheralerts) pour récupérer les conditions météorologiques actuelles pour des coordonnées situées à Cheyenne, WY.

>[!NOTE]
>Cet exemple récupère des alertes météorologiques graves au moment de la rédaction de cet article. Il est probable qu’il n’y ait plus d’alertes météorologiques graves à l’emplacement demandé. Pour récupérer les données d’alerte grave réelles lors de l’exécution de cet exemple, vous devez récupérer les données d’un emplacement avec des coordonnées différentes.

1. Pour créer la demande, dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer), sélectionnez **HTTP Request** (Requête HTTP). Entrez un **Request name** (Nom de demande) pour la demande.

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/weather/severe/alerts/json?api-version=1.0&query=41.161079,-104.805450&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Cliquez sur le bouton bleu **Envoyer**. S’il n’y a pas d’alertes météorologiques graves, le corps de la réponse contient un tableau `results[]` vide. S’il existe des alertes météorologiques graves, le corps de la réponse est similaire à la réponse JSON suivante :

    ```json
    {
    "results": [
        {
            "countryCode": "US",
            "alertId": 2194734,
            "description": {
                "localized": "Red Flag Warning",
                "english": "Red Flag Warning"
            },
            "category": "FIRE",
            "priority": 54,
            "source": "U.S. National Weather Service",
            "sourceId": 2,
            "alertAreas": [
                {
                    "name": "Platte/Goshen/Central and Eastern Laramie",
                    "summary": "Red Flag Warning in effect until 7:00 PM MDT.  Source: U.S. National Weather Service",
                    "startTime": "2020-10-05T15:00:00+00:00",
                    "endTime": "2020-10-06T01:00:00+00:00",
                    "latestStatus": {
                        "localized": "Continue",
                        "english": "Continue"
                    },
                    "alertDetails": "...RED FLAG WARNING REMAINS IN EFFECT FROM 9 AM THIS MORNING TO\n7 PM MDT THIS EVENING FOR STRONG GUSTY WINDS AND LOW HUMIDITY...\n\n* WHERE...Fire weather zones 303, 304, 305, 306, 307, 308, 309,\n  and 310 in southeast Wyoming. Fire weather zone 313 in Nebraska.\n\n* WIND...West to northwest 15 to 30 MPH with gusts around 40 MPH.\n\n* HUMIDITY...10 to 15 percent.\n\n* IMPACTS...Any fires that develop will likely spread rapidly.\n  Outdoor burning is not recommended.\n\nPRECAUTIONARY/PREPAREDNESS ACTIONS...\n\nA Red Flag Warning means that critical fire weather conditions\nare either occurring now...or will shortly. A combination of\nstrong winds...low relative humidity...and warm temperatures can\ncontribute to extreme fire behavior.\n\n&&",
                    "alertDetailsLanguageCode": "en"
                }
            ]
            },...
        ]
    }
    ```

## <a name="request-daily-weather-forecast-data"></a>Demander des données de prévisions météorologiques quotidiennes

[L’API Get Daily Forecast](/rest/api/maps/weather/getdailyforecast) retourne des prévisions météorologiques quotidiennes détaillées, telles que la température et le vent. La demande peut spécifier le nombre de jours à retourner : 1, 5, 10, 15, 25 ou 45 jours pour des coordonnées données. La réponse comprend des détails tels que la température, le vent, les précipitations, la qualité de l’air et l’indice UV.  Dans cet exemple, nous avons demandé des données sur cinq jours en définissant `duration=5`.

>[!IMPORTANT]
>Dans le niveau tarifaire S0, vous pouvez demander des prévisions quotidiennes pour les 1, 5, 10 et 15 jours suivants. Les niveaux tarifaires Gen1 (S1) et Gen2 offrent tous deux la possibilité de demander les prévisions quotidiennes à 25 jours et à 45 jours.

Dans cet exemple, vous allez utiliser [l’API Get Daily Forecast](/rest/api/maps/weather/getdailyforecast) pour récupérer les prévisions météorologiques sur cinq jours pour des coordonnées situées à Seattle, WA.

1. Pour créer la demande, dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer), sélectionnez **HTTP Request** (Requête HTTP). Entrez un **Request name** (Nom de demande) pour la demande.

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/weather/forecast/daily/json?api-version=1.0&query=47.60357,-122.32945&duration=5&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Cliquez sur le bouton bleu **Envoyer**. Le corps de la réponse contient les données de prévisions météorologiques sur cinq jours. Par souci de concision, la réponse JSON ci-dessous montre les prévisions pour le premier jour.
    ```json
    {
    "summary": {
        "startDate": "2020-10-18T17:00:00+00:00",
        "endDate": "2020-10-19T23:00:00+00:00",
        "severity": 2,
        "phrase": "Snow, mixed with rain at times continuing through Monday evening and a storm total of 3-6 cm",
        "category": "snow/rain"
    },
    "forecasts": [
        {
            "date": "2020-10-19T04:00:00+00:00",
            "temperature": {
                "minimum": {
                    "value": -1.1,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 1.3,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "realFeelTemperature": {
                "minimum": {
                    "value": -6.0,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 0.5,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "realFeelTemperatureShade": {
                "minimum": {
                    "value": -6.0,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 0.7,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "hoursOfSun": 1.8,
            "degreeDaySummary": {
                "heating": {
                    "value": 18.0,
                    "unit": "C",
                    "unitType": 17
                },
                "cooling": {
                    "value": 0.0,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "airAndPollen": [
                {
                    "name": "AirQuality",
                    "value": 23,
                    "category": "Good",
                    "categoryValue": 1,
                    "type": "Ozone"
                },
                {
                    "name": "Grass",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Mold",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Ragweed",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Tree",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "UVIndex",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                }
            ],
            "day": {
                "iconCode": 22,
                "iconPhrase": "Snow",
                "hasPrecipitation": true,
                "precipitationType": "Mixed",
                "precipitationIntensity": "Light",
                "shortPhrase": "Chilly with snow, 2-4 cm",
                "longPhrase": "Chilly with snow, accumulating an additional 2-4 cm",
                "precipitationProbability": 90,
                "thunderstormProbability": 0,
                "rainProbability": 54,
                "snowProbability": 85,
                "iceProbability": 8,
                "wind": {
                    "direction": {
                        "degrees": 36.0,
                        "localizedDescription": "NE"
                    },
                    "speed": {
                        "value": 9.3,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "windGust": {
                    "direction": {
                        "degrees": 70.0,
                        "localizedDescription": "ENE"
                    },
                    "speed": {
                        "value": 25.9,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "totalLiquid": {
                    "value": 4.3,
                    "unit": "mm",
                    "unitType": 3
                },
                "rain": {
                    "value": 0.5,
                    "unit": "mm",
                    "unitType": 3
                },
                "snow": {
                    "value": 2.72,
                    "unit": "cm",
                    "unitType": 4
                },
                "ice": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "hoursOfPrecipitation": 9.0,
                "hoursOfRain": 1.0,
                "hoursOfSnow": 9.0,
                "hoursOfIce": 0.0,
                "cloudCover": 96
            },
            "night": {
                "iconCode": 29,
                "iconPhrase": "Rain and snow",
                "hasPrecipitation": true,
                "precipitationType": "Mixed",
                "precipitationIntensity": "Light",
                "shortPhrase": "Showers of rain and snow",
                "longPhrase": "A couple of showers of rain or snow this evening; otherwise, cloudy; storm total snowfall 1-3 cm",
                "precipitationProbability": 65,
                "thunderstormProbability": 0,
                "rainProbability": 60,
                "snowProbability": 54,
                "iceProbability": 4,
                "wind": {
                    "direction": {
                        "degrees": 16.0,
                        "localizedDescription": "NNE"
                    },
                    "speed": {
                        "value": 16.7,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "windGust": {
                    "direction": {
                        "degrees": 1.0,
                        "localizedDescription": "N"
                    },
                    "speed": {
                        "value": 35.2,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "totalLiquid": {
                    "value": 4.3,
                    "unit": "mm",
                    "unitType": 3
                },
                "rain": {
                    "value": 3.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "snow": {
                    "value": 0.79,
                    "unit": "cm",
                    "unitType": 4
                },
                "ice": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "hoursOfPrecipitation": 4.0,
                "hoursOfRain": 1.0,
                "hoursOfSnow": 3.0,
                "hoursOfIce": 0.0,
                "cloudCover": 94
            },
            "sources": [
                "AccuWeather"
            ]
        },...
    ]
    }
    ```

## <a name="request-hourly-weather-forecast-data"></a>Demander des données de prévisions météorologiques horaires

[L’API Get Hourly Forecast](/rest/api/maps/weather/gethourlyforecast) retourne les prévisions météorologiques détaillées par heure pour les 1, 12, 24 (1 jour), 72 (3 jours), 120 (5 jours) et 240 (10 jours) prochaines heures pour les coordonnées données. L’API retourne des détails tels que la température, l’humidité, le vent, les précipitations et l’index UV.

>[!IMPORTANT]
>Dans le niveau tarifaire S0, vous pouvez demander des prévisions horaires pour les 1, 12, 24 (1 jour) prochaines heures, et les 72 prochaines heures (3 jours). Les niveaux tarifaires Gen1 (S1) et Gen2 offrent tous deux la possibilité de demander les prévisions horaires à 120 heures (5 jours) et à 240 heures (10 jours).

Dans cet exemple, vous allez utiliser [l’API Get Hourly Forecast](/rest/api/maps/weather/gethourlyforecast) pour récupérer les prévisions météorologiques horaires des 12 prochaines heures pour des coordonnées situées à Seattle, WA.

1. Pour créer la demande, dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer), sélectionnez **HTTP Request** (Requête HTTP). Entrez un **Request name** (Nom de demande) pour la demande.

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/weather/forecast/hourly/json?api-version=1.0&query=47.60357,-122.32945&duration=12&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Cliquez sur le bouton bleu **Envoyer**. Le corps de la réponse contient les données des prévisions météorologiques pour les 12 prochaines heures. Par souci de concision, la réponse JSON ci-dessous montre les prévisions pour la première heure.

    ```json
    {
    "forecasts": [
        {
            "date": "2020-10-19T21:00:00+00:00",
            "iconCode": 12,
            "iconPhrase": "Showers",
            "hasPrecipitation": true,
            "precipitationType": "Rain",
            "precipitationIntensity": "Light",
            "isDaylight": true,
            "temperature": {
                "value": 14.7,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperature": {
                "value": 13.3,
                "unit": "C",
                "unitType": 17
            },
            "wetBulbTemperature": {
                "value": 12.0,
                "unit": "C",
                "unitType": 17
            },
            "dewPoint": {
                "value": 9.5,
                "unit": "C",
                "unitType": 17
            },
            "wind": {
                "direction": {
                    "degrees": 242.0,
                    "localizedDescription": "WSW"
                },
                "speed": {
                    "value": 9.3,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "windGust": {
                "speed": {
                    "value": 14.8,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "relativeHumidity": 71,
            "visibility": {
                "value": 9.7,
                "unit": "km",
                "unitType": 6
            },
            "cloudCover": 100,
            "ceiling": {
                "value": 1128.0,
                "unit": "m",
                "unitType": 5
            },
            "uvIndex": 1,
            "uvIndexPhrase": "Low",
            "precipitationProbability": 51,
            "rainProbability": 51,
            "snowProbability": 0,
            "iceProbability": 0,
            "totalLiquid": {
                "value": 0.3,
                "unit": "mm",
                "unitType": 3
            },
            "rain": {
                "value": 0.3,
                "unit": "mm",
                "unitType": 3
            },
            "snow": {
                "value": 0.0,
                "unit": "cm",
                "unitType": 4
            },
            "ice": {
                "value": 0.0,
                "unit": "mm",
                "unitType": 3
            }
        }...
    ]
    }
    ```

## <a name="request-minute-by-minute-weather-forecast-data"></a>Demander des données de prévisions météorologiques minute par minute

 [L’API Get Minute Forecast](/rest/api/maps/weather/getminuteforecast) retourne des prévisions minute par minute pour un emplacement donné pour les 120 prochaines minutes. Les utilisateurs peuvent demander des prévisions météorologiques par intervalles de 1, 5 et 15 minutes. La réponse inclut des détails tels que le type de précipitation (y compris la pluie, la neige ou une combinaison des deux), l’heure de début et la valeur d’intensité des précipitations (dBZ).

Dans cet exemple, vous allez utiliser [l’API Get Minute Forecast](/rest/api/maps/weather/getminuteforecast) pour récupérer les prévisions météorologiques minute par minute pour des coordonnées situées à Seattle, WA. Les prévisions météorologiques sont fournies pour les 120 prochaines minutes. Notre requête demande que les prévisions soient données à des intervalles de 15 minutes, mais vous pouvez ajuster le paramètre sur une valeur de 1 ou 5 minutes.

1. Pour créer la demande, dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer), sélectionnez **HTTP Request** (Requête HTTP). Entrez un **Request name** (Nom de demande) pour la demande.

2. Sélectionnez la méthode HTTP **GET** sous l’onglet Générateur, puis entrez l’URL suivante. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/weather/forecast/minute/json?api-version=1.0&query=47.60357,-122.32945&interval=15&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Cliquez sur le bouton bleu **Envoyer**. Le corps de la réponse contient les données de prévisions météorologiques pour les 120 prochaines minutes, à intervalles de 15 minutes.

    ```json
    {
    "summary": {
        "briefPhrase60": "No precipitation for at least 60 min",
        "shortPhrase": "No precip for 120 min",
        "briefPhrase": "No precipitation for at least 120 min",
        "longPhrase": "No precipitation for at least 120 min",
        "iconCode": 7
    },
    "intervalSummaries": [
        {
            "startMinute": 0,
            "endMinute": 119,
            "totalMinutes": 120,
            "shortPhrase": "No precip for %MINUTE_VALUE min",
            "briefPhrase": "No precipitation for at least %MINUTE_VALUE min",
            "longPhrase": "No precipitation for at least %MINUTE_VALUE min",
            "iconCode": 7
        }
    ],
    "intervals": [
        {
            "startTime": "2020-10-19T20:51:00+00:00",
            "minute": 0,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:06:00+00:00",
            "minute": 15,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:21:00+00:00",
            "minute": 30,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:36:00+00:00",
            "minute": 45,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:51:00+00:00",
            "minute": 60,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:06:00+00:00",
            "minute": 75,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:21:00+00:00",
            "minute": 90,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:36:00+00:00",
            "minute": 105,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        }
        ]
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Concepts des services Météo d’Azure Maps](./weather-services-concepts.md)

> [!div class="nextstepaction"]
> [API REST des services Météo d’Azure Maps](/rest/api/maps/weather)