---
title: Esquema JSON de módulo gemelo - Azure
description: En este tema se describe el esquema JSON de módulo gemelo de Live Video Analytics on IoT Edge.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 8bd86bdc2c8de9ee586e785db2074fa772100420
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87053064"
---
# <a name="module-twin-json-schema"></a>Esquema JSON de módulo gemelo

Los dispositivos gemelos son documentos JSON que almacenan información acerca del estado del dispositivo, incluidos metadatos, configuraciones y condiciones. Azure IoT Hub mantiene un dispositivo gemelo para cada dispositivo que se conecta a IoT Hub. Para obtener una explicación detallada, consulte [Uso de módulos gemelos en IoT Hub](../../iot-hub/iot-hub-devguide-module-twins.md).

En este tema se describe el esquema JSON de módulo gemelo de Live Video Analytics on IoT Edge.

> [!NOTE]
> Para ser autorizado a acceder a recursos de Media Services y a Media Services API, se debe autenticar primero. Para más información, consulte [Acceso a la API de Azure Media Services](../latest/media-services-apis-overview.md#accessing-the-azure-media-services-api).

## <a name="module-twin-properties"></a>Propiedad del módulo gemelo

Live Video Analytics on IoT Edge expone las siguientes propiedades de módulo gemelo. 

|Propiedad |Obligatorio |Dinámica |Descripción |
|---|---|---|---|
|applicationDataDirectory |Sí |No |Ruta de acceso a un volumen montado para una configuración persistente. |
|azureMediaServicesArmId |Sí |No |Identificador único de Azure Resource Manager para la cuenta de Media Services.|
|aadTenantId |Sí |No |Identificador de inquilino de Azure AD de cliente.|
|aadServicePrincipalAppId |Sí |Sí |AppId de Azure AD creado por el cliente.|
|aadServicePrincipalCertificate |Sí<sup>*</sup>  |Sí |Certificado de AppId de Azure AD creado por el cliente.|
|aadServicePrincipalPassword |Sí<sup>*</sup>  |Sí |Contraseña de AppId de Azure AD creada por el cliente.|
|aadEndpoint |No |No |Punto de conexión de Azure AD específico de la nube. <br/>Predeterminado: `https://login.microsoftonline.com` |
|aadResourceId |No |No |Identificador de recurso o de audiencia de Azure AD específico de la nube. <br/>Predeterminado: `https://management.core.windows.net/` |
|armEndpoint |No |No |Punto de conexión de Azure Resource Manager específico de la nube. <br/>Predeterminado: `https://management.azure.com/` |
|diagnosticsLevel |No |Sí |Nivel de detalle de eventos: <br/>Información &#x02758; Advertencia &#x02758; Error &#x02758; Crítico &#x02758; Ninguno |
|diagnosticsEventsOutputName |No |Sí |Salida del centro para los eventos de diagnóstico. <br/>(Vacío significa que los diagnósticos no están publicados)|
|operationalEventsOutputName|No|Sí|Salida del concentrador para eventos operativos.<br/>(Vacío significa que los eventos operativos no se publican)
|logLevel|No|Sí|Uno de los siguientes: <br/>&#x000B7; Detallado<br/>&#x000B7; Información (predeterminado)<br/>&#x000B7; Advertencia<br/>&#x000B7; Error<br/>&#x000B7; Ninguno|
|logCategories|No|Sí|Lista separada por comas de los siguientes elementos: Application, MediaPipeline, Events <br/>Valor predeterminado: Application, Events|
|debugLogsDirectory|No|Sí|Directorio para los registros de depuración. Si está presente, se generan registros. Si no están presentes, los registros de depuración están deshabilitados.

<sup>*</sup>DEBE proporcionar una contraseña o certificado de entidad de servicio. 

Las propiedades dinámicas se pueden actualizar sin necesidad de reiniciar el módulo. Para obtener los valores de algunas de estas propiedades, siga las instrucciones del artículo [Obtención de credenciales para acceder a la API de Media Services](../latest/access-api-howto.md). 

Consulte el artículo sobre [la supervisión y el registro](monitoring-logging.md) para obtener más información sobre el rol de la configuración de diagnóstico opcional.

```
{ 
    "properties.desired": { 
        // Required 
        "applicationDataDirectory": "/var/lib/azuremediaservices", 
        "azureMediaServicesArmId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/<rgname>/providers/microsoft.media/mediaservices/<ams_account>", 
        "aadTenantId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalAppId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalPassword": "{Service principal password}", 

        // Optional API Access 
        "aadEndpoint": "https://<aad-endpoint>", 
        "aadResourceId": "https://management.core.windows.net/", 
        "armEndpoint": "https://management.azure.com/", 
        
        // Optional Diagnostics 
        "diagnosticsEventsOutputName": "lvaEdgeDiagnostics",
        "operationalEventsOutputName": "lvaEdgeOperational",
        "logLevel": "Information",
        "logCategories": "Application,Events"
    } 
} 
```

## <a name="next-steps"></a>Pasos siguientes

[Métodos directos](direct-methods.md)
