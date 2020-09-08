---
title: Configuración de diagnósticos
titleSuffix: Azure Digital Twins
description: Consulte cómo habilitar el registro con la configuración de diagnóstico.
author: baanders
ms.author: baanders
ms.date: 7/28/2020
ms.topic: troubleshooting
ms.service: digital-twins
ms.openlocfilehash: fc397b6d6beb719e11dc3959bbcf4d75c08a8dda
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723935"
---
# <a name="troubleshooting-azure-digital-twins-diagnostics-logging"></a>Solución de problemas de Azure Digital Twins: Registro de diagnóstico

Azure Digital Twins recopila [métricas](troubleshoot-metrics.md) para la instancia del servicio que proporcionan información sobre el estado de los recursos. Puede usar estas métricas para evaluar el estado general del servicio Azure Digital Twins y los recursos conectados a él. Estas estadísticas orientadas al usuario ayudan a ver lo que está ocurriendo con su instancia de Azure Digital Twins y a realizar el análisis de la causa raíz de los problemas sin necesidad de ponerse en contacto con el soporte técnico de Azure.

En este artículo se muestra cómo activar el **registro de diagnóstico** para los datos de métricas de la instancia de Azure Digital Twins. Estos registros se pueden usar como ayuda para solucionar problemas del servicio y configurar opciones de diagnóstico para enviar métricas de Azure Digital Twins a diferentes destinos. Puede encontrar más información sobre esta configuración en [*Creación de una configuración de diagnóstico para enviar los registros y las métricas de la plataforma a diferentes destinos*](../azure-monitor/platform/diagnostic-settings.md).

## <a name="turn-on-diagnostic-settings-with-the-azure-portal"></a>Activación de la configuración de diagnóstico con Azure Portal

Aquí se describe cómo habilitar la configuración de diagnóstico de la instancia de Azure Digital Twins:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la instancia de Azure Digital Twins. Escriba el nombre en la barra de búsqueda para encontrarla. 

2. En el menú, seleccione **Configuración de diagnóstico** y **Agregar configuración de diagnóstico**.

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings.png" alt-text="Captura de pantalla que muestra la página de configuración de diagnóstico y el botón para agregarla":::

