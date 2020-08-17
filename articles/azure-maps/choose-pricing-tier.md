---
title: Elección del plan de tarifa adecuado para Microsoft Azure Maps
description: En este artículo, aprenderá sobre los planes de tarifa que ofrece Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 359c2270f3de269adae13ce976cedeb4248935d2
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285771"
---
# <a name="choose-the-right-pricing-tier-in-azure-maps"></a>Elección del plan de tarifa adecuado de Azure Maps

Azure Maps ofrece dos planes de tarifa: S0 y S1. El propósito de este artículo es ayudarle a elegir el plan de tarifa adecuado para satisfacer sus necesidades. Para elegir el plan de tarifa adecuado, plantéese las dos preguntas siguientes.

## <a name="how-many-concurrent-users-do-i-plan-to-support"></a>¿Cuántos usuarios simultáneos tiene previsto admitir?

Los planes de tarifa S0 y S1 manipulan diferentes volúmenes de rendimiento de datos. El plan de tarifa S0 manipula hasta **50 consultas por segundo**. Por el contrario, el plan de tarifa S1 administra **más de 50 consultas por segundo**.

## <a name="what-geospatial-capabilities-do-i-plan-to-use"></a>¿Qué funcionalidades geoespaciales tiene previsto usar?

Si las API geoespaciales básicas satisfacen los requisitos del servicio, elija el plan de tarifa S0. Si desea funcionalidades más avanzadas para su aplicación, considere la posibilidad de optar por el plan de tarifa S1. Las funcionalidades avanzadas son: imágenes aéreas e híbridas, obtención del alcance de la ruta y geocodificación por lotes. Para seleccionar el plan de tarifa más apropiado para la aplicación, consulte la tabla con las **funcionalidades del plan de tarifa**  a continuación:

### <a name="pricing-tier-capabilities"></a>Funcionalidades de los planes de tarifa

| Capacidad                              |        S0           |  S1      |
|-----------------------------------------|:-------------------:|:--------:|
| Representación de mapas                              | ✓                   | ✓       |
| Imágenes de satélite                       |                     | ✓        |
| Search                                  | ✓                    | ✓        |
| Búsqueda por lotes                            |                     | ✓        |
| Enrutar                                   | ✓                    |✓        |
| Enrutamiento por lotes                            |                    | ✓        |
| Enrutamiento de matriz                          |                     | ✓        |
| Alcance de ruta (isocronas)                |                     | ✓        |
| Tráfico                                |✓                    |✓        |
| Zona horaria                               |✓                    |✓        |
| Geolocalización (versión preliminar)                    |✓                   |✓        |
| Spatial Operations                        |                    |✓        |
| Geovalla                                |                    |✓        |
| Datos de Azure Maps (versión preliminar)                |                     | ✓        |
| Movilidad (versión preliminar)                       |                     | ✓        |
| Meteorología (versión preliminar)                        |✓                    |✓        |

Tenga en cuenta también estos otros aspectos:

* ¿Qué tipo de empresa tiene?
* ¿Es crítica la aplicación?

### <a name="pricing-tier-targeted-customers"></a>Clientes objetivo de los planes de tarifa

Consulte la tabla **Clientes objetivo de los planes de tarifa** para obtener una visión general de los planes de tarifa S0 y S1. Para más información, consulte [Precios de Azure Maps](https://azure.microsoft.com/pricing/details/azure-maps/). 

| Plan de tarifa  |     Clientes objetivo                                                                |
|-----------------|:-----------------------------------------------------------------------------------------|
| S0            |    El plan de tarifa S0 funciona para las aplicaciones en todas las fases de producción, desde el desarrollo de la prueba de concepto y la primera fase de pruebas hasta la puesta en producción e implementación de la aplicación. Sin embargo, este nivel está diseñado para el desarrollo a pequeña escala, para clientes con pocos usuarios simultáneos o ambos. 
| S1            |    El plan de tarifa S1 está orientado a clientes con aplicaciones empresariales a gran escala, aplicaciones críticas para la empresa o grandes volúmenes de usuarios simultáneos. También es para aquellos clientes que requieren servicios geoespaciales avanzados.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo ver y cambiar los planes de tarifa:

> [!div class="nextstepaction"]
> [Administración de un plan de tarifa](how-to-manage-pricing-tier.md)
