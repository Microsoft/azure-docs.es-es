---
title: Cobertura de geocodificación en Azure Maps | Microsoft Docs
description: Información sobre la cobertura de geocodificación en Azure Maps
author: walsehgal
ms.author: v-musehg
ms.date: 03/22/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: a5e5f4ab286289e223a2fe10ff8cf45f43309f04
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785938"
---
# <a name="azure-maps-geocoding-coverage"></a>Cobertura de geocodificación de Azure Maps

Cuando busca una ubicación con Azure Maps, el servicio de búsqueda toma los términos de su búsqueda y devuelve las coordenadas de latitud y longitud mediante un proceso denominado geocodificación. Sin embargo, Maps no tiene el mismo nivel de información y precisión en todas las regiones y países. Use este artículo para determinar qué tipo de ubicaciones puede buscar de forma confiable en cada región. 

La capacidad de código geográfico en un país o región depende de la cobertura de datos de la carretera y la precisión de codificación geográfica del servicio de geocodificación. Se usan las siguientes clasificaciones de especificar el nivel de compatibilidad de geocodificación en cada país o región.
* **Puntos de dirección**: Los datos de direcciones se pueden resolver en una coordenada de latitud y longitud dentro de la parcela de la dirección (límite de la propiedad). Este nivel de precisión, a veces llamada de azotea, es el mayor disponible para las direcciones. 
* **Números de casa**: Las direcciones se interpolan a una coordenada de latitud y longitud de la calle.
* **Nivel de calle**: Las direcciones se resuelven en la coordenada de latitud y longitud de la calle que contiene la dirección. No se puede procesar el número de casa.
* **Nivel de ciudad**: Se admiten nombres de ciudad.

## <a name="americas"></a>América

| País o región                                       | Puntos de dirección | Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Anguila                                            |                 |                |              |      ✓     |          ✓         |
| Antártida                                          |                 |                |              |      ✓     |          ✓         |
| Antigua y Barbuda                                 |                 |                |       ✓      |      ✓     |          ✓         |
| Argentina                                           |       ✓         |        ✓       |       ✓      |      ✓     |          ✓         |
| Aruba                                               |                 |                |              |      ✓     |          ✓         |
| Las Bahamas                                             |                 |                |       ✓      |      ✓     |          ✓         |
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
| Datos de agencias y Islas Vírgenes                                 |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |

## <a name="asia-pacific"></a>Asia Pacífico

| País o región                                      | Puntos de dirección |Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Samoa Americana                                      |                 |                |       ✓      |      ✓     |          ✓         |
| Australia                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bangladés                                          |                 |                |              |      ✓     |          ✓         |
| Bhután                                              |                 |                |              |      ✓     |          ✓         |
| Territorio Británico del Océano Índico                      |                 |                |              |      ✓     |          ✓         |
| Brunei                                              |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Camboya                                            |                 |                |              |      ✓     |          ✓         |
| China                                               |                 |                |              |      ✓     |          ✓         |
| Isla Christmas                                    |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Islas Cocos                             |                 |                |              |      ✓     |          ✓         |
| Comores                                             |                 |                |              |      ✓     |          ✓         |
| Islas Cook                                        |                 |                |              |      ✓     |          ✓         |
| Fiji                                                |                  |                |              |      ✓     |          ✓        |
| Polinesia Francesa                                    |                 |                |              |      ✓     |          ✓         |
| Islas Heard y McDonald                   |                 |                |              |      ✓     |          ✓         |
| Hong Kong RAE                                       |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Indonesia                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| India                                               |        ✓        |        ✓       |       ✓      |      ✓     |                   |
| Japón                                               |                 |                |              |      ✓     |          ✓         |
| Kiribati                                            |                 |                |              |      ✓     |          ✓         |
| Corea                                         |                 |                |              |      ✓     |          ✓         |
| Laos                                                |                 |                |              |      ✓     |          ✓         |
| Macao RAE                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
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
| Paracelso                                     |                 |                |              |      ✓     |                    |
| Filipinas                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Pitcairn                                            |                 |                |              |      ✓     |          ✓         |
| Estado Independiente de Samoa                                               |                 |                |              |      ✓     |          ✓         |
| Islas Senkaku/Diaoyutai                                     |        ✓        |                |              |      ✓     |          ✓         |
| Singapur                                           |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Islas Salomón                                     |                 |                |              |      ✓     |          ✓         |
| Islas de Kuriles del Sur                                     |        ✓        |                |              |      ✓     |          ✓         |
| Islas Spratly                                     |                 |                |              |      ✓     |                    |
| Sri Lanka                                           |                 |                |              |      ✓     |          ✓         |
| Taiwán                                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Tailandia                                            |        ✓        |                |       ✓      |      ✓     |          ✓         |
| Tokelau                                             |                 |                |              |      ✓     |          ✓         |
| Tonga                                               |                 |                |              |      ✓     |          ✓         |
| Islas Turks y Caicos                            |                 |                |              |      ✓     |          ✓         |
| Tuvalu                                              |                 |                |              |      ✓     |          ✓         |
| Vanuatu                                             |                 |                |              |      ✓     |          ✓         |
| Vietnam                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Wallis y Futuna                                   |                 |                |              |      ✓     |          ✓         |

## <a name="europe"></a>Europa

| País o región                                      | Puntos de dirección |Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Albania                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Andorra                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Armenia                                             |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Austria                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Azerbaiyán                                          |        ✓        |        ✓       |              |      ✓     |          ✓         |
| Bélgica                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bosnia y Herzegovina                              |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Bulgaria                                            |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
| Belarús                                             |                 |        ✓       |       ✓      |      ✓     |          ✓         |
| Croacia (Hrvatska)                                             |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
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
| Guernesey                                            |                 |        ✓       |       ✓      |      ✓     |          ✓         |
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
| Países bajos                                         |        ✓        |        ✓       |       ✓      |      ✓     |          ✓         |
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

| País o región                                      | Puntos de dirección |Números de casa | Nivel de calle | Nivel de ciudad | Puntos de interés |
|-----------------------------------------------------|:---------------:|:--------------:|:------------:|:----------:|:------------------:|
| Afganistán                                         |                 |                |              |      ✓     |          ✓         |
| Argelia                                             |                 |                |       ✓      |      ✓     |          ✓         |
| Angola                                              |                 |                |       ✓      |      ✓     |          ✓         |
| Reino de Baréin                                             |        ✓        |       ✓        |       ✓      |      ✓     |          ✓         |
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
| Côte d'Ivoire                                       |                 |                |       ✓      |      ✓     |          ✓         |
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
| Guinea-Bissau                                       |                 |                |       ✓      |      ✓     |          ✓         |
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
| Catar                                               |        ✓        |                |       ✓      |      ✓     |          ✓         |
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

Para más información acerca de la geocodificación de Azure Maps, consulte las páginas de referencia de [Búsqueda](https://docs.microsoft.com/rest/api/maps/search).

Conozca más información acerca de las [áreas de cobertura del servicio de tráfico de Maps](traffic-coverage.md). 