3. En la página siguiente, rellene los valores a continuación:
     * **Diagnostic setting name** (Nombre de la configuración de diagnóstico): Asigne un nombre a la configuración de diagnóstico.
     * **Category details** (Detalles de categoría): Elija qué operaciones desea supervisar y marque las casillas para habilitar el diagnóstico para ellas. Las operaciones sobre las que puede informar la configuración de diagnóstico son:
        - DigitalTwinsOperation
        - EventRoutesOperation
        - ModelsOperation
        - QueryOperation
        - AllMetrics
        
        Para más información sobre estas opciones, consulte la sección [*Detalles de categoría*](#category-details) a continuación.
     * **Detalles de destino**: Elija dónde desea enviar los registros. También puede seleccionar cualquier combinación de las tres opciones:
        - Enviar a Log Analytics
        - Archivar en una cuenta de almacenamiento
        - Transmitir a un centro de eventos

        Si es necesario para el destino seleccionado, se le pedirá que rellene detalles adicionales.  
    
4. Guarde la configuración nueva. 

    :::image type="content" source="media/troubleshoot-diagnostics/diagnostic-settings-details.png" alt-text="Captura de pantalla que muestra la página de configuración de diagnóstico en la que el usuario ha rellenado un nombre de configuración de diagnóstico y ha realizado algunas selecciones para los detalles de la categoría y los detalles del destino. El botón Guardar está resaltado.":::

La nueva configuración surte efecto en unos 10 minutos. Después, los registros aparecen en el destino configurado en la página **Configuración de diagnóstico** de la instancia. 

## <a name="category-details"></a>Detalles de categoría

Aquí encontrará más información sobre las categorías de registro que se pueden seleccionar en **Category details** (Detalles de categoría) al configurar los valores de diagnóstico.

| Categoría de registro | Descripción |
| --- | --- |
| ADTModelsOperation | Registra todas las llamadas API relativas a modelos. |
| ADTQueryOperation | Registra todas las llamadas API relativas a consultas. |
| ADTEventRoutesOperation | Registra todas las llamadas API relativas a rutas de eventos, así como la salida de eventos de Azure Digital Twins a un servicio de punto de conexión como Event Grid, Event Hubs y Service Bus. |
| ADTDigitalTwinsOperation | Registra todas las llamadas API relativas a Azure Digital Twins. |

Cada categoría de registro consta de operaciones de escritura, lectura, eliminación y acción.  Estas se asignan a las llamadas API REST de la siguiente manera:

| Tipo de evento | Operaciones de API REST |
| --- | --- |
| Escritura | PUT y PATCH |
| Lectura | GET |
| Eliminar | Delete |
| Acción | POST |

Esta es una lista completa de las operaciones y las [llamadas API REST de Azure Digital Twins](https://docs.microsoft.com/rest/api/azure-digitaltwins/) correspondientes que se registran en cada categoría. 

>[!NOTE]
> Cada categoría de registro contiene varias operaciones y llamadas API REST. En la tabla siguiente, cada categoría de registro se asigna a todas las operaciones y llamadas API REST que hay debajo hasta que se muestra la siguiente categoría de registro. 

| Categoría de registro | Operación | Llamadas API REST y otros eventos |
| --- | --- | --- |
| ADTModelsOperation | Microsoft.DigitalTwins/models/write | Digital Twin Models Update API |
|  | Microsoft.DigitalTwins/models/read | Digital Twin Models Get By ID API y List API |
|  | Microsoft.DigitalTwins/models/delete | Digital Twin Models Delete API |
|  | Microsoft.DigitalTwins/models/action | Digital Twin Models Add API |
| ADTQueryOperation | Microsoft.DigitalTwins/query/action | Query Twins API |
| ADTEventRoutesOperation | Microsoft.DigitalTwins/eventroutes/write | Event Routes Add API |
|  | Microsoft.DigitalTwins/eventroutes/read | Event Routes Get By ID API y List API |
|  | Microsoft.DigitalTwins/eventroutes/delete | Event Routes Delete API |
|  | Microsoft.DigitalTwins/eventroutes/action | Error al intentar publicar eventos en un servicio de punto de conexión (no una llamada API) |
| ADTDigitalTwinsOperation | Microsoft.DigitalTwins/digitaltwins/write | Agregar, agregar relaciones, actualizar, actualizar componente de Digital Twins |
|  | Microsoft.DigitalTwins/digitaltwins/read | Obtener por identificador, obtener componente, obtener relación por identificador, enumerar relaciones entrantes, enumerar relaciones de Digital Twins |
|  | Microsoft.DigitalTwins/digitaltwins/delete | Eliminar, eliminar relación de Digital Twins |
|  | Microsoft.DigitalTwins/digitaltwins/action | Enviar telemetría de componentes, enviar telemetría de Digital Twins |

## <a name="log-schemas"></a>Esquemas de registro 

Cada categoría de registro tiene un esquema que define cómo se notifican los eventos de la categoría de que se trate. Cada entrada de registro individual se almacena como texto y se le da formato como un blob JSON. Los campos del registro y los cuerpos JSON de ejemplo se proporcionan para cada tipo de registro siguiente. 

`ADTDigitalTwinsOperation`, `ADTModelsOperation` y `ADTQueryOperation` usan un esquema de registro de API coherente; `ADTEventRoutesOperation` tiene su propio esquema independiente.

### <a name="api-log-schemas"></a>Esquemas de registro de API

Este esquema de registro es coherente para `ADTDigitalTwinsOperation`, `ADTModelsOperation` y `ADTQueryOperation`. Contiene información pertinente para las llamadas a API para una instancia de Azure Digital Twins.

Estas son las descripciones de campo y propiedad de los registros de API.

| Nombre del campo | Tipo de datos | Descripción |
|-----|------|-------------|
| `Time` | DateTime | La fecha y la hora en que ocurrió el evento de auditoría, en UTC |
| `ResourceID` | String | El identificador de recurso Azure Resource Manager para el recurso en el que tuvo lugar el evento |
| `OperationName` | String  | El tipo de acción que se realiza durante el evento |
| `OperationVersion` | String | La versión de API utilizada durante el evento |
| `Category` | String | El tipo de recurso que se está emitiendo |
| `ResultType` | String | El resultado del evento |
| `ResultSignature` | String | El código de estado HTTP para el evento |
| `ResultDescription` | String | Detalles adicionales acerca del evento |
| `DurationMs` | String | Cuánto tiempo se tardó en realizar el evento en milisegundos |
| `CallerIpAddress` | String | Una dirección IP de origen enmascarada para el evento |
| `CorrelationId` | Guid | Identificador único proporcionado por el cliente para el evento |
| `Level` | String | La gravedad del registro del evento |
| `Location` | String | La región en la que tuvo lugar el evento |
| `RequestUri` | Identificador URI | El punto de conexión utilizado durante el evento |

A continuación se muestran ejemplos de cuerpos JSON para estos tipos de registros.

#### <a name="adtdigitaltwinsoperation"></a>ADTDigitalTwinsOperation

```json
{
  "time": "2020-03-14T21:11:14.9918922Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/digitaltwins/write",
  "operationVersion": "2020-05-31-preview",
  "category": "DigitalTwinOperation",
  "resultType": "Success",
  "resultSignature": "200",
  "resultDescription": "",
  "durationMs": "314",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "2f6a8e64-94aa-492a-bc31-16b9f0b16ab3",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/digitaltwins/factory-58d81613-2e54-4faa-a930-d980e6e2a884?api-version=2020-05-31-preview"
}
```

#### <a name="adtmodelsoperation"></a>ADTModelsOperation

```json
{
  "time": "2020-10-29T21:12:24.2337302Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/models/write",
  "operationVersion": "2020-05-31-preview",
  "category": "ModelsOperation",
  "resultType": "Success",
  "resultSignature": "201",
  "resultDescription": "",
  "durationMs": "935",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "9dcb71ea-bb6f-46f2-ab70-78b80db76882",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/Models?api-version=2020-05-31-preview",
}
```

#### <a name="adtqueryoperation"></a>ADTQueryOperation

```json
{
  "time": "2020-12-04T21:11:44.1690031Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/query/action",
  "operationVersion": "2020-05-31-preview",
  "category": "QueryOperation",
  "resultType": "Success",
  "resultSignature": "200",
  "resultDescription": "",
  "durationMs": "255",
  "callerIpAddress": "13.68.244.*",
  "correlationId": "1ee2b6e9-3af4-4873-8c7c-1a698b9ac334",
  "level": "4",
  "location": "southcentralus",
  "uri": "https://myinstancename.api.scus.digitaltwins.azure.net/query?api-version=2020-05-31-preview",
}
```

### <a name="egress-log-schemas"></a>Esquemas de registro de salida

Este es el esquema para los registros `ADTEventRoutesOperation`. Contienen detalles relativos a las excepciones y las operaciones de API en torno a los puntos de conexión de salida conectados a una instancia de Azure Digital Twins.

|Nombre del campo | Tipo de datos | Descripción |
|-----|------|-------------|
| `Time` | DateTime | La fecha y la hora en que ocurrió el evento de auditoría, en UTC |
| `ResourceId` | String | El identificador de recurso Azure Resource Manager para el recurso en el que tuvo lugar el evento |
| `OperationName` | String  | El tipo de acción que se realiza durante el evento |
| `Category` | String | El tipo de recurso que se está emitiendo |
| `ResultDescription` | String | Detalles adicionales acerca del evento |
| `Level` | String | La gravedad del registro del evento |
| `Location` | String | La región en la que tuvo lugar el evento |
| `EndpointName` | String | El nombre del punto de conexión de salida creado en Azure Digital Twins |

A continuación se muestran ejemplos de cuerpos JSON para estos tipos de registros.

#### <a name="adteventroutesoperation"></a>ADTEventRoutesOperation

```json
{
  "time": "2020-11-05T22:18:38.0708705Z",
  "resourceId": "/SUBSCRIPTIONS/BBED119E-28B8-454D-B25E-C990C9430C8F/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.DIGITALTWINS/DIGITALTWINSINSTANCES/MYINSTANCENAME",
  "operationName": "Microsoft.DigitalTwins/eventroutes/action",
  "category": "EventRoutesOperation",
  "resultDescription": "Unable to send EventGrid message to [my-event-grid.westus-1.eventgrid.azure.net] for event Id [f6f45831-55d0-408b-8366-058e81ca6089].",
  "correlationId": "7f73ab45-14c0-491f-a834-0827dbbf7f8e",
  "level": "3",
  "location": "southcentralus",
  "properties": {
    "endpointName": "endpointEventGridInvalidKey"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la configuración de diagnóstico, consulte el artículo [*Recopilación y consumo de los datos de registro provenientes de los recursos de Azure*](../azure-monitor/platform/platform-logs-overview.md).
* Para información sobre las métricas de Azure Digital Twins, consulte el artículo [*Solución de problemas: visualización de las métricas con Azure Monitor*](troubleshoot-metrics.md).
* Para ver cómo habilitar las alertas de las métricas, consulte [*Solución de problemas: Configuración de alertas*](troubleshoot-alerts.md).
