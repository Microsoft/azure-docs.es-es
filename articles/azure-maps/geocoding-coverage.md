---
title: Cobertura de geocodificación en el servicio Search de Microsoft Azure Maps
description: Vea qué regiones cubre el servicio Search de Azure Maps. Las categorías de geocodificación incluyen puntos de dirección, números de casa, nivel de calle, nivel de ciudad y puntos de interés.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/28/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 1f79add26af40abfa320bc23a3287d8246237701
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037446"
---
# <a name="azure-maps-geocoding-coverage"></a>Cobertura de geocodificación de Azure Maps

El [servicio Search](https://docs.microsoft.com/rest/api/maps/search) de Azure Maps admite la geocodificación, lo que significa que la solicitud de API puede tener términos de búsqueda, como una dirección o el nombre de un lugar, y devuelve el resultado como coordenadas de latitud y longitud. Por ejemplo, [Get Search Address API](https://docs.microsoft.com/rest/api/maps/search/getsearchaddress) de Azure Maps recibe consultas que contienen información de ubicación y devuelve los resultados como coordenadas de latitud y longitud.

Pero el [servicio Search](https://docs.microsoft.com/rest/api/maps/search) de Azure Maps no tiene el mismo nivel de información y precisión en todas las regiones y países. Use este artículo para determinar qué tipo de ubicaciones puede buscar de forma confiable en cada región.

La capacidad de obtener las coordenadas geográficas en un país o una región depende de la cobertura de los datos de las carreteras y la precisión del servicio de geocodificación. Las clasificaciones siguientes se usan para especificar el nivel de geocodificación en cada país o región.

* **Puntos de dirección**: los datos de direcciones se pueden resolver en coordenadas de latitud y longitud dentro de la parcela de la dirección (límite de la propiedad). A menudo se hace referencia a los puntos de dirección como una precisión de "azotea", que es el nivel más alto de precisión disponible para las direcciones.
* **Números de casa**: Las direcciones se interpolan a una coordenada de latitud y longitud de la calle.
* **Nivel de calle**: Las direcciones se resuelven en la coordenada de latitud y longitud de la calle que contiene la dirección. No se puede procesar el número de casa.
* **Nivel de ciudad**: Se admiten nombres de ciudad.

## <a name="americas"></a>América

| País/región                                       | Puntos de dirección | Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Anguila                                            |                 |                |              |      ✓     |          ✓         |
| Antártida                                          |                 |                |              |      ✓     |          ✓         |
| Antigua y Barbuda                                 |                 |                |       ✓      |      ✓     |          ✓         |
| Argentina                                           |       ✓         |        ✓       |       ✓      |      ✓     |          ✓         |
| Aruba                                               |                 |                |              |      ✓     |          ✓         |
| Bahamas                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Barbados                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Belice                                              |                 |                |              |      ✓     |          ✓         |
| Bermudas                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Bolivia                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Bonaire, San Eustaquio y Saba                   |                 |                |              |      ✓     |          ✓         |
| Brasil                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Canadá                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Islas Caimán                                      |                 |                |       ✓      |      ✓     |          ✓         |
| Chile                                               |       ✓         |        ✓       |       ✓      |      ✓     |          ✓         |
| Colombia                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Costa Rica                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Cuba                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Dominica                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Dominicana                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Ecuador                                             |                 |                |       ✓      |      ✓     |          ✓         |
| El Salvador                                         |                 |                |       ✓      |      ✓     |          ✓         |
| Islas Malvinas                                    |                 |                |              |      ✓     |          ✓         |
| Guayana Francesa                                       |                 |                |       ✓      |      ✓     |          ✓         |
| Granada                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Guadalupe                                          |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Guam                                                |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Guatemala                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Guayana                                              |                |             |           |      ✓     |                 |
| Haití                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Honduras                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Jamaica                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Martinica                                          |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| México                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Montserrat                                          |                 |                |              |      ✓     |          ✓         |
| Nicaragua                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Panamá                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Paraguay                                            |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Perú                                                |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Puerto Rico                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| San Bartolomé                                    |                 |                |       ✓      |      ✓     |          ✓         |
| San Cristóbal y Nieves                               |                 |                |       ✓      |      ✓     |          ✓         |
| Santa Lucía                                         |                 |                |              |      ✓     |          ✓         |
| San Martín                                        |                 |                |       ✓      |      ✓     |          ✓         |
| San Pedro y Miquelón                           |                 |                |       ✓      |      ✓     |          ✓         |
| San Vicente y las Granadinas                    |                 |                |              |      ✓     |          ✓         |
| Sint Maarten                                        |                 |                |       ✓      |      ✓     |          ✓         |
| Georgia del Sur e Islas Sandwich del Sur        |                 |                |              |      ✓     |          ✓         |
| Surinam                                            |                 |                |              |      ✓     |          ✓         |
| Trinidad y Tobago                                 |                 |                |       ✓      |      ✓     |          ✓         |
| Islas menores alejadas de los Estados Unidos                |                 |                |              |      ✓     |          ✓         |
| Estados Unidos de América                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Uruguay                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Venezuela                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Islas Vírgenes Británicas                              |                 |                |              |      ✓     |          ✓         |
| EE. UU. Vírgenes de EE. UU.                                 |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |

## <a name="asia-pacific"></a>Asia Pacífico

| País/región                                      | Puntos de dirección |Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Samoa Americana                                      |                 |                |       ✓      |      ✓     |          ✓         |
| Australia                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bangladés                                          |                 |                |              |      ✓     |          ✓         |
| Bután                                              |                 |                |              |      ✓     |          ✓         |
| Territorio Británico del Océano Índico                      |                 |                |              |      ✓     |          ✓         |
| Brunéi                                              |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Camboya                                            |                 |                |              |      ✓     |          ✓         |
| China                                               |                 |                |              |      ✓     |          ✓         |
| Isla de Navidad                                    |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Islas Cocos (Keeling)                             |                 |                |              |      ✓     |          ✓         |
| Comores                                             |                 |                |              |      ✓     |          ✓         |
| Islas Cook                                        |                 |                |              |      ✓     |          ✓         |
| Islas Fiji                                                |                  |                |              |      ✓     |          ✓        |
| Polinesia francesa                                    |                 |                |              |      ✓     |          ✓         |
| Isla Heard e Islas McDonald                   |                 |                |              |      ✓     |          ✓         |
| RAE de Hong Kong                                       |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Indonesia                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| India                                               |        ✓        |        ✓       |       ✓      |      ✓     |                   |
| Japón                                               |                 |                |              |      ✓     |          ✓         |
| Kiribati                                            |                 |                |              |      ✓     |          ✓         |
| Corea                                         |                 |                |              |      ✓     |          ✓         |
| Laos                                                |                 |                |              |      ✓     |          ✓         |
| RAE de Macao                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Malasia                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Micronesia                                          |                 |                |              |      ✓     |          ✓         |
| Mongolia                                            |                 |                |              |      ✓     |          ✓         |
| Nauru                                               |                 |                |              |      ✓     |          ✓         |
| Nepal                                               |                 |                |              |      ✓     |          ✓         |
| Nueva Caledonia                                       |                 |                |              |      ✓     |          ✓         |
| Nueva Zelanda                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Niue                                                |                 |                |              |      ✓     |          ✓         |
| Isla Norfolk                                      |                 |                |              |      ✓     |          ✓         |
| Corea del Norte                                         |                 |                |              |      ✓     |          ✓         |
| Islas Marianas del Norte                            |                 |                |       ✓      |      ✓     |          ✓         |
| Pakistán                                            |                 |                |              |      ✓     |          ✓         |
| Palaos                                               |                 |                |              |      ✓     |          ✓         |
| Papúa Nueva Guinea                                    |                 |                |              |      ✓     |          ✓         |
| Filipinas                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Islas Pitcairn                                            |                 |                |              |      ✓     |          ✓         |
| Samoa                                               |                 |                |              |      ✓     |          ✓         |
| Islas Senkaku/Diaoyutai                                     |        ✓        |                |              |      ✓     |          ✓         |
| Singapur                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Islas Salomón                                     |                 |                |              |      ✓     |          ✓         |
| Islas de Kuriles del Sur                                     |        ✓        |                |              |      ✓     |          ✓         |
| Sri Lanka                                           |                 |                |              |      ✓     |          ✓         |
| Taiwán                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Tailandia                                            |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Tokelau                                             |                 |                |              |      ✓     |          ✓         |
| Tonga                                               |                 |                |              |      ✓     |          ✓         |
| Islas Turcas y Caicos                            |                 |                |              |      ✓     |          ✓         |
| Tuvalu                                              |                 |                |              |      ✓     |          ✓         |
| Vanuatu                                             |                 |                |              |      ✓     |          ✓         |
| Vietnam                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Wallis y Futuna                                   |                 |                |              |      ✓     |          ✓         |

## <a name="europe"></a>Europa

| País/región                                      | Puntos de dirección |Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Albania                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Andorra                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Armenia                                             |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Austria                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Azerbaiyán                                          |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Bélgica                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bosnia y Herzegovina                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bulgaria                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bielorrusia                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Croacia                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Chipre                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| República Checa                                      |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Dinamarca                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Estonia                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Islas Feroe                                       |                 |                |              |      ✓     |          ✓         |
| Finlandia                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Francia                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Georgia                                             |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Alemania                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Gibraltar                                           |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Grecia                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Groenlandia                                           |                 |                |              |      ✓     |          ✓         |
| Guernsey                                            |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Hungría                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Islandia                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Irlanda                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Isla de Man                                         |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Italia                                               |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Jan Mayen                                           |        ✓        |                |              |      ✓     |          ✓         |
| Jersey                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Kazajistán                                          |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Kosovo                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Kirguistán                                          |                 |                |              |      ✓     |          ✓         |
| Letonia                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Liechtenstein                                       |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Lituania                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Luxemburgo                                          |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Macedonia del Norte                                     |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Malta                                               |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Moldavia                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Mónaco                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Montenegro                                          |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Países Bajos                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Noruega                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Polonia                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Portugal                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| +Azores y Madeira                                 |                 |                |       ✓      |      ✓     |          ✓         |
| Rumanía                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Federación Rusa                                  |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| San Marino                                          |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Serbia                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Eslovaquia                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Eslovenia                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| España                                               |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Svalbard                                            |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Suecia                                              |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Suiza                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Tayikistán                                          |                 |                |              |      ✓     |          ✓         |
| Turquía                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Turkmenistán                                        |                 |                |              |      ✓     |          ✓         |
| Ucrania                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Reino Unido                                      |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Uzbekistán                                          |                 |                |              |      ✓     |          ✓         |
| Ciudad del Vaticano                                        |                 |                |       ✓      |      ✓     |          ✓         |

## <a name="middle-east-and-africa"></a>Oriente Medio y África

| País/región                                      | Puntos de dirección |Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Afganistán                                         |                 |                |              |      ✓     |          ✓         |
| Argelia                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Angola                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Bahréin                                             |        ✓        |       ✓        |       ✓      |      ✓     |          ✓         |
| Benín                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Botsuana                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Isla Bouvet                                       |                 |                |              |      ✓     |          ✓         |
| Burkina Faso                                        |                 |                |       ✓      |      ✓     |          ✓         |
| Burundi                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Camerún                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Cabo Verde                                          |                 |                |       ✓      |      ✓     |          ✓         |
| República Centroafricana                            |                 |                |       ✓      |      ✓     |          ✓         |
| Chad                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Congo (RDC)                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Costa de Marfil                                       |                 |                |       ✓      |      ✓     |          ✓         |
| República Democrática del Congo                    |                 |                |       ✓      |      ✓     |          ✓         |
| Yibuti                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Egipto                                               |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| República de Guinea Ecuatorial                      |                 |                |       ✓      |      ✓     |          ✓         |
| Eritrea                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Etiopía                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Territorios Australes Franceses|                        |                |              |      ✓     |          ✓         |
| Gabón                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Gambia                                              |                 |                |              |      ✓     |          ✓         |
| Ghana                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Guinea                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Guinea-Bisáu                                       |                 |                |       ✓      |      ✓     |          ✓         |
| Irán                                                |                 |                |              |      ✓     |          ✓         |
| Iraq                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Israel                                              |        ✓        |       ✓        |              |      ✓     |          ✓         |
| Jordania                                              |        ✓        |       ✓        |       ✓      |      ✓     |          ✓         |
| Kenia                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Kuwait                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Líbano                                             |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Lesoto                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Liberia                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Libia                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Madagascar                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Malaui                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Maldivas                                            |                 |                |              |      ✓     |          ✓         |
| Malí                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Islas Marshall                                    |                 |                |              |      ✓     |          ✓         |
| Mauritania                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Mauricio                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Mayotte                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Marruecos                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Mozambique                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Myanmar                                             |                 |                |              |      ✓     |          ✓         |
| Namibia                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Níger                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Nigeria                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Omán                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Qatar                                               |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Reunión                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Ruanda                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Santa Elena                                        |                 |                |              |      ✓     |          ✓         |
| Arabia Saudí                                        |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Senegal                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Seychelles                                          |                 |                |       ✓      |      ✓     |          ✓         |
| Sierra Leona                                        |                 |                |       ✓      |      ✓     |          ✓         |
| Somalia                                             |                 |                |              |      ✓     |          ✓         |
| Sudáfrica                                        |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Sudán del Sur                                         |                 |                |       ✓      |      ✓     |          ✓         |
| Sudán                                               |                 |                |       ✓      |      ✓     |          ✓         |
| Suazilandia                                           |                 |                |       ✓      |      ✓     |          ✓         |
| Siria                                               |                 |                |              |      ✓     |          ✓         |
| Santo Tomé y Príncipe                               |                 |                |       ✓      |      ✓     |          ✓         |
| Tanzania                                            |                 |                |       ✓      |      ✓     |          ✓         |
| Togo                                                |                 |                |       ✓      |      ✓     |          ✓         |
| Túnez                                             |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Uganda                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Emiratos Árabes Unidos                                |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Yemen                                               |                 |                |              |      ✓     |          ✓         |
| Zambia                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Zimbabue                                            |                 |                |       ✓      |      ✓     |          ✓         |

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la geocodificación de Azure Maps:
> [!div class="nextstepaction"]
> [Servicio Search de Azure Maps](https://docs.microsoft.com/rest/api/maps/search)
