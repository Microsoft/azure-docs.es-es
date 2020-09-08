---
title: Estructuras de datos del servicio Mobility en Azure Maps | Microsoft Azure Maps
description: Aprenda cómo se organizan los datos en áreas metropolitanas en los Mobility Services de Azure Maps. Vea qué campos almacenan información acerca de las líneas y paradas de transporte público.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/05/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 02d89226bd2df4bfe5d11897199c50c702e7bc1c
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88033230"
---
# <a name="data-structures-in-azure-maps-mobility-service"></a>Estructuras de datos de Mobility Service en Azure Maps

En este artículo se presenta el concepto de área metropolitana en [Azure Maps Mobility Service](https://aka.ms/AzureMapsMobilityService). Se describen algunos de los campos comunes que se devuelven cuando se consultan las líneas y paradas de transporte público en este servicio. Se recomienda leer este artículo antes de comenzar a desarrollar con las API de Mobility Service.

## <a name="metro-area"></a>Área metropolitana

Los datos de Mobility Service se agrupan en áreas metropolitanas admitidas. Las áreas metropolitanas no siguen los límites de la ciudad. Un área metropolitana puede contener varias ciudades, ciudades con una gran densidad de población y ciudades circundantes. De hecho, un país o región puede ser un área metropolitana. 

`metroID` es un identificador de área metropolitana que se puede usar para llamar a [Get Metro Area Info API](https://aka.ms/AzureMapsMobilityMetroAreaInfo). Use "Get Metro" API de Azure Maps para solicitar tipos de transporte público, empresas de transporte público, alertas activas y detalles adicionales del área metropolitana elegida. También puede solicitar las áreas metropolitanas admitidas y identificadores metroID. Los identificadores de área metropolitana están sujetos a cambios.

**metroID:** 522   **Nombre:** Seattle-Tacoma-Bellevue

![Área metropolitana de Seattle](./media/mobility-service-data-structure/seattle-metro.png)

## <a name="stop-ids"></a>Identificadores de parada

Se puede hacer referencia a las paradas de transporte público por dos tipos de identificador: el identificador de [General Transit Feed Specification (GFTS)](http://gtfs.org/) y el identificador de parada de Azure Maps. El identificador de GFTS se conoce como stopKey y el identificador de parada de Azure Maps como stopID. Cuando se hace referencia con frecuencia a paradas de transporte público, se recomienda usar el identificador de parada de Azure Maps. stopID es más estable y es probable que permanezca igual siempre que exista la parada física. El identificador de parada de GTFS se actualiza con más frecuencia. Por ejemplo, se puede actualizar según la solicitud del proveedor de GTFS o cuando se lanza una nueva versión de GTFS. Aunque no se haya producido ningún cambio en la parada física, el identificador de parada de GTFS puede cambiar.

Para empezar, puede solicitar las paradas de transporte público próximas mediante [Get Nearby Transit API](https://aka.ms/AzureMapsMobilityNearbyTransit).

## <a name="line-groups-and-lines"></a>Líneas y grupos de líneas

Mobility Service usa un modelo de datos paralelo para las líneas y los grupos de líneas. Este modelo se usa para contrarrestar mejor los cambios heredados de las rutas y los datos de viajes de [GTFS](http://gtfs.org/).


### <a name="line-groups"></a>Grupos de líneas

Un grupo de líneas es una entidad que reúne todas las líneas que lógicamente forman parte del mismo grupo. Normalmente, un grupo de líneas contiene dos líneas, una que va desde el punto A al punto B y la otra que vuelve del punto B al punto A. Ambas pertenecerían a la misma empresa de transporte público y tienen el mismo número de línea. Sin embargo, puede haber casos en los que un grupo de líneas tenga más de dos o solo una línea.


### <a name="lines"></a>Líneas

Como se explicó anteriormente, cada grupo de líneas se compone de un conjunto de líneas. Cada grupo de líneas se compone de dos líneas y cada línea describe una dirección.  Sin embargo, hay casos en los que el grupo de líneas contiene un número mayor de líneas. Por ejemplo, hay una línea que a veces se desvía por un determinado barrio y otras no. En ambos casos, funciona con el mismo número de línea. Además, un grupo de líneas puede estar formado por una sola línea. Una línea circular con una dirección única es un grupo de líneas con una línea.

Para empezar, puede solicitar grupos de líneas mediante [Get Transit Line API](https://aka.ms/AzureMapsMobilityTransitLine).


## <a name="next-steps"></a>Pasos siguientes

Aprenda a solicitar datos de tránsito mediante Mobility Service:

> [!div class="nextstepaction"]
> [Cómo solicitar datos de tránsito](how-to-request-transit-data.md)

Aprenda a solicitar datos en tiempo real mediante Mobility Service:

> [!div class="nextstepaction"]
> [Cómo solicitar datos en tiempo real](how-to-request-real-time-data.md)

Explore la documentación de la API Mobility Service de Azure Maps.

> [!div class="nextstepaction"]
> [Documentación de Mobility Service API](https://aka.ms/AzureMapsMobilityService)
