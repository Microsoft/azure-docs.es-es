---
title: Métricas y registros de diagnóstico de Media Services con Azure Monitor
titleSuffix: Azure Media Services
description: Aprenda a supervisar las métricas y los registros de diagnóstico de Azure Media Services a través de Azure Monitor.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: c03950d64c9ead17dfa5c07ef70ab2b7ee0e90bb
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89296658"
---
# <a name="monitor-media-services-metrics-and-diagnostic-logs-via-azure-monitor"></a>Supervisión de métricas y registros de diagnóstico de Media Services a través de Azure Monitor

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

[Azure Monitor](../../azure-monitor/overview.md) le permite supervisar las métricas y los registros de diagnóstico que le ayudan a comprender el rendimiento de sus aplicaciones. Todos los datos recopilados por Azure Monitor pueden clasificarse en uno de los dos tipos fundamentales: métricas y registros. Puede supervisar registros de diagnóstico de Media Services y crear alertas y notificaciones para las métricas y los registros recopilados. Puede usar el [Explorador de métricas](../../azure-monitor/platform/metrics-getting-started.md) para visualizar y analizar los datos de métrica. Puede enviar registros a [Azure Storage](https://azure.microsoft.com/services/storage/), difundirlos a [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/), exportarlos a [Log Analytics](https://azure.microsoft.com/services/log-analytics/), o bien usar servicios de terceros.

Para más información, consulte [Métricas de Azure Monitor](../../azure-monitor/platform/data-platform.md) y [Registros de diagnóstico de Azure Monitor](../../azure-monitor/platform/platform-logs-overview.md).

En este tema se analizan las [métricas de Media Services](#media-services-metrics) y los [registro de diagnóstico de Media Services](#media-services-diagnostic-logs) compatibles.

## <a name="media-services-metrics"></a>Métricas de Media Services

Las métricas se recopilan a intervalos regulares independientemente de si cambia el valor o no. Son útiles para las alertas, dado que se pueden muestrear con frecuencia y se puede activar una alerta con rapidez con una lógica relativamente sencilla. Para más información sobre cómo crear alertas de métricas, consulte [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../../azure-monitor/platform/alerts-metric.md).

Media Services permite supervisar las métricas para los siguientes recursos:

* Cuenta
* Punto de conexión de streaming

### <a name="account"></a>Cuenta

Puede supervisar las siguientes métricas de cuenta.

|Nombre de métrica|Nombre para mostrar|Descripción|
|---|---|---|
|AssetCount|Número de recursos|Recursos en su cuenta.|
|AssetQuota|Cuota de recursos|Cuota de recursos en su cuenta.|
|AssetQuotaUsedPercentage|Porcentaje de cuota de recursos usado|Porcentaje de la cuota de recursos ya utilizada.|
|ContentKeyPolicyCount|Número de directivas de clave de contenido|Directivas de clave de contenido en su cuenta.|
|ContentKeyPolicyCount|Cuota de directiva de clave de contenido|Cuota de directivas de clave de contenido en su cuenta.|
|ContentKeyPolicyQuotaUsedPercentage|Porcentaje utilizado de la cuota de directivas de clave de contenido|Porcentaje de la cuota de la directiva de clave de contenido ya utilizado.|
|StreamingPolicyCount|Número de directivas de streaming|Directivas de streaming en su cuenta.|
|StreamingPolicyQuota|Cuota de directivas de streaming|Cuota de directivas de streaming en su cuenta.|
|StreamingPolicyQuotaUsedPercentage|Porcentaje usado de la cuota de directiva de streaming|Porcentaje de la cuota de la directiva de streaming ya utilizado.|

También debe revisar las [cuotas y límites](limits-quotas-constraints.md) de la cuenta.

### <a name="streaming-endpoint"></a>Punto de conexión de streaming

Se admiten estas métricas de [Puntos de conexión de streaming](/rest/api/media/streamingendpoints) de Media Services:

|Nombre de métrica|Nombre para mostrar|Descripción|
|---|---|---|
|Requests|Requests|Proporciona el número total de solicitudes HTTP atendidas por el punto de conexión de streaming.|
|Salida|Salida|Bytes de salida totales por minuto por punto de conexión de streaming.|
|SuccessE2ELatency|Latencia correcta de extremo a extremo|Tiempo que transcurre desde que el punto de conexión de streaming recibió la solicitud hasta el momento en que se envió el último byte de la respuesta.|

### <a name="why-would-i-want-to-use-metrics"></a>¿Por qué querría usar las métricas?

Estos son algunos ejemplos de cómo la supervisión de las métricas de Media Services puede ayudarle a comprender el rendimiento de las aplicaciones. Algunas preguntas que se pueden solucionar con Media Services métricas son:

* ¿Cómo puedo supervisar el punto de conexión de streaming estándar para saber cuándo se superan los límites?
* ¿Cómo puedo saber si tengo suficientes unidades de escalado de puntos de conexión de streaming Premium?
* ¿Cómo puedo establecer una alerta para saber cuándo escalar verticalmente los puntos de conexión de streaming?
* ¿Cómo puedo establecer una alerta para saber cuándo se alcanzó el número máximo de salidas configuradas en la cuenta?
* ¿Cómo puedo ver el desglose de las solicitudes con errores y qué está causando el error?
* ¿Cómo puedo ver cuántas solicitudes HLS o DASH se extraen del empaquetador?
* ¿Cómo puedo establecer una alerta para saber cuándo se alcanza el umbral de n.º de solicitudes erróneas?

### <a name="example"></a>Ejemplo

Consulte [Supervisión de las métricas de Media Services](media-services-metrics-howto.md).

## <a name="media-services-diagnostic-logs"></a>Registros de diagnóstico de Media Services

Los registros de diagnóstico proporcionan datos exhaustivos y frecuentes acerca del funcionamiento de un recurso de Azure. Para más información, aprenda a [recopilar y usar los datos de registro provenientes de los recursos de Azure](../../azure-monitor/platform/platform-logs-overview.md).

Azure Media Services admite los siguientes registros de diagnóstico:

* Entrega de claves

### <a name="key-delivery"></a>Entrega de claves

|Nombre|Descripción|
|---|---|
|Solicitud de servicio de entrega de claves|Registros que muestran la información de la solicitud de servicio de entrega de claves. Para más información, consulte los [esquemas](media-services-diagnostic-logs-schema.md).|

### <a name="why-would-i-want-to-use-diagnostics-logs"></a>¿Por qué sería conveniente usar los registros de diagnóstico?

Algunos aspectos que puede examinar con los registros de diagnóstico de entrega de claves son:

* Ver el número de licencias que entrega el tipo DRM.
* Ver el número de licencias que entrega la directiva.
* Ver los errores por DRM o tipo de directiva.
* Ver el número de solicitudes de licencias no autorizadas de los clientes.

### <a name="example"></a>Ejemplo

Consulte [Supervisión de los registros de diagnósticos de Media Services](media-services-diagnostic-logs-howto.md).

## <a name="next-steps"></a>Pasos siguientes

* [Recopilación y consumo de datos de registro provenientes de los recursos de Azure](../../azure-monitor/platform/platform-logs-overview.md)
* [Creación, visualización y administración de alertas de métricas mediante Azure Monitor](../../azure-monitor/platform/alerts-metric.md)
* [Supervisión de las métricas de Media Services](media-services-metrics-howto.md)
* [Supervisión de los registros de diagnósticos de Media Services](media-services-diagnostic-logs-howto.md)
