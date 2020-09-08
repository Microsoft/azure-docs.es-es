---
author: baanders
description: archivo de inclusión para opciones de filtro de ruta de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 8/3/2020
ms.author: baanders
ms.openlocfilehash: c5b78d80b1aa958af50b81dc152b4a746ce85f70
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87902217"
---
| Nombre de filtro | Descripción | Esquema de texto de filtro | Valores admitidos | 
| --- | --- | --- | --- |
| True o False | Permite crear una ruta sin filtrado o deshabilitar una ruta para que no se envíen eventos. | `<true/false>` | `true`: la ruta está habilitada sin filtrado. <br> `false`: la ruta está deshabilitada. |
| Tipo | [Tipo de evento](../articles/digital-twins/concepts-route-events.md#types-of-event-messages) que fluye a través de la instancia de Digital Twins. | `type = '<eventType>'` | Estos son los posibles valores de tipo de evento: <br>`Microsoft.DigitalTwins.Twin.Create` <br> `Microsoft.DigitalTwins.Twin.Delete` <br> `Microsoft.DigitalTwins.Twin.Update`<br>`Microsoft.DigitalTwins.Relationship.Create`<br>`Microsoft.DigitalTwins.Relationship.Update`<br> `Microsoft.DigitalTwins.Relationship.Delete` <br> `microsoft.iot.telemetry`  |
| Source | Nombre de la instancia de Azure Digital Twins. | `source = '<hostname>'`| Estos son los posibles valores de nombre de host: <br> **Para notificaciones**: `<yourDigitalTwinInstance>.<yourRegion>.azuredigitaltwins.net` <br> **Para telemetría**: `<yourDigitalTwinInstance>.<yourRegion>.azuredigitaltwins.net/digitaltwins/<twinId>`|
| Asunto | Descripción del evento en el contexto del origen del evento anterior. | `subject = '<subject>'` | Estos son los posibles valores para el asunto: <br>**Para notificaciones**: el asunto es `<twinid>` <br> o un formato de URI para los asuntos, que se identifican de forma única mediante varias partes o id.:<br>`<twinid>/relationships/<relationshipid>`<br> **Para telemetría**: el asunto es la ruta de acceso del componente (si la telemetría se emite desde un componente gemelo), como `comp1.comp2`. Si no se emite la telemetría desde un componente, el campo de asunto está vacío. |
| Esquema de datos | Id. del modelo de DTDL. | `dataschema = '<model-dtmi-ID>'` | **Para telemetría**: el esquema de datos es el identificador del modelo del gemelo o el componente que emite la telemetría. Por ejemplo: `dtmi:example:com:floor4;2` <br>**Para notificaciones**: no se admite el esquema de datos.|
| Tipo de contenido | Tipo de contenido del valor de datos. | `datacontenttype = '<contentType>'` | El tipo de contenido es `application/json`. |
| Versión de especificación | Versión del esquema de evento que se usa. | `specversion = '<version>'` | La versión debe ser `1.0`. Esto indica la versión de esquema de CloudEvents es la 1.0. |

También es posible combinar filtros mediante las operaciones siguientes:

| Nombre de filtro | Esquema de texto de filtro | Ejemplo | 
| --- | --- | --- |
| Y/O | `<filter1> AND <filter2>` | `type != 'microsoft.iot.telemetry' AND datacontenttype = 'application/json'` |
| Y/O | `<filter1> OR <filter2>` | `type != 'microsoft.iot.telemetry' OR datacontenttype = 'application/json'` |
| Operaciones anidadas | `(<Comparison1>) AND (<Comparison2>)` | `(type != 'microsoft.iot.telemetry' OR datacontenttype = 'application/json') OR (specversion != '1.0')` |

> [!NOTE]
> En la versión preliminar, solo se admite la igualdad de cadenas (=, !=).

Al implementar o actualizar un filtro, puede que el cambio tarde unos minutos en reflejarse en la canalización de datos.
