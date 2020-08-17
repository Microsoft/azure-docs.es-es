---
title: Azure Maps como origen de Event Grid
description: Describe las propiedades y los esquemas que se proporcionan para los eventos de Azure Maps con Azure Event Grid
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 4203bdf5222278b698d656835afebd9769557303
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461993"
---
# <a name="azure-maps-as-an-event-grid-source"></a>Azure Maps como origen de Event Grid

En este artículo se proporcionan las propiedades y los esquemas de los eventos de Azure Maps. Para una introducción a los esquemas de eventos, consulte [Esquema de eventos de Azure Event Grid](./event-schema.md). También proporciona una lista de inicios rápidos y tutoriales para usar Azure Maps como un origen de eventos.

## <a name="event-grid-event-schema"></a>Esquema de eventos de Event Grid

### <a name="available-event-types"></a>Tipos de eventos disponibles

Una cuenta de Azure Maps emite los siguientes tipos de eventos:

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.Maps.GeofenceEntered | Se genera cuando se mueven las coordenadas recibidas desde fuera de una determinada geovalla a dentro |
| Microsoft.Maps.GeofenceExited | Se genera cuando se mueven las coordenadas recibidas desde dentro de una determinada geovalla a fuera |
| Microsoft.Maps.GeofenceResult | Se genera cada vez que una consulta de geovalla devuelve un resultado, independientemente del estado |

### <a name="event-examples"></a>Ejemplos de eventos

En el ejemplo siguiente, se muestra el esquema de un evento **GeofenceEntered**

```JSON
{   
   "id":"7f8446e2-1ac7-4234-8425-303726ea3981", 
   "topic":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}", 
   "subject":"/spatial/geofence/udid/{udid}/id/{eventId}", 
   "data":{   
      "geometries":[   
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":-999.0, 
            "nearestLat":47.618786, 
            "nearestLon":-122.132151 
         } 
      ], 
      "expiredGeofenceGeometryId":[   
      ], 
      "invalidPeriodGeofenceGeometryId":[   
      ] 
   }, 
   "eventType":"Microsoft.Maps.GeofenceEntered", 
   "eventTime":"2018-11-08T00:54:17.6408601Z", 
   "metadataVersion":"1", 
   "dataVersion":"1.0" 
}
```

En el ejemplo siguiente, se muestra el esquema de un evento **GeofenceResult** 

```JSON
{   
   "id":"451675de-a67d-4929-876c-5c2bf0b2c000", 
   "topic":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}", 
   "subject":"/spatial/geofence/udid/{udid}/id/{eventId}", 
   "data":{   
      "geometries":[   
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"1", 
            "distance":999.0, 
            "nearestLat":47.609833, 
            "nearestLon":-122.148274 
         }, 
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":999.0, 
            "nearestLat":47.621954, 
            "nearestLon":-122.131841 
         } 
      ], 
      "expiredGeofenceGeometryId":[   
      ], 
      "invalidPeriodGeofenceGeometryId":[   
      ] 
   }, 
   "eventType":"Microsoft.Maps.GeofenceResult", 
   "eventTime":"2018-11-08T00:52:08.0954283Z", 
   "metadataVersion":"1", 
   "dataVersion":"1.0" 
}
```

### <a name="event-properties"></a>Propiedades de evento

Un evento tiene los siguientes datos de nivel superior:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| topic | string | Ruta de acceso completa a los recursos del origen del evento. En este campo no se puede escribir. Event Grid proporciona este valor. |
| subject | string | Ruta al asunto del evento definida por el anunciante. |
| eventType | string | Uno de los tipos de eventos registrados para este origen de eventos. |
| eventTime | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| id | string | Identificador único para el evento |
| datos | object | Datos de eventos de geovalla. |
| dataVersion | string | Versión del esquema del objeto de datos. El publicador define la versión del esquema. |
| metadataVersion | string | Versión del esquema de los metadatos del evento. Event Grid define el esquema de las propiedades de nivel superior. Event Grid proporciona este valor. |

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| apiCategory | string | Categoría de API del evento. |
| apiName | string | Nombre de API del evento. |
| issues | object | Se enumeran los problemas que se encontraron durante el procesamiento. Si se devuelve algún problema, no se devolverán geometrías con la respuesta. |
| responseCode | number | Código de respuesta HTTP |
| geometries | object | Muestra las geometrías de valla que contienen la posición de la coordenada o superpone searchBuffer en torno a esa posición. |

