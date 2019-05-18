---
title: Procedimientos recomendados para elegir un id. de serie temporal en la versión preliminar de Azure Time Series Insights | Microsoft Docs
description: Descripción de los procedimientos recomendados al elegir un id. de serie temporal en la versión preliminar de Azure Time Series Insights.
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 05/08/2019
ms.custom: seodec18
ms.openlocfilehash: 4b2f538831ee9410eaf1a2d272f01fd30a9236e6
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65519434"
---
# <a name="best-practices-for-choosing-a-time-series-id"></a>Procedimientos recomendados al elegir un id. de serie temporal

En este artículo se detalla la clave de partición de la versión preliminar de Azure Time Series Insights, el id. de serie temporal y los procedimientos recomendados para elegir uno.

## <a name="choose-a-time-series-id"></a>Elección de un identificador de Time Series

Elegir un id. de serie temporal es como elegir una clave de partición para una base de datos. Es una decisión importante que debe tomarse en el tiempo de diseño. Tenga en cuenta que no puede actualizar un entorno de la versión preliminar de Time Series Insights existente para usar un id. de serie temporal distinto. En otras palabras, cuando se crea un entorno con un id. de serie temporal, la directiva es una propiedad inmutable que no se puede cambiar.

> [!IMPORTANT]
> El id. de serie temporal distingue entre mayúsculas y minúsculas y es inmutable (esto es, no se puede cambiar después de configurarse).

Teniendo eso en cuenta, es fundamental seleccionar un id. de serie temporal adecuado. Cuando seleccione un id. de serie temporal, tenga en cuenta estos procedimientos recomendados:

* Elija un nombre de propiedad que tenga una amplia gama de valores e incluso patrones de acceso. Se recomienda tener una clave de partición con muchos valores distintos (por ejemplo, centenares o miles). Para muchos clientes, esto será algo como DeviceID o SensorID en su JSON.
* El id. de serie temporal debe ser único en el nivel de nodo hoja de su [Modelo de serie temporal](./time-series-insights-update-tsm.md).
* Una cadena de caracteres para el nombre de propiedad del id. de serie temporal puede tener hasta 128 caracteres, y los valores de propiedad del id. de serie temporal pueden tener hasta 1024 caracteres.
* Si faltan algunos valores únicos de la propiedad del id. de serie temporal, se tratan como valores NULL, que participan en la restricción de exclusividad.

Además, puede seleccionar hasta *tres* (3) propiedades clave como id. de serie temporal.

  > [!NOTE]
  > Recuerde que sus *tres* (3) propiedades clave deben ser cadenas.

En los siguientes escenarios se describe la opción para seleccionar más de una propiedad clave como id. de serie temporal:  

### <a name="scenario-one"></a>Escenario uno

* Tiene varios activos heredados, cada uno con una clave única.
* Por ejemplo, la propiedad *deviceId* se encarga de identificar de manera única un tipo de activo y la propiedad única *objectId* se encarga de otro tipo. Ninguno de los dos tipos de activos contiene la propiedad única del otro. En este ejemplo, debe seleccionar dos claves, deviceId y objectId, como claves únicas.
* Como se aceptan valores de tipo NULL, si falta una propiedad en la carga del evento, esta cuenta como un valor de `null`. Esta es también la forma adecuada de manejar el envío de datos a dos orígenes de eventos diferentes donde los datos de cada origen de eventos tienen un id. de serie temporal único.

### <a name="scenario-two"></a>Escenario dos

* Necesita que varias propiedades sean únicas dentro del mismo tipo de activos. 
* Por ejemplo, digamos que es un fabricante de edificios inteligentes e implementa sensores en cada habitación. En cada habitación tendrá normalmente los mismos valores para *sensorId*, como *sensor1*, *sensor2* y *sensor3*.
* Además, el edificio tiene números de pisos y habitaciones que se superponen en los sitios de la propiedad *flrRm*, que tienen valores como *1a*, *2b*, *3a*, etc.
* Finalmente, tiene la propiedad *location*, que contiene valores como *Redmond*, *Barcelona* y *Tokio*. Para que los valores sean únicos, debe designar las tres propiedades siguientes como sus claves de id. de serie temporal: *sensorId*, *flrRm* y *location*.

## <a name="next-steps"></a>Pasos siguientes

* Consiga más información sobre el [modelado de datos](./time-series-insights-update-tsm.md).

* Planifique el [entorno de Azure Time Series Insights (versión preliminar)](./time-series-insights-update-plan.md).