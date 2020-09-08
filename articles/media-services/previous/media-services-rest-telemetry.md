---
title: Configuración de la telemetría de Azure Media Services con REST | Microsoft Docs
description: En este artículo se muestra cómo usar la telemetría de Azure Media Services mediante la API de REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: e1a314fb-cc05-4a82-a41b-d1c9888aab09
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 7fec4a958e730c485c998d1f0591b8e4d61df050
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89264205"
---
# <a name="configuring-azure-media-services-telemetry-with-rest"></a>Configuración de la telemetría de Azure Media Services con REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

En este tema se describen los pasos generales que puede llevar a cabo al configurar la telemetría de Azure Media Services (AMS) mediante la API de REST. 

>[!NOTE]
>Para una explicación detallada de lo que es la telemetría AMS y cómo consumirla, consulte el tema de [información general](media-services-telemetry-overview.md).

Los pasos descritos en este tema son:

- Obtener la cuenta de almacenamiento asociada con una cuenta de Media Services
- Obtener los puntos de conexión de notificación
- Crear un punto de conexión de notificación para la supervisión 

    Para crear un punto de conexión de notificación, establezca EndPointType en AzureTable (2) y endPontAddress en la tabla de almacenamiento (por ejemplo, https:\//telemetryvalidationstore.table.core.windows.net/).
  
- Obtener las configuraciones de supervisión

    Crear unos valores de configuración de supervisión para los servicios que desea supervisar. No se permite más que una configuración de supervisión. 

- Agregar una configuración de supervisión


 
## <a name="get-the-storage-account-associated-with-a-media-services-account"></a>Obtener la cuenta de almacenamiento asociada con la cuenta de Media Services

### <a name="request"></a>Solicitud

```console
GET https://wamsbnp1clus001rest-hs.cloudapp.net/api/StorageAccounts HTTP/1.1
x-ms-version: 2.19
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept: application/json; odata=verbose
Authorization: (redacted)
Host: wamsbnp1clus001rest-hs.cloudapp.net
Response
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 370
Content-Type: application/json;odata=verbose;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 8206e222-2a59-482c-a6a9-de6b8bda57fb
x-ms-request-id: 8206e222-2a59-482c-a6a9-de6b8bda57fb
X-Content-Type-Options: nosniff
DataServiceVersion: 2.0;
access-control-expose-headers: request-id, x-ms-request-id
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 02 Dec 2015 05:10:40 GMT
    
{"d":{"results":[{"__metadata":{"id":"https://wamsbnp1clus001rest-hs.cloudapp.net/api/StorageAccounts('telemetryvalidationstore')","uri":"https://wamsbnp1clus001rest-hs.cloudapp.net/api/StorageAccounts('telemetryvalidationstore')","type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.StorageAccount"},"Name":"telemetryvalidationstore","IsDefault":true,"BytesUsed":null}]}}
```

## <a name="get-the-notification-endpoints"></a>Obtener los puntos de conexión de notificación

### <a name="request"></a>Solicitud

```console
GET https://wamsbnp1clus001rest-hs.cloudapp.net/api/NotificationEndPoints HTTP/1.1
x-ms-version: 2.19
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept: application/json; odata=verbose
Authorization: (redacted)
Host: wamsbnp1clus001rest-hs.cloudapp.net
```

### <a name="response"></a>Response

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 20
Content-Type: application/json;odata=verbose;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: c68de2b3-0be1-4823-b622-6ca6f94a96b5
x-ms-request-id: c68de2b3-0be1-4823-b622-6ca6f94a96b5
X-Content-Type-Options: nosniff
DataServiceVersion: 2.0;
access-control-expose-headers: request-id, x-ms-request-id
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 02 Dec 2015 05:10:40 GMT
    
{  
    "d":{  
        "results":[]
    }
}
 ```

## <a name="create-a-notification-endpoint-for-monitoring"></a>Crear un punto de conexión de notificación para la supervisión

### <a name="request"></a>Solicitud

```console
POST https://wamsbnp1clus001rest-hs.cloudapp.net/api/NotificationEndPoints HTTP/1.1
x-ms-version: 2.19
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept: application/json; odata=verbose
Authorization: (redacted)
Content-Type: application/json; charset=utf-8
Host: wamsbnp1clus001rest-hs.cloudapp.net
Content-Length: 115

{  
    "Name":"monitoring",
    "EndPointAddress":"https:\//telemetryvalidationstore.table.core.windows.net/",
    "EndPointType":2
}
```

> [!NOTE]
> No olvide cambiar el valor de "https:\//telemetryvalidationstore.table.core.windows.net" en la cuenta de almacenamiento.

### <a name="response"></a>Response

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 578
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbnp1clus001rest-hs.cloudapp.net/api/NotificationEndPoints('nb%3Anepid%3AUUID%3A76bb4faf-ea29-4815-840a-9a8e20102fc4')
Server: Microsoft-IIS/8.5
request-id: e8fa5a60-7d8b-4b00-a7ee-9b0f162fe0a9
x-ms-request-id: e8fa5a60-7d8b-4b00-a7ee-9b0f162fe0a9
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
access-control-expose-headers: request-id, x-ms-request-id
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 02 Dec 2015 05:10:42 GMT
    
{"d":{"__metadata":{"id":"https://wamsbnp1clus001rest-hs.cloudapp.net/api/NotificationEndPoints('nb%3Anepid%3AUUID%3A76bb4faf-ea29-4815-840a-9a8e20102fc4')","uri":"https://wamsbnp1clus001rest-hs.cloudapp.net/api/NotificationEndPoints('nb%3Anepid%3AUUID%3A76bb4faf-ea29-4815-840a-9a8e20102fc4')","type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.NotificationEndPoint"},"Id":"nb:nepid:UUID:76bb4faf-ea29-4815-840a-9a8e20102fc4","Name":"monitoring","Created":"\/Date(1449033042667)\/","EndPointAddress":"https://telemetryvalidationstore.table.core.windows.net/","EndPointType":2}}
 ```

## <a name="get-the-monitoring-configurations"></a>Obtener las configuraciones de supervisión

### <a name="request"></a>Solicitud

```console
GET https://wamsbnp1clus001rest-hs.cloudapp.net/api/MonitoringConfigurations HTTP/1.1
x-ms-version: 2.19
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept: application/json; odata=verbose
Authorization: (redacted)
Host: wamsbnp1clus001rest-hs.cloudapp.net
```

### <a name="response"></a>Response

```console  
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 20
Content-Type: application/json;odata=verbose;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 00a3ee37-bb19-4fca-b5c7-a92b629d4416
x-ms-request-id: 00a3ee37-bb19-4fca-b5c7-a92b629d4416
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
access-control-expose-headers: request-id, x-ms-request-id
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 02 Dec 2015 05:10:42 GMT
    
{"d":{"results":[]}}
```

## <a name="add-a-monitoring-configuration"></a>Agregar una configuración de supervisión

### <a name="request"></a>Solicitud

```console
POST https://wamsbnp1clus001rest-hs.cloudapp.net/api/MonitoringConfigurations HTTP/1.1
x-ms-version: 2.19
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept: application/json; odata=verbose
Authorization: (redacted)
Content-Type: application/json; charset=utf-8
Host: wamsbnp1clus001rest-hs.cloudapp.net
Content-Length: 133
    
{  
   "NotificationEndPointId":"nb:nepid:UUID:76bb4faf-ea29-4815-840a-9a8e20102fc4",
   "Settings":[  
      {  
     "Component":"Channel",
     "Level":"Normal"
      }
   ]
}
```

### <a name="response"></a>Response

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 825
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbnp1clus001rest-hs.cloudapp.net/api/MonitoringConfigurations('nb%3Amcid%3AUUID%3A1a8931ae-799f-45fd-8aeb-9641740295c2')
Server: Microsoft-IIS/8.5
request-id: daede9cb-8684-41b0-a921-a3af66430cbe
x-ms-request-id: daede9cb-8684-41b0-a921-a3af66430cbe
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
access-control-expose-headers: request-id, x-ms-request-id
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 02 Dec 2015 05:10:43 GMT
    
{"d":{"__metadata":{"id":"https://wamsbnp1clus001rest-hs.cloudapp.net/api/MonitoringConfigurations('nb%3Amcid%3AUUID%3A1a8931ae-799f-45fd-8aeb-9641740295c2')","uri":"https://wamsbnp1clus001rest-hs.cloudapp.net/api/MonitoringConfigurations('nb%3Amcid%3AUUID%3A1a8931ae-799f-45fd-8aeb-9641740295c2')","type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.MonitoringConfiguration"},"Id":"nb:mcid:UUID:1a8931ae-799f-45fd-8aeb-9641740295c2","NotificationEndPointId":"nb:nepid:UUID:76bb4faf-ea29-4815-840a-9a8e20102fc4","Created":"2015-12-02T05:10:43.7680396Z","LastModified":"2015-12-02T05:10:43.7680396Z","Settings":{"__metadata":{"type":"Collection(Microsoft.Cloud.Media.Vod.Rest.Data.Models.ComponentMonitoringSettings)"},"results":[{"Component":"Channel","Level":"Normal"},{"Component":"StreamingEndpoint","Level":"Disabled"}]}}}
```

## <a name="stop-telemetry"></a>Detención de la telemetría

### <a name="request"></a>Solicitud

```console
DELETE https://wamsbnp1clus001rest-hs.cloudapp.net/api/MonitoringConfigurations('nb%3Amcid%3AUUID%3A1a8931ae-799f-45fd-8aeb-9641740295c2')
x-ms-version: 2.19
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept: application/json; odata=verbose
Authorization: (redacted)
Content-Type: application/json; charset=utf-8
Host: wamsbnp1clus001rest-hs.cloudapp.net
```

## <a name="consuming-telemetry-information"></a>uso de información de telemetría

Para información sobre el consumo de la información de telemetría, consulte [este](media-services-telemetry-overview.md) tema.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
