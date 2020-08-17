---
title: 'Esquemas de registros de diagnóstico de Azure Media Services: Azure'
description: En este artículo se muestran los esquemas de registros de diagnóstico de Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/20/2019
ms.author: juliako
ms.openlocfilehash: 2bc688a61235a31c3d6b6e88cbb90c0d14cca179
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092021"
---
# <a name="diagnostic-logs-schemas"></a>Esquemas de registros de diagnóstico

[Azure Monitor](../../azure-monitor/overview.md) le permite supervisar métricas y registros de diagnóstico que le ayudan a conocer el rendimiento de sus aplicaciones. Puede supervisar registros de diagnóstico de Media Services y crear alertas y notificaciones para las métricas y los registros recopilados. Puede enviar registros a [Azure Storage](https://azure.microsoft.com/services/storage/), difundirlos a [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) y exportarlos a [Log Analytics](https://azure.microsoft.com/services/log-analytics/), o bien usar servicios de terceros.

Para obtener información detallada, consulte [Métricas de Azure Monitor](../../azure-monitor/platform/data-platform.md) y [Registros de diagnóstico de Azure Monitor](../../azure-monitor/platform/platform-logs-overview.md).

En este artículo se describen los esquemas de registros de diagnóstico de Media Services.

## <a name="top-level-diagnostic-logs-schema"></a>Esquema de registros de diagnósticos de nivel superior

Para obtener una descripción detallada del esquema de registros de diagnóstico de nivel superior, consulte [Servicios, esquemas y categorías admitidos en los registros de diagnóstico de Azure](../../azure-monitor/platform/resource-logs-schema.md).

## <a name="key-delivery-log-schema"></a>Esquema de registro de entrega de claves

### <a name="properties"></a>Propiedades

Estas propiedades son específicas del esquema de registro de entrega de claves.

|Nombre|Descripción|
|---|---|
|keyId|Identificador de la clave solicitada.|
|keyType|Podría ser uno de los siguientes valores: "Clear" (sin cifrado), "FairPlay", "PlayReady" o "Widevine".|
|policyName|Nombre de Azure Resource Manager de la directiva.|
|tokenType|El tipo del token.|
|statusMessage|Mensaje de estado.|

### <a name="examples"></a>Ejemplos

Propiedades del esquema de solicitudes de entrega de claves.

```json
{
    "time": "2019-01-11T17:59:10.4908614Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Succeeded",
    "resultSignature": "OK",
    "durationMs": 315,
    "identity": {
        "authorization": {
            "issuer": "http://testacs",
            "audience": "urn:test"
        },
        "claims": {
            "urn:microsoft:azure:mediaservices:contentkeyidentifier": "3321e646-78d0-4896-84ec-c7b98eddfca5",
            "iss": "http://testacs",
            "aud": "urn:test",
            "exp": "1547233138"
        }
    },
    "level": "Informational",
    "location": "uswestcentral",
    "properties": {
        "requestId": "b0243468-d8e5-4edf-a48b-d408e1661050",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "JWT",
        "statusMessage": "OK"
    }
} 
```

```json
 {
    "time": "2019-01-11T17:59:33.4676382Z",
    "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000/RESOURCEGROUPS/SBKEY/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/SBDNSTEST",
    "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
    "operationVersion": "1.0",
    "category": "KeyDeliveryRequests",
    "resultType": "Failed",
    "resultSignature": "Unauthorized",
    "durationMs": 2,
    "level": "Error",
    "location": "uswestcentral",
    "properties": {
        "requestId": "875af030-b77c-416b-b7e1-58f23ebec182",
        "keyType": "Clear",
        "keyId": "3321e646-78d0-4896-84ec-c7b98eddfca5",
        "policyName": "56a70229-82d0-4174-82bc-e9d3b14e5dbf",
        "tokenType": "None",
        "statusMessage": "No token present in authorization header or URL."
    }
} 
```

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes

[Supervisión de los registros de diagnóstico y las métricas de Media Services](media-services-metrics-diagnostic-logs.md)
