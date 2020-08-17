---
title: Supervisión de los registros de diagnósticos de Media Services mediante Azure Monitor | Microsoft Docs
description: En este artículo se muestra cómo enrutar y ver registros de diagnóstico mediante Azure Monitor.
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
ms.date: 07/08/2019
ms.author: juliako
ms.custom: devx-track-azurecli
ms.openlocfilehash: 15c8cd3eff7b0eb64e1a512282de129f606a33ce
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501242"
---
# <a name="monitor-media-services-diagnostic-logs"></a>Supervisión de los registros de diagnósticos de Media Services

[Azure Monitor](../../azure-monitor/overview.md) le permite supervisar métricas y registros de diagnóstico que le ayudan a conocer el rendimiento de sus aplicaciones. Para obtener una descripción detallada de esta característica y saber por qué se recomienda usar las métricas y los registros de diagnósticos de Azure Media Services, consulte [Supervisar las métricas de Media Services y los registros de diagnóstico](media-services-metrics-diagnostic-logs.md).

En este artículo se muestra cómo enrutar los datos a la cuenta de almacenamiento y, después, ver dichos datos.

## <a name="prerequisites"></a>Prerrequisitos

- [Cree una cuenta de Media Services](./create-account-howto.md).
- Consulte [Supervisar las métricas de Media Services y los registros de diagnóstico](media-services-metrics-diagnostic-logs.md).

## <a name="route-data-to-the-storage-account-using-the-portal"></a>Enrutamiento de datos a la cuenta de almacenamiento mediante el portal

1. Inicie sesión en Azure Portal en https://portal.azure.com.
1. Vaya a la cuenta de Media Services en y haga clic en **Configuración de diagnóstico** en **Supervisión**. Aquí verá una lista de todos los recursos de la suscripción que generan datos de supervisión a través de Azure Monitor.

    ![Sección Configuración de diagnóstico](media/media-services-diagnostic-logs/logs01.png)

1. Haga clic en **Agregar configuración de diagnóstico**.

   Una configuración de diagnóstico de recurso es una definición de *qué* datos de supervisión deberían enrutarse desde un recurso en particular y *adónde* deberían ir esos datos de supervisión.

1. En la sección que aparece, otorgue un **nombre** a la configuración y marque la casilla **Archivar en una cuenta de almacenamiento**.

    Seleccione la cuenta de almacenamiento a la que desea enviar registros y pulse **Aceptar**.
1. Marque todas las casillas debajo de **Registro** y **Métrica**. Según el tipo de recurso, puede que solo tenga una de estas opciones. Estas casillas controlan qué categorías de datos de registros y de métricas disponibles para ese tipo de recurso se envían al destino que ha seleccionado, en este caso, una cuenta de almacenamiento.

   ![Sección Configuración de diagnóstico](media/media-services-diagnostic-logs/logs02.png)
1. Establezca el control deslizante **Retención (días)** en 30. Este control deslizante establece un número de días para retener los datos de supervisión en la cuenta de almacenamiento. Azure Monitor elimina automáticamente los datos anteriores al número de días especificado. Con una retención de cero días, los datos se almacenan indefinidamente.
1. Haga clic en **Save**(Guardar).

Los datos de supervisión del recurso ahora fluyen a la cuenta de almacenamiento.

## <a name="route-data-to-the-storage-account-using-the-azure-cli"></a>Enrutamiento de datos a la cuenta de almacenamiento mediante la CLI de Azure

Para habilitar el almacenamiento de registros de diagnóstico en una cuenta de almacenamiento, debe usar el comando `az monitor diagnostic-settings` de la CLI de Azure:

```azurecli-interactive
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <name or ID of storage account> \
    --resource <target resource object ID> \
    --resource-group <storage account resource group> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

Por ejemplo:

```azurecli-interactive
az monitor diagnostic-settings create --name amsv3diagnostic \
    --storage-account storageaccountforams  \
    --resource "/subscriptions/00000000-0000-0000-0000-0000000000/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount" \
    --resource-group "amsResourceGroup" \
    --logs '[{"category": "KeyDeliveryRequests",  "enabled": true, "retentionPolicy": {"days": 3, "enabled": true }}]'