Se devuelve el objeto de error cuando se produce un error en la API de Maps. El objeto de error tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| error | ErrorDetails |Se devuelve este objeto cuando se produce un error en la API de Maps  |

Se devuelve el objeto ErrorDetails cuando se produce un error en la API de Maps. El objeto ErrorDetails tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| código | string | El código de estado HTTP. |
| message | string | Si está disponible, una descripción legible del error. |
| innererror | InnerError | Si está disponible, un objeto que contiene información específica del servicio sobre el error. |

InnerError es un objeto que contiene información específica del servicio sobre el error. El objeto InnerError tiene las siguientes propiedades: 

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| código | string | El mensaje de error. |

El objeto "geometries" muestra los identificadores de geometría de las geovallas que han expirado en relación con la hora que especificó el usuario en la solicitud. El objeto "geometries" tiene elementos de geometría con las siguientes propiedades: 

| Propiedad | Tipo | Descripción |
|:-------- |:---- |:----------- |
| deviceid | string | Identificador del dispositivo. |
| distancia | string | <p>Distancia entre la coordenada y el borde más cercano de la geovalla. Un número positivo significa que la coordenada está fuera de la geovalla. Si la coordenada está fuera de la geovalla, pero es superior al valor de la distancia de searchBuffer al borde más cercano de la geovalla, el valor es 999. Un número negativo significa que la coordenada está dentro de la geovalla. Si la coordenada está dentro del polígono, pero es superior al valor de la distancia de searchBuffer al borde más cercano de la geovalla, el valor es -999. Un valor de 999 significa que hay una gran confianza en que la coordenada está fuera de la geovalla. Un valor de 999 significa que hay una gran confianza en que la coordenada está dentro de la geovalla.<p> |
| geometryid |string | El identificador único identifica la geometría de la geovalla. |
| nearestlat | number | Latitud del punto más cercano de la geometría. |
| nearestlon | number | Longitud del punto más cercano de la geometría. |
| udId | string | El identificador único devuelto desde el servicio de carga del usuario al cargar una geovalla. No se incluirá en la API POST de geovalla. |

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| expiredGeofenceGeometryId | string[] | Muestra el identificador de geometría de la geovalla que ha expirado en relación con la hora que especificó el usuario en la solicitud. |
| geometries | geometries[] |Muestra las geometrías de valla que contienen la posición de la coordenada o superpone searchBuffer en torno a esa posición. |
| invalidPeriodGeofenceGeometryId | string[]  | Muestra el identificador de geometría de la geovalla que está en un período no válido en relación con la hora que especificó el usuario en la solicitud. |
| isEventPublished | boolean | El valor es "true" si se publica al menos un evento en el suscriptor de eventos de Azure Maps y "false" en caso contrario. |

## <a name="tutorials-and-how-tos"></a>Tutoriales y procedimientos
|Título  |Descripción  |
|---------|---------|
| [Reaccionar a eventos de Azure Maps mediante Event Grid](../azure-maps/azure-maps-event-grid-integration.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Información general de la integración de Azure Maps con Event Grid. |
| [Tutorial: configuración de una geovalla](../azure-maps/tutorial-geofence.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Este tutorial le guiará por los pasos básicos para configurar la geovalla con Azure Maps. Usará Azure Event Grid para transmitir los resultados de la geovalla y establecer una notificación basada en los resultados de esa geovalla. |

## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a Azure Event Grid, consulte [Introducción a Azure Event Grid](overview.md).
* Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
