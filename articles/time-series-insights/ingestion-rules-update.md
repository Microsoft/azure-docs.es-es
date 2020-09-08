---
title: Próximos cambios en las reglas de ingesta y simplificación en Azure Time Series Insights Gen2 | Microsoft Docs
description: Cambios en la regla de ingesta
ms.service: time-series-insights
services: time-series-insights
author: lyrana
ms.author: lyhughes
manager: dpalled
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/12/2020
ms.custom: lyhughes
ms.openlocfilehash: cdf90614e1f766fc37e04a1416081bd35e19b74e
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88168208"
---
# <a name="upcoming-changes-to-the-json-flattening-and-escaping-rules-for-new-environments"></a>Próximos cambios en las reglas de aplanamiento y escape de JSON para los entornos nuevos

**Estos cambios solo se aplicarán a entornos de Azure Time Series Insights Gen2 *recién creados*. Estos cambios no se aplican a entornos de Gen1.**

El entorno de Azure Time Series Insights Gen2 crea dinámicamente las columnas de almacenamiento, siguiendo un conjunto determinado de convenciones de nomenclatura. Cuando se ingiere un evento, se aplica un conjunto de reglas a la carga y los nombres de propiedad de JSON. Los cambios en la forma en que se aplanan y se almacenan los datos JSON entrarán en vigor en los entornos nuevos de Azure Time Series Insights Gen2 en julio de 2020. Este cambio afecta su entorno en los siguientes casos:

* Si la carga de JSON contiene objetos anidados
* Si la carga de JSON contiene matrices
* Si usa cualquiera de los cuatro caracteres especiales siguientes en un nombre de propiedad JSON: [ \ . '
* Si una o varias de sus propiedades de identificador de TS se encuentran dentro de un objeto anidado.

Si crea un entorno nuevo y uno o varios de los casos anteriores se aplican a la carga del evento, verá que los datos se han aplanado y almacenado de forma diferente. A continuación se muestra un resumen de los cambios:

| Regla actual | Nueva regla | Ejemplo de JSON | Nombre de columna anterior | Nombre de columna nuevo
|---|---| ---| ---|  ---|
| El objeto JSON anidado se aplana con un carácter de subrayado como el delineador |El objeto JSON anidado se aplana con un punto como el delineador  | ``{"series" : { "value" : 19.338 }}`` | `series_value_double` |`series.value_double` |
| Los caracteres especiales no incluyen caracteres de escape | Nombres de propiedad JSON que incluyen los caracteres especiales. [  \ y ' se convierten en caracteres de escape con [' y ']. Dentro de [' y '], hay un escape adicional de comillas simples y barras diagonales inversas. Una comilla simple se escribirá como \' y una barra diagonal inversa, como \\\  | ```"Foo's Law Value": "17.139999389648"``` | `Foo's Law Value_double` | `['Foo\'s Law Value']_double` |
| Las matrices de primitivas se almacenan como una cadena | Las matrices de tipos primitivos se almacenan como un tipo dinámico  | `"values": [154, 149, 147]` | `values_string`  | `values_dynamic` |
Las matrices de objetos siempre están aplanadas, lo que genera varios eventos | Si los objetos dentro de una matriz no tienen las propiedades de marca de tiempo ni el identificador de TS, la matriz de objetos se almacena completamente como un tipo dinámico | `"values": [{"foo" : 140}, {"bar" : 149}]` | `values_foo_long | values_bar_long` | `values_dynamic` |

## <a name="recommended-changes-for-new-environments"></a>Cambios recomendados para los entornos nuevos

### <a name="if-your-ts-id-andor-timestamp-property-is-nested-within-an-object"></a>Si la propiedad de marca de tiempo o de identificador de TS está anidada dentro de un objeto:

* Las implementaciones nuevas deberán coincidir con las reglas de ingesta nuevas. Por ejemplo, si el identificador de TS es `telemetry_tagId`, deberá actualizar las plantillas de Resource Manager (ARM) o los scripts de implementación automatizados para configurar `telemetry.tagId` como el identificador de TS del entorno. Este cambio también es necesario para las marcas de tiempo de origen del evento en el objeto JSON anidado.

### <a name="if-your-payload-contains-nested-json-or-special-characters-and-you-automate-authoring-time-series-model-variable-expressions"></a>Si la carga contiene un objeto JSON anidado o caracteres especiales y se automatiza la creación de expresiones variables del [modelo de serie temporal](.\time-series-insights-update-tsm.md)

* Actualice el código de cliente que ejecuta [TypesBatchPut](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch#typesbatchput) para que coincida con las reglas de ingesta nuevas. Por ejemplo, una [expresión de serie temporal](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) de `"value": {"tsx": "$event.series_value.Double"}` se debe actualizar a una de las opciones siguientes:
  * `"value": {"tsx": "$event.series.value.Double"}`
  * `"value": {"tsx": "$event['series']['value'].Double"}`

## <a name="next-steps"></a>Pasos siguientes

* Lea [Almacenamiento y entrada de Azure Time Series Insights Gen2](./time-series-insights-update-storage-ingress.md).

* Lea más datos sobre cómo realizar consultas de datos mediante las [API de consulta de serie temporal](./concepts-query-overview.md).

* Lea más información sobre la [nueva sintaxis de Time Series Expression](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax).