```

## <a name="view-data-in-the-storage-account-using-the-portal"></a>Visualización de datos en la cuenta de almacenamiento mediante el portal

Si ha seguido los pasos anteriores, los datos han comenzado a fluir a la cuenta de almacenamiento.

Puede que tenga que esperar hasta cinco minutos antes de que el evento aparezca en la cuenta de almacenamiento.

1. En el portal, vaya a la sección **Cuentas de almacenamiento** desde la barra de navegación a la izquierda.
1. Identifique la cuenta de almacenamiento que creó en la sección anterior y haga clic en ella.
1. Haga clic en **Blobs** y, después, en el contenedor con la etiqueta **insights-logs-keydeliveryrequests**. Este es el contenedor que contiene sus registros. Los datos de supervisión se desglosan en contenedores por Id. de recurso y, después, por fecha y hora.
1. Haga clic en los contenedores de ld. de recurso, fecha y hora para navegar hasta el archivo PT1H.json. Haga clic en el archivo PT1H.json y haga clic en **Descargar**.

 Ahora puede ver el evento JSON que se almacenó en la cuenta de almacenamiento.

### <a name="examples-of-pt1hjson"></a>Ejemplos de PT1H.json

#### <a name="clear-key-delivery-log"></a>Registro de entrega de claves sin cifrado

```json
{
  "time": "2019-05-21T00:07:33.2820450Z",
  "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/amsResourceGroup/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/AMSACCOUNT",
  "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
  "operationVersion": "1.0",
  "category": "KeyDeliveryRequests",
  "resultType": "Succeeded",
  "resultSignature": "OK",
  "durationMs": 253,
  "identity": {
    "authorization": {
      "issuer": "myIssuer",
      "audience": "myAudience"
    },
    "claims": {
      "urn:microsoft:azure:mediaservices:contentkeyidentifier": "e4276e1d-c012-40b1-80d0-ac15808b9277",
      "nbf": "1558396914",
      "exp": "1558400814",
      "iss": "myIssuer",
      "aud": "myAudience"
    }
  },
  "level": "Informational",
  "location": "westus2",
  "properties": {
    "requestId": "fb5c2b3a-bffa-4434-9c6f-73d689649add",
    "keyType": "Clear",
    "keyId": "e4276e1d-c012-40b1-80d0-ac15808b9277",
    "policyName": "SharedContentKeyPolicyUsedByAllAssets",
    "tokenType": "JWT",
    "statusMessage": "OK"
  }
}
```

#### <a name="widevine-encrypted-key-delivery-log"></a>Registro de entrega de claves con cifrado Widevine

```json
{
  "time": "2019-05-20T23:15:22.7088747Z",
  "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/amsResourceGroup/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/AMSACCOUNT",
  "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
  "operationVersion": "1.0",
  "category": "KeyDeliveryRequests",
  "resultType": "Succeeded",
  "resultSignature": "OK",
  "durationMs": 69,
  "identity": {
    "authorization": {
      "issuer": "myIssuer",
      "audience": "myAudience"
    },
    "claims": {
      "urn:microsoft:azure:mediaservices:contentkeyidentifier": "0092d23a-0a42-4c5f-838e-6d1bbc6346f8",
      "nbf": "1558392430",
      "exp": "1558396330",
      "iss": "myIssuer",
      "aud": "myAudience"
    }
  },
  "level": "Informational",
  "location": "westus2",
  "properties": {
    "requestId": "49613dd2-16aa-4595-a6e0-4e68beae6d37",
    "keyType": "Widevine",
    "keyId": "0092d23a-0a42-4c5f-838e-6d1bbc6346f8",
    "policyName": "DRMContentKeyPolicy",
    "tokenType": "JWT",
    "statusMessage": "OK"
  }
}
```

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="see-also"></a>Consulte también

* [Métricas de Azure Monitor](../../azure-monitor/platform/data-platform.md)
* [Registros de diagnóstico de Azure Monitor](../../azure-monitor/platform/platform-logs-overview.md)
* [Recopilación y consumo de datos de registro provenientes de los recursos de Azure](../../azure-monitor/platform/platform-logs-overview.md)

## <a name="next-steps"></a>Pasos siguientes

[Supervisión de métricas](media-services-metrics-howto.md)
